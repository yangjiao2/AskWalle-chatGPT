# Automated Analytics Page View Event Tracking

- Author: John Liedtke
- Decision: Pending

## Introduction

A `pageView` analytics event is defined as the following according to the [standard spec](https://gecgithub01.walmart.com/glass/docs/blob/master/docs/mobile/analytics/standard-spec.md#pageview):

> Triggered when a view-component comes to foreground.

In the context of iOS, this typically translates to the root view controller of a feature appearing on screen, e.g. the "home page view controller" fires a `pageView` event during _every_ `viewDidAppear` appearance event.

In addition, a feature may decide to delay the firing of a `pageView` event until the content of a page is deemed "ready", e.g. the "item details page" fires the `pageView` event only after the product details has successfully loaded on the page and if the view has not _disappeared_ while becoming "ready".

### Today

The only way to track a `pageView` event on iOS is to manually call the [`trackPageView`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/GlassPlatform/WalmartPlatform/Sources/Analytics/AnalyticsTracking.swift#L172) method on `AnalyticsTracking` during or after the `viewDidAppear` appearance event.

### Android

Android currently automates all `pageView` tracking. Features need only to declare the name of the page on their activity or fragment and the analytics system handles the rest. Furthermore, they provide convenient hooks for manual `pageView` tracking and for delaying the `pageView` event based on a predicate.

## Motivation

The Glass iOS codebase is growing exponentially and repetitive/duplicative code bloats the binary, increases build times, sullies the code by increasing complexity, and degrades Xcode indexing (maybe). Therefore, the Platform team is actively auditing usage of its APIs to determine if they can be improved in a way that reduces the amount of work and code necessary for integration. For example, if a Platform API that is used by all modules is modified in such a way that it reduces integration code by ~10 LOC, that would remove over 1000 LOC today assuming a single integration. If multiple integrations are required, the potential reduction in code increases precipitously.

Analytics instrumentation accounts for a breathtaking and disconcerting amount of code in the Glass workspace. For example, the `CheckoutAnalytics.swift` file is 3500+ LOC and the accompanying test file, `CheckoutAnalyticsTests.swift` is approaching 7000 LOC. This represents 4% and 8% of all code in the module and test bundle respectively. However, this does not even include the requisite code to fire the analytics events and any delegation code from the view layer.

Clearly, Analytics is a prime candidate for _improvement_ and overall reduction in code since it impacts all modules and has a myriad of integration points. The ADR addresses the ubiquitous `pageView` event in particular.

Today, teams must manually delegate from the view layer to the coordinator layer and beyond to fire their `pageView` events. A typical `pageView` call looks akin to the following:

```swift
protocol SomePageViewControllerDelegate: AnyObject {

    func somePageViewControllerViewDidAppear(_: SomePageViewController)
}

final class SomePageViewController: BaseViewController {

    weak var delegate: SomePageViewControllerDelegate?

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        // 1
        delegate?.somePageViewControllerViewDidAppear(self) 
    }
}

final class SomePageCoordinator: Coordinator<Void> {

    let tracker: AnalyticsTracking

    init(tracker: AnalyticsTracking) {
        self.tracker = tracker
    }
}

// MARK: - SomePageViewControllerDelegate

extension SomePageCoordinator: SomePageViewControllerDelegate {
    func somePageViewControllerViewDidAppear(_: SomePageViewController) {
        // 2
        tracker.trackPageView(
            forPage: .init(nm: .myPage), 
            inContext: .myPageContext
            payload: [:] // construct mammoth payload
        )
    }
}
```

In addition to the delegation and actual firing of the event, test code would need to be written for (1) and (2).

As mentioned before, many teams must write additional code to delay the `pageView` event and ensure that it is fired _once_ per `viewDidAppear` event. Not only is this error-prone and repetitive, many teams do not implement the "once per `viewDidAppear`" requirement correctly; for example, some teams simply fire the event _once_ in the `start()` method of their coordinator.

### Guiding Principles

The goals of an "automated solution" include the following:

- **Reduction of code**: The LOC necessary to implement a "happy-path"/"standard" analytics `pageView` event _must_ be decreased.
- **Reduction of tests**: The number of unit tests required to verify a `pageView` must decrease.
- **Expedite instrumentation**: The development time necessary to implement a standard analytics `pageView` event must decrease.
- **Consistency**: The solution increases consistency throughout the app and precludes erroneous implementations of the `pageView` event spec.
- **Identification of current page**: The solution should allow for querying of the currently visible page; this will help with future analytics asks that require knowledge of the visible page, e.g. grabbing the current page for a deep link referrer!.

## Options

### Option 1: Swizzle `UIViewController` view appearance methods

The `pageView` event is driven by the [view appearance](https://developer.apple.com/documentation/uikit/uiviewcontroller#1652793) methods on `UIViewController`. [Swizzling](https://nshipster.com/method-swizzling/) these methods provide an automatic and convenient way to instrument view controllers that represent "analytics pages" to emit the `pageView` event in the appropriate appearance method, i.e. `viewDidAppear`. The view controllers that represent an analytics page can conform to a new `AnalyticsPageViewController` protocol. The protocol requirements include the requisite analytics data (page name, context, etc.) for the analytics system to emit the event for the page automatically.

The proposed `public` API and invariants are documented below:

```swift
/// An object that holds and publishes the current analytics page state of an ``AnalyticsPageViewController``
public final class AnalyticsPageViewNotifier {

    /// The various states of an analytics page view event.
    public enum PageState {
        /// The event is not ready and should not be fired.
        case pending(AnalyticsPageViewEvent.PageContext)

        /// The analytics page is ready and the `pageView` event can be fired.
        case ready(AnalyticsPageViewEvent)

        /// Page and context information about the analytics page.
        public var pageContext: AnalyticsPageViewEvent.PageContext {
            switch self {
            case .pending(let context): return context
            case .ready(let event): return event.pageContext
            }
        }

        /// Returns the ``AnalyticsPageViewEvent`` event if the page is ready.
        public var event: AnalyticsPageViewEvent? {
            switch self {
            case .pending: return nil
            case .ready(let event): return event
            }
        }
    }

    /// The current analytics page state.
    @Published public var state: PageState

    public init(_ state: PageState) {
        self.state = state
    }

    public init(_ event: AnalyticsPageViewEvent) {
        state = .ready(event)
    }

    public init(pending pageContext: AnalyticsPageViewEvent.PageContext) {
        state = .pending(pageContext)
    }
}

/// A structure that represents a page view event in the application.
public struct AnalyticsPageViewEvent {
    public typealias Publisher = AnyValuePublisher<AnalyticsPageViewEvent?, Never>

    /// Page and context information about an analytics page.
    public struct PageContext {
        /// The name of the page.
        public var name: AnalyticsSchema.PageEnum

        /// The context of the page.
        public var context: AnalyticsSchema.ContextEnum

        /// Creates a new page context object.
        ///
        /// - Parameters:
        ///   - name: The name of the page.
        ///   - context: The context that the page appears in.
        public init(name: AnalyticsSchema.PageEnum, context: AnalyticsSchema.ContextEnum) {
            self.name = name
            self.context = context
        }
    }

    /// The page and context information.
    public var pageContext: PageContext

    /// Additional metadata pertaining to the page view.
    public var payload: AnalyticsPayload

    /// Creates a page view event object.
    ///
    /// - Parameters:
    ///   - pageContext: The page name and context of the analytics page.
    ///   - payload: An optional metadata payload to include with the page view event.
    public init(pageContext: PageContext, payload: AnalyticsPayload = [:]) {
        self.pageContext = pageContext
        self.payload = payload
    }
}

/// A view controller that correlates to an analytics page.
///
/// Typically, the view controller is the primary content view controller on screen.
///
/// Page view events will automatically be recorded for view controllers that conform to this protocol. The event is
/// fired during the view controller's `viewDidAppear` lifecycle event. If the page state is
/// ``AnalyticsPageViewNotifier/PageState/pending(_:)`` , the analytics system subscribes to
/// ``AnalyticsPageViewNotifier/event`` from ``AnalyticsPageViewController/pageViewNotifier``  and
/// records the page view when the publisher emits the event in the ``AnalyticsPageViewNotifier/PageState/ready(_:)``
/// state. Only one page view event will ever be recorded per `viewDidAppear` lifecycle event.
///
/// If the view controller's view is not installed in a window, the page view event is **not** recorded. Furthermore,
/// the page view event will not be recorded if the `viewDidDisappear` lifecycle method occurs before the
/// ``AnalyticsPageViewNotifier/event`` emits a value.
///
public protocol AnalyticsPageViewController: UIViewController {
    /// A notifier that emits a page view event when the page's content is deemed visible to the user, i.e. 'ready'.
    ///
    /// For example, a view controller that wishes to fire the event when asynchronous content has successfully
    /// loaded can update the ``AnalyticsPageViewNotifier/state`` to `ready` when the content has loaded.
    var pageViewNotifier: AnalyticsPageViewNotifier { get }
}
```

#### Example 'ready' integration

Below is an example integration where the page is 'ready' by default. Notice that the consumer need only conform to `AnalyticsPageViewController` and declare a `AnalyticsPageViewNotifier` property initialized with a page view event.

Arguably, the consumer does not even need to bother _unit testing_ the implementation because they'd simply be verifying the values of constants!

```swift
final class GlobalScannerViewController: LDBaseViewController, AnalyticsPageViewController {
    let pageViewNotifier = AnalyticsPageViewNotifier(.scanItem)
    // ...
}

extension AnalyticsPageViewEvent {
    static let scanItem = Self(pageContext: .init(name: .scanItem, context: .globalScanner))
}
```

#### Example 'pending' integration

Below is an example integration where the page delays the `pageView` event until the model layer deems the page `ready`. Most features inflate their views with a single `Model` (view data) object that is applied in an `applyModel()` method. When the `model` layer creates the event, it need only pass the it in the `Model` with the rest of the view data.

```swift
final class DelayedPageViewController: UIViewController, AnalyticsPageViewController {
    private(set) lazy var pageViewNotifier = AnalyticsPageViewNotifier(model.pageViewState)

    struct Model {
        /// The current page state is driven by the model layer
        var pageViewState: AnalyticsPageViewNotifier.PageState = .pending(.myPage)
        // ...
    }

    var model = Model() {
        didSet {
            applyModel()
        }
    }

    func applyModel() {
        pageViewNotifier.state = model.pageViewState
        // ...
    }
}

extension AnalyticsPageViewEvent.PageContext {
    static let myPage = Self(name: .scanItem, context: .globalScanner)
}
```

Notice that even with the slightly more "complex" implementation, the consumer does not need to perform any delegation from the view layer and need only to set the `state` property when the page becomes ready. Instead of having to verify and test that a chain of methods are called, the consumer ensures that a property is set on the view controller. This helps shift testing to the construction of the event rather than testing the `viewDidAppear` requirement spec. The system also ensures that the event is only recorded once per `viewDidAppear` event!

### Swizzling

The analytics system would swizzle and listen for the view appearance events of all view controllers conforming to `AnalyticsPageViewController`. Internally, it would handle all of the requisite book keeping for the `pageView` analytics spec. The implementation is purposefully left out in this ADR.

### Pros/Cons

#### Pros

- Standard page view events require as little as 2 LOC and 0 unit tests.
- Delayed page view events are simplified for the consumer.
- Delayed page views are driven by the model layer.
- Delegation of view appearance methods are no longer required.
- Unit testing is simplified to verifying the construction of the event; there is no need to ensure that it is fired at the correct time.
- Enables querying of the currently visible page.
- Implementation is swappable with less intrusive mechanism such as `Base` classes or `NavigationAPI`.
- Decoupled completely from all other dependencies besides UIKit.

Major Pros:

- Enables future `buttonTap` enhancement where button taps could inherit the page and context information.

#### Cons

- Magical swizzling can cause unforeseen issues.
- Black magic to the consumer.

### Option 2: Wait for the omnipotent Navigation API

The NavigationAPI that is currently in development promises to provide fine-grained control over all navigational actions in the application. The control provides hooks for observing navigational actions that could subsequently be used to trigger analytics events such as `pageView`. However, even with the adoption of `NavigationAPI`, view controllers would still need to be identified as analytics pages like how view controllers conform to `AnalyticsViewController` in Option 1.

In fact, NavigationAPI like Swizzling is an implementation detail for automated `pageView` events rather than a full solution. The same API presented in Option 1 could be used, except NavigationAPI would be the source of appearance events rather than Swizzling.

Therefore, the same or a similar API for automated `pageView` events would be used for Option 2 except it would not be provided to consumers until NavigationAPI is completed and integrated with their feature.

### Option 2: Pros/Cons

#### Option 2: Pros

- No magical swizzling that has the potential to interfere with other dependencies.
- Potentially more control over when `pageView` events would be fired since all navigational actions are monitored.
- Similar API to Option 1.

#### Option 2: Cons

- Feature teams must fully integrate with NavigationAPI.
- NavigationAPI must be finished before teams can leverage automatic analytics.

## Option 3: Publish appearance events from base classes

Option 3, like Option 2, requires a similar "analytics page identification API" like the one proposed in Option 1, but relies on base classes to publish the requisite appearance events rather than Swizzling or a NavigationAPI.

Currently, Platform provides, but does not require base UI classes such as `BaseViewController` when constructing UI. Most features have adopted the base classes and migration would be fairly straightforward. In addition, requiring usage of base classes could enable future Platform enhancements.

### Example implementation

```swift
open class BaseViewController: UIViewController, ViewConstructable, Accessible {
    // ...

    /// Posted when a view controller appears.
    public static let viewDidAppearNotification = Notification.Name("viewDidAppear")

    open override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        NotificationCenter.default.post(
            name: Self.viewDidAppearNotification, 
            object: self
        )
    }
    // ...
}
```

### Option 3: Pros/Cons

#### Option 3: Pros

- No magical swizzling that has the potential to interfere with other dependencies.
- Most features leverage base classes today and arguably it should be a Platform requirement.
- Adoption of Base classes opens the door for similar global Platform enhancements.
- Similar API to Option 1.

#### Option 3: Cons

- Feature teams must migrate to base classes.
- Analytics implementation is coupled to base classes and cannot exist without it. If the automated solution were to be used by another Walmart team, they'd have to refactor their code to utilize base classes or implement a similar solution.

## Recommended Solution

Option 1 - Option 1 provides the most expedient way for teams to adopt automated `pageView` analytics and is completely decoupled from dependencies. Additionally, the solution provides the ability to swap the implementation for another such as the ones outlined in Options 1 & 2.

## Decision Outcome

Option 3.
