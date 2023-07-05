# ADR: Market Agnostic Membership API
###### Authored: 02/21/2023
###### Author(s): Caleb Halford (vn54dmn)
###### Decision: Option 2
​
## Introduction
Walmart is growing! The past several years have come with significant growth, both in our existing markets, and through some strategic acquisitions, we have expanded into new markets as well. In addition, Walmart Plus has proven to be very successful, and as we continue to grow, we want to accomodate the unique needs of these markets with as little disruption as possible to existing features and functionality.

With new membership initiatives starting to add their respective functionalities into the glass app, we are seeing a need to provide a "template" to provide a clear path for these new initiatives to integrate into our platform.
​
## Motivation
With the success of Walmart Plus in the US market, other markets are looking to grab their share of that sweet, sweet membership money. As things are now, each membership program is creating their own API for integration into their respective market applications. Since these market applications share a common codebase, integrating multiple membership program APIs directly into our various Plugins will quickly become overly complicated, leading to conflicts and a lot of bad code smell...

Before things get bad, we propose the creation of a common Membership API that each membership program would conform to. If successful this should greatly simplify the integration of these membership programs, and lead to more sustainable and reliable code. This will also align with our stated mission of providing a market agnostic platform, capable of scaling into any business segment we choose.
​
### Guiding Principles
1. Decouple Membership Program APIs from being integrated into Plugins directly.
2. Have a clear migration path to a market agnostic Membership Plugin
3. Keep refactoring to a minimum (at least initially).
​
## Considered Options
### Option 1: Create a new "Membership" plugin and wrap existing APIs.
For this option, we will create a new "Membership" plugin and define our generic membership API. Walmart Plus is our most complete membership offering at the moment, so this API would be tailored to the needs of W+ initially. However, we will want to keep adoption of this API by other membership programs in mind. This would likely mean making many of the parameters and functions optional, allowing them to be adopted by other memberships when they are ready.

With the generic membership API defined, we will then create a wrapper for each of the existing membership programs (W+, Delivery Pass, etc) which will handle the translation of API calls to each membership program's API as it is currently defined. This allows for the immediate integration of the generic API while needing only minimal refactoring on existing membership APIs, until they can adopt the generic API completely. Keep in mind that these wrappers are intended to be temporary, and not a permanent solution.
​
#### Pros:
- Minimal refactoring of existing membership APIs.
- Provides a clear path for migration efforts through the definition of an "ideal" Membership API.
- Allows for immediate integration of the Membership API through the use of API wrappers.
​
#### Cons:
- Due to the size of the Walmart Plus API, there will be a good amount of code needed just to wrap everything.
- Any changes to the APIs will require changes in more than one place. For example, changes to the W+ API would necessitate changes to the W+ API wrapper as well.
​
#### Example:
*​Note: The code below will not compile. It is intended as high-level pseudocode for demonstration purposes only.*

Using Walmart Plus as the example subject, the following showcases a possible implementation:

**Example MembershipAPI definition:**
```swift
public protocol MembershipAPI {
    /// Subscribe to get infomation about the available membership plans.
    var planInfo: AnyValuePublisher<MembershipPlansResponse?, Never> { get }

    /// Generic membership coordinator builder
    func makeMembershipCoordinator(
        presenter: GlassNavigationController,
        configuration: MembershipHubConfiguration,
        destination: MembershipDestination?,
        deeplink: DeepLink?
    ) -> Coordinator<MembershipFeedback>?
    
    ...
}

// MARK: Structs

/// Membership Hub Screen Configuration
public struct MembershipHubConfiguration {
    /// Membership Hub's selected segment/tab page
    ...
}

// MARK: - Enums

/// Feedback cases supported by Membership coordinators
public enum MembershipFeedback: Equatable {
    ...
}

/// Routing destinations
public enum MembershipDestination: Equatable {
    ...
}

...
```

**WalmartPlusAPI converted to use MembershipAPI defined configuration data models and data types:**
```swift
public protocol WalmartPlusAPI {
    /// This method returns a coordinator for a Walmart Plus flow.
    /// - Parameters:
    ///   - presenter: The navigation controller the Walmart Plus coordinator should use to present itself
    ///   - configuration: Walmart+ Hub Coordinator's landing screen configuration
    ///   - destination: Walmart+ Hub Coordinator's deeplink route
    ///   - skipBenefitsInfo: Pass in true if you would like to Directly land on Select Plan Page
    ///   - bottomSheetConfiguration: customizable options to populate the Bottom sheet confirmation
    func makeWalmartPlusCoordinator(
        presenter: GlassNavigationController,
        configuration: MembershipHubConfiguration,
        destination: MembershipDestination?,
        skipBenefitsInfo: Bool,
        bottomSheetConfiguration: MembershipBottomSheetConfiguration?,
        deepLink: DeepLink?
    ) -> Coordinator<MembershipFeedback>?

    ...
}

...
```

