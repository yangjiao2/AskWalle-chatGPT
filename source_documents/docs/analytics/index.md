---
title: How to Analytics
navTitle: Analytics
---

# How to Analytics

- [Standard Spec](#standard-spec)
- [Tracking Events](#tracking-events)
- [Implementing Analytics](#implementing-analytics)
- [Writing Unit Tests](#writing-unit-tests)
- [Viewing Events](#viewing-events)
    - [Splunk](#splunk)
	- [Xcode](#xcode)
	- [Console.app](#console.app)
    - [Terminal.app](#terminal.app)
    - [Network log](#network-log)
- [Diagnostic Events](#diagnostic-events)
- [Updating SDK](#updating-the-sdk)

## Standard Spec

Analytics are tracked based on the [standard analytics spec](https://gecgithub01.walmart.com/glass/docs/blob/master/docs/mobile/analytics/standard-spec.md). The spec contains standardized event formats for events like page views, button taps, performance, and more.

## Tracking Events

### Analytics API

The [`AnalyticsTracking`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Protocols/AnalyticsTracking.html) protocol provides an API for tracking analytics events. The app's default container, instantiated in main AppDelegate, registers an implementation for the `AnalyticsTracking` protocol and a convenience property for accessing the implementation.

```swift
func trackAnalytics(container: Container) {
    let analytics: AnalyticsTracking = container.analytics
}
```

### Page Views

```swift
container.analytics.trackPageView(pageName: "my page name", context: "my section")
```

### Button Taps

```swift
container.analytics.trackButtonTap(buttonName: "my button", pageName: "my page", context: "my section")
```

### Async Events

```swift
container.analytics.trackAsyncEvent(name: "my async event")
```

### Performance Events

```swift
let phasedEvent = AnalyticsPhasedEvent(name: "performanceMetric", payload: ["name": "my performance event"])
phasedEvent.startPhase(name: "my phase", payload: [:])

// .. later
phasedEvent.complete()

container.analytics.trackPerformanceEvent(phasedEvent)
```

### ModuleView Events

```swift
// .. conform to `AnalyticsProduct` protocol
struct PDPProduct: AnalyticsProduct {
    let productName: String
    let itemBadges: String?
    let isSponsored: String?
    let productDisplayOrder: String?
    let athenaPayload: AnyDictionaryEncodable?
    let itemId: String
    let additionalProperty: Int
}

let product = PDPProduct(productName: "Apple TV", itemBadges: "clearance", isSponsored: "0", productDisplayOrder: nil, athenaPayload: AnyDictionaryEncodable(value: ["some_key": "some_value"]), itemId: "abc111", additionalProperty: 1)

let moduleViewEvent = AnalyticsModuleViewEvent(context: "Related Items",
                                               pageName: "PDP",
                                               payload: ["moduleZone":"contentZone1", "marketingMessageId": "holiday season"],
                                               products: [product])
                                               
moduleViewEvent.moduleLoaded(timeStamp: Date())

// .. when module is viewed
moduleViewEvent.moduleViewed(timeStamp: Date())

// .. when product is viewed
moduleViewEvent.productViewed(itemId: "abc111", viewedTimestamp: Date())

// .. later
moduleViewEvent.complete()
container.analytics.trackModuleViewEvent(moduleViewEvent)
```

### Custom Events

```swift
container.analytics.track(name: "my custom event", payload: ["name": "my event name"])
```

## Implementing Analytics

The section walks through a best practices approach to implementing analytics tracking in a feature.

First, create a class to contain helper functions for all of the events you'll be tracking in your feature. Inject an instance of `AnalyticsTracking` as an initializer parameter.

```swift
final class OnboardingAnalytics {
    private let analytics: AnalyticsTracking

    init(analytics: AnalyticsTracking) {
        self.analytics = analytics
    }
}
```

Instantiating an instance should look something like this:

```swift
let onboardingAnalytics = OnboardingAnalytics(analytics: container.analytics)
```

Next, we''ll define constants for values like page, section, and button names.

```swift
final class OnboardingAnalytics {
    enum Button: AnalyticsValue {
        case enterZipCode = "enter zip code"
        case getStarted = "get started"
    }

    enum Page: AnalyticsValue {
        case location
        case shareYourZipCode = "share your zip code"
        case welcome
    }

    enum Section: AnalyticsValue {
        case onboarding
    }

    private let analytics: AnalyticsTracking

    init(analytics: AnalyticsTracking) {
        self.analytics = analytics
    }
}
```
Now we can create tracking methods for each analytics event spec. It's very useful to include a link to the yabas spec as a reference as well as a comment above each function with the sensor ID.

```swift
/// http://yabas.edge.walmart.com/vast/feature/2138
extension OnboardingAnalytics {
    // helper for tracking page view events using the "onboarding" section
    private func trackPageView(page: Page, section: Section = .onboarding) {
        analytics.trackPageView(pageName: page.rawValue, context: section.rawValue)
    }

    /// USGLCO-4142S2T1
    func trackWelcomePageView() {
        trackPageView(page: .welcome)
    }
}
```

Finally, we can track a page view event from the coordinator that manages the welcome screen.

```swift
class WelcomeCoordinator: Coordinator<Void> {
    private let analytics: OnboardingAnalytics

    init(analytics: OnboardingAnalytics) {
        self.analytics = analytics
    }

}

extension WelcomeCoordinator: OnboardingViewControllerDelegate {
    func didAppear() {
        analytics.trackWelcomePageView()
    }
}
```

## Writing Unit Tests

Use `MockAnalyticsIntent` from the [`TestUtilities`](https://gecgithub01.walmart.com/walmart-ios/glass-platform/tree/development/TestUtilities) framework as a mock instance of the [`AnalyticsTracking`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Protocols/AnalyticsTracking.html) protocol. `MockAnalyticsIntent` will store tracked events in the `trackedEvents` array that you can use to make assertions that your analytics events are being tracked as expected.

The following is an example of a unit test that verifies that the welcome `pageView` is being tracked as expected when `didAppear` is called.

```swift
func test_didAppear_tracksPageViewEvent() throws {
    let intent = MockAnalyticsIntent()
    let coordinator = WelcomeCoordinator(
        analytics: OnboardingAnalytics(analytics: intent)
    )

    coordinator.didAppear()

    let event = try XCTUnwrap(intent.trackedEvents.first)
    XCTAssertEqual(event.name, .pageView)
    XCTAssertEqual(
        try XCTUnwrap(event.payload["ctx"] as? AnalyticsValue),
        OnboardingAnalytics.Section.onboarding.rawValue
    )
    XCTAssertEqual(
        try XCTUnwrap(event.payload["name"] as? AnalyticsValue),
        OnboardingAnalytics.Page.welcome.rawValue
    )
}
```

## Viewing Events

### Splunk

See the [Viewing Events](https://gecgithub01.walmart.com/glass/docs/blob/master/docs/mobile/analytics/viewing-events.md) section of the mobile analytics documentation.

### Xcode

A real-time log stream of the analytics events can be viewed in the Xcode log console.
To enable, open the Walmart App and enable Analytics console logging in the [debug panel](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/debug-panel/index.md).
Under "Settings", navigate to "Console Logging" and enable the "Analytics Logging" switch.

### Console.app

A real-time log stream of the analytics events can also be viewed using the [`Console.app`](https://support.apple.com/guide/console/welcome/mac) on a Mac computer. 

1. Enable as described [above](#xcode)
3. Open Console.app on your Mac.
4. Enable "Include Debug Messages" in the Menu Bar: Action->Include Debug Messages
5. Select the device from (1) in the panel on the left.
6. Tap "Start Streaming".
7. To filter for Walmart logs, type `subsystem:com.walmart` into the search box.
8. To filter for Analytics Logs, type `category:WalmartApp Analytics` into the search box.

### Terminal.app

Analytics events can also be viewed using the Terminal app on a Mac computer.

1. Open Terminal.app and enter

```
xcrun simctl spawn booted log stream\
 --level debug\
 --style syslog\
 --color auto\
 --predicate 'category contains "WalmartApp Analytics"'
 ```
 
 Various log options can be explored by typing `man log`.
 
### Network log

A non-real-time log of the analytics network requests can be viewed in the [debug panel](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/debug-panel/index.md).
Under "Networking"", navigate to "Analytics Log" to see a log of the network requests to analytics URLs.

## Diagnostic Events

See the [Diagnostic Events](diagnostic-events.md) documentation for information on recording events useful for developer monitoring and debugging.

## Updating the sdk

## Updating the sdk

Our analytics SDK version is controlled with [carthage package manager](https://github.com/carthage/carthage). You'll see it listed in the Cartfile and Cartfile.resolved. You can pull in a new compatible version of the Analytics SDK by executing:
```
mint run carthage update --platform ios --no-build cbb-glass-stubgenerator-ios
```
This should update the Cartfile.resolved file with a new compatible version. If you need to bump the major version, or update the compatibilty spec you'll update the Cartfile, then let `carthage` update the resolved file, to ensure that your packages stay [semver compatible](https://doc.rust-lang.org/cargo/reference/semver.html).

**WARNING**: The ccb-glass-stubgenerator-ios project does not use semantic versioning. So at this time a patch version is just as likely to contain breaking changes as it is bug fixes. 

Note the `mint run` prefix to ensure you are using the `carthage` tool that is compatible with our project.