**An example MembershipAPI adaptor for WalmartPlusAPI:**
```swift
final class MembershipWalmartPlusAdapter {
    static func makeWalmartPlusCoordinator(container: Container,
                                           presenter: GlassNavigationController,
                                           configuration: MembershipHubConfiguration,
                                           destination: MembershipDestination?,
                                           deeplink: DeepLink?) -> Coordinator<MembershipFeedback>? {
        let walmartPlusAPI: WalmartPlusAPI = container.get()
        return walmartPlusAPI.makeWalmartPlusCoordinator(presenter: presenter,
                                                         configuration: configuration.convert(),
                                                         destination: destination?.convert(),
                                                         deepLink: deeplink)
    }
    
    ...
}

extension MembershipHubConfiguration {
    func convert() -> WalmartPlusHubConfiguration {
        /// Code to convert MembershipHubConfiguration to it's Walmart Plus equivalent
    }
}

extension MembershipDestination {
    func convert() -> WalmartPlusDestination {
        /// Code to convert MembershipDestination to it's Walmart Plus equivalent
    }
}

...
```
And finally:

**MembershipAPI implementation for returning the correct membership program coordinator:**
```swift
public class MembershipPluginAPI: MembershipAPI {
    private let container: Container
    private var membershipType: MembershipType = .WalmartPlus
    // Will likely use a different solution for determining which membership program is relevant,
    // but for demonstration purposes, this will suffice.

    // MARK: Initializers

    public init(container: Container) {
        self.container = container
    }

    // MARK: Coordinator Factories
    public func makeMembershipCoordinator(presenter: GlassNavigationController,
                                          configuration: MembershipHubConfiguration,
                                          destination: MembershipDestination?,
                                          deeplink: DeepLink?) -> Coordinator<MembershipFeedback>? {

        switch membershipType {
        case .WalmartPlus:
            return MembershipWalmartPlusAdapter.makeWalmartPlusCoordinator(container: container,
                                                                           presenter: presenter,
                                                                           configuration: configuration,
                                                                           destination: destination,
                                                                           deeplink: deeplink)
        case .DeliveryPass:
            return MembershipDeliveryPassAdapter.makeDeliveryPassCoordinator(presenter: presenter)
        }
    }
    
    ...
}
```

**Example usage of the above setup in a Plugin:**
Existing W+ Implementation:
```swift
public class ScanAndGoOnboardingCoordinator {
    func tryWalmartPlusTapped(sender: OnboardingViewController) {
        // Old W+ API implementation
        if let walmartPlusAPI: WalmartPlusAPI = container.getIfRegistered(),
           let navigationController = sender.navigationController as? GlassNavigationController {
            // TODO:- use 'W+ pick plan'
            guard let walmartPlusCoordinator = walmartPlusAPI.makeWalmartPlusCoordinator(
                    presenter: navigationController,
                    configuration: .init(),
                    destination: nil) else {
                let event = CustomerDiagnosticsEvent(
                    error: ScanAndGoDiagnosticError.failedToStartWalmartPlusCoordinator,
                    owner: .walmartPlus)
                self.analytics.trackDiagnostic(event: event, error: true)
                return
            }

            addChild(coordinator: walmartPlusCoordinator)
            walmartPlusCoordinator.start()
        }
    }
    
    ...
}
```

New MembershipAPI implementation:
```swift
public class ScanAndGoOnboardingCoordinator {
    func tryWalmartPlusTapped(sender: OnboardingViewController) {
        // New Membership API implementation
        let membershipAPI: MembershipAPI = container.get()
        if let navigationController = sender.navigationController as? GlassNavigationController {
            guard let membershipCoordinator = membershipAPI.makeMembershipCoordinator(
                    presenter: navigationController,
                    configuration: .init(),
                    destination: nil,
                    deeplink: nil) else {
                let event = CustomerDiagnosticsEvent(
                    error: ScanAndGoDiagnosticError.failedToStartWalmartPlusCoordinator,
                    owner: .walmartPlus
                )
                self.analytics.trackDiagnostic(event: event, error: true)
                return
            }

            addChild(coordinator: membershipCoordinator)
            membershipCoordinator.start()
        }
    }
    
    ...
}
```
As you can see above, the implementation of the existing WalmartPlusAPI can be easily adapted to the proposed MembershipAPI equivalent.
​
### Option 2: Re-evaluate and rename the current Walmart Plus API
Since Walmart Plus is our most complete membership offering, and it's functionality will overlap with the planned functionality of other membership initiatives, it may be most efficient to adapt the current Walmart Plus API into a more market agnostic version.

This is a compelling option for a couple reasons:
1. Walmart Plus is already coupled with many plugins, and can provide new memberships access to those integrations by simply implementing the relevant part of the API.
2. Obviates the need for API wrappers or adapter classes.

Due to the size of the Walmart Plus API (Currently over 2k lines long) it will be necessary to break it up into multiple APIs, grouping related functionalities together. In addition to making the API more easier to understand, it will also benefit new membership programs because they can focus on implementing one API at a time, simplifying their onboarding and integration.
For example:
"WalmartPlusAPI" could be broken-up into "MembershipRegistrationAPI", "MembershipBannerAPI", "MembershipReservationAPI", etc.

Similarly to option 1, new membership programs won't need every integration or function that the Walmart Plus API currently provides. Some effort will need to be applied toward making many functions and variables optional, so that they can be integrated when needed, and so customer facing features don't break.
​
#### Pros:
- Potentially easier integration for new membership programs.
- No need for the addition of temporary API wrappers.

#### Cons:
- Can be done in phases, but will require a fair amount of refactoring.
​
#### Example:
*​Note: The code below will not compile. It is intended as high-level pseudocode for demonstration purposes only.*

​In the below example, I have taken a portion of the WalmartPlusAPI and renamed the various components to a more market agnostic equivalent:
```swift
public protocol MembershipAPI {
    /// Subscribe to get information about the Customer's Membership information
    var membershipInfo: AnyValuePublisher<MembershipStatusResponse?, Never> { get }

    /// Subscribe to get infomation about the available membership plans.
    var membershipPlanInfo: AnyValuePublisher<WalmartPlusPlansResponse?, Never> { get }

    /// Subscribe to get infomation about early access ccm value.
    var earlyAccessCCMValue: AnyValuePublisher<Bool?, Never> { get }

    /// This method returns a coordinator for a Membership flow.
    /// - Parameters:
    ///   - presenter: The navigation controller the Membership coordinator should use to present itself
    ///   - configuration: Membership Hub Coordinator's landing screen configuration
    ///   - destination: Membership Hub Coordinator's deeplink route
    ///   - skipBenefitsInfo: Pass in true if you would like to Directly land on Select Plan Page
    ///   - bottomSheetConfiguration: customizable options to populate the Bottom sheet confirmation
    func makeMembershipCoordinator(
        presenter: GlassNavigationController,
        configuration: MembershipHubConfiguration,
        destination: MembershipDestination?,
        skipBenefitsInfo: Bool,
        bottomSheetConfiguration: MembershipBottomSheetConfiguration?,
        deepLink: DeepLink?
    ) -> Coordinator<MembershipFeedback>?

    /// This method used to present splash page
    /// - Parameters:
    /// - type: To distinguish between splash/signup pagetype
    /// - navigationController: To push benefits and cancellation screen after bottomSheet dismiss
    /// - presenter: To present the bottom sheet
    /// - Confirmation Context: Sign Up Confirmation Screen context,
    ///   pass nil if you want default handling to navigate to explore benefits page
    /// - Configuration: To configure Splash Page with ProgramId, program Name, sourceType etc
    /// - Source: Analytics context to be passed
    func makeSplashPageCoordinator(type: SplashPageType,
                                   navigationController: GlassNavigationController,
                                   presenter: UIViewController,
                                   configuration: SplashPageConfiguration?,
                                   source: SplashPageAnalyticsPayload) -> Coordinator<MembershipFeedback>

    /// This method returns a coordinator for a Benefits Details Page.
    /// - Parameters:
    ///   - presenter: The navigation controller the Benefits Details Page should be presented on
    ///   - config: Benefits Details Page  requested program type config
    func makeBenefitsDetailPageCoordinator(presenter: GlassNavigationController,
                                           config: BenefitsDetailsPageConfiguration) ->
        Coordinator<MembershipFeedback>

    /// This method returns a cordinator for Offer List Page
    /// - Parameters:
    ///   - presenter: The navigation controller the Benefits Details Page should be presented on
    func makeOfferListPageCoordinator(presenter: GlassNavigationController, deeplink: DeepLink) ->
        Coordinator<MembershipFeedback>
        
    ...
}
```

And below is an example of breaking out a subset of Walmart Plus API functionality (Reserve Slot) into a separate API definition:
```swift
public protocol MembershipReservationAPI {
    /// This method returns a coordinator for a Manage Weekly Reservation.
    /// - Parameters:
    ///   - sourcePage: The page decides the sorting logic of the cards displayed
    ///   - presenter: The  controller in which Membership bottom sheet should be presented on
    ///   This API is only for active and trial users
    func makeManageWeeklyReservationCoordinator(
        presenter: GlassNavigationController,
        sourcePage: ManageWeeklySourcePage
    ) -> Coordinator<MembershipFeedback>
}

/// WRS Time Selection Screen Configuration
public struct WeeklyReserveSlotConfiguration: Equatable {
    ...
}

/// ManageWeeklyReservation Screen Configuration
public struct ManageWeeklyReservationConfiguration: Equatable {
    ...
}

extension WeeklyReserveSlotConfiguration.WeeklyReservation.Address {
    public init(shippingAddress: CartType.CartContext.ShippingAddress) {
        ...
    }
}

public enum WeeklyReserveSlotState: Equatable {
    ...
}

...
```
​
### Recommended Solution:
Option 2

### Additional Notes:
As part of this effort, some investigation and thought will need to be given to how the WalmartPlus specific UIShared components will be handled in this new paradigm of an agnostic membership API.
