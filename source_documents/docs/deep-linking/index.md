---
title: Deep Linking
navTitle: Deep Linking
---

# Deep Linking

- [What Are Deep Links?](#what-are-deep-links)
- [How Are They Handled?](#how-are-they-handled)
- [Creating Deep Links](#creating-deep-links)
- [Registering Deep Links](#registering-deep-links)
- [Universal Links](#universal-links)
- [URL Scheme](#url-scheme)
- [Domain](#domain)
- [Supported List](#supported-list)
- [Testing](#testing-deep-links)
- [Debug Panel](#debug-panel)
- [Dashboard](#dashboard)
- [Documenting Deep Links](#documenting-deep-links)
- [More Information](#more-information)

## What Are Deep Links?

Deep Links are a form of [hyperlink](https://en.wikipedia.org/wiki/Hyperlink) that include a [uniform resource identifier (URI)](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) and a path that keys to a specific location or behavior within an app.  There are other potential uses for deep links, but generally speaking, when invoked, a properly handled deeplink will not only launch an app, but also navigate the user directly to the specified location within.

## How Are They Handled?

Deep Linking behavior is split between the `DeepLinking` plugin and the `WalmartPlatform` framework, where `WalmartPlatform` provides interception and interpretation, and `DeepLinking` provides configuration.

The `WalmartPlatform` framework receives openURL calls via its `AppDelegateMessaging` class.  It then broadcasts them to all registered `ApplicationDelegate`s, including an instance of `DeepLinkingApplicationDelegate`.  That delegate, on initialization, will have been provided a polymorphic instance of `DeepLinker`, likely originating from a `walmartApp` `Container` as the concrete class `DeepLinkRouter`.  Upon receiving the openURL call, the delegate forms a `DeepLink` object from the provided URL, and passes it into the `route(deepLink:)` call on its instance of `DeepLinker`.  From there, the `DeepLinker` checks to see if the Deep Link's route had been registered with a corresponding closure, and if so, the closure is called. Along with the valid path, the router also looks for allowed domains & schemes.

The `DeepLinking` plugin provides the `Routable` class, and its subclass variant `CoordinatorRoutable`, to be used in defining Deep Link routes and their expected behaviors.  The plugin's primary API class initializes and stores strong references to all registered Deep Links within a private `routables` array.  Upon initialization, each `Routable` object registers its paths and behaviors with the `DeepLinker` instance of the `Container` it was initialized with.

## Creating Deep Links

Each Deep Link is adapted as a subclass of `Routable` or `CoordinatorRoutable` (a variant capable of interacting with a `Coordinator`) within the `Deeplinking` Plugin.  Such classes must implement, at a minimum, the following:  

1. Any potential routes (or paths) to be handled
2. The expected behavior to occur when invoked

### Routable Example

```swift
private typealias RoutableExampleRoutes = String
private extension RoutableExampleRoutes {
    static let `default` = "example/path"
}

class RoutableExample: Routable {
    override var actionRoutes: [String] {
        [.default] // Any routes or paths should be included within this array
    }
    
    override func handleDeepLink(_ deepLink: DeepLink, state: DeepLinkNavigationState) -> Bool {
    
        // Any behavior expected to occur when invoked should go here
        
        // return true or false, depending on whether or not it succeeded
    }
}
```

### CoordinatorRoutable Example

```swift
private typealias CoordinatorRoutableExampleRoutes = String
private extension CoordinatorRoutableExampleRoutes {
    static let `default` = "example/path"
}

class RoutableExample: CoordinatorRoutable<CoordinatorResult> {
    override var actionRoutes: [String] {
        [.default] // Any routes or paths should be included within this array
    }
    
    override func handleDeepLink(_ deepLink: DeepLink, state: DeepLinkNavigationState) -> Coordinator<CoordinatorResult>? {
    
        // Any behavior expected to occur when invoked should go here
        
        // return coordinator or nil, depending on whether or not it succeeded
    }
}
```

## Registering Deep Links

After a `Routable` subclass has been created, it can be registered as simply as adding an instance of it to the `routables` array of the `DeepLinkingPluginAPI` class.

As part of documenting the supported deep links, newly added routes needs to be updated [here](supported-list.md).

## Universal Links

Universal Links are automatically converted to Deep Links for you. If you need to add a new Universal Link host, you would need to do so in `DeepLinker+UniversalLinks.swift` located in `WalmartPlatformExtensions`.

`DeepLinkRouter` automatically handles the resolution if it is configured correctly.
Update the supported list [here](supported-list.md)

### Example

```swift
struct ExampleUniversalLink: UniversalLink {
    let host = "example.walmart.com"
    
    func deepLinkURL(components: UniversalLinkComponents) -> URL? {
        URL(string: "walmart://\(components.url.path)")
    }
}
```

#### How to add a new Universal Link route

In order to register a new universal link in Glass, you should follow this steps:

1. Fork [Torbit-Static-Origins/USGM-Static-Assets](https://gecgithub01.walmart.com/Torbit-Static-Origins/USGM-Static-Assets) to update the AASA file on the Torbit Server.

2. Clone the repository you've just forked and switch to **Prod** branch.

3. Open the file `USGM-Static-Assets/nativemobile/well-known/apple-app-site-association`.
There are two sections to support both Production and Debug versions.

  Identifier                              | Configuration
  :--------------------------------------:|:----------:
  74G6H8XY8B.com.walmart.electronics      | Production
  DXZ5VF8836.com.walmart.beta.electronics | Debug

  Check out [Apple's documentation on the format of this file if you have questions](https://developer.apple.com/documentation/bundleresources/applinks). You will see the existing universal links routes defined (for example, `/plus/redeem`). Add your new link route to the list. Update both the Production and Debug sections as needed.

  Make your changes, and push your branch

4. Browse to the upstream repo and open a PR from your pushed branch. Make sure you set your PR to "land" to the **Prod** branch. Here is an [example PR](https://gecgithub01.walmart.com/Torbit-Static-Origins/USGM-Static-Assets/pull/138/files).

5. Post a CRQ (Change Request) for updating Torbit.
    Here is an [example CQR](https://walmartglobal.service-now.com/nav_to.do?uri=%2Fchange_request.do%3Fsys_id%3Dc2a35b3a970aa1144d6b3a37f053af6b%26sysparm_domain%3Dnull%26sysparm_domain_scope%3Dnull%26sysparm_view%3D%26sysparm_view_forced%3Dtrue) for the PR mentioned above.
    It is always recommended to make AASA updates for stage identifier, ie updating paths under `DXZ5VF8836.com.walmart.beta.electronics`, get your changes verified and plan updates for Production identifier in a separate CRQ. Also, the recommendation is to keep the CRQ window for 5-6 hrs to cover testing.

6. Go to `#glass-ios` on Slack and ask the `@glass-platform-ios` to review your PR and CRQ.

#### AASA

The AASA (apple-app-site-association) file must be updated to correctly tie the URL to the app. 
For more information, go [here](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html).

##### Validate Existing

As soon your PR gets merged to Prod branch of [USGM-Static-Assets](https://gecgithub01.walmart.com/Torbit-Static-Origins/USGM-Static-Assets), the [AASA](https://www.walmart.com/.well-known/apple-app-site-association) will reflect with the latest changes. However, [Apple CDN](https://app-site-association.cdn-apple.com/a/v1/walmart.com) may take around 3-4 hours to refresh based on historic updates. The [official documentation](https://developer.apple.com/documentation/xcode/supporting-associated-domains) of Apple says - *"Apple’s content delivery network requests the `apple-app-site-association` file for your domain within 24 hours. Devices check for updates approximately once per week after app installation."*  

You might like to validate your AASA file by running `swcutil dl -d <yourdomain>` at the commandline.
Note also that as of iOS 14.0 the AASA file will be downloaded from an Apple
CDN instead of directly from our web server. [More information here](https://developer.apple.com/videos/play/wwdc2020/10098/)

##### Useful Links 

More (possibly outdated) info might be found the [OneApp confluence page][oneapp_confluence].

[aasa_file]: https://gecgithub01.walmart.com/Torbit-Static-Origins/USGM-Static-Assets/blob/Prod/nativemobile/well-known/apple-app-site-association
[sample]: https://walmartglobal.service-now.com/nav_to.do?uri=%2Fchange_request.do%3Fsysparm_view_forced%3Dtrue%26sysparm_view%3D%26sysparm_domain_scope%3Dnull%26sysparm_domain%3Dnull%26sys_id%3D690bc1f41b6fe850a8926310604bcb6a
[oneapp_confluence]: https://confluence.walmart.com/display/USFEFER/Universal+Linking+Setup+and+Best+Practices


## URL Scheme

URL schemes are defined as part of your info.plist under [CFBundleURLSchemes](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleurltypes/cfbundleurlschemes) key. These schemes acts as qualifiers to open the app. The `DeepLinkRouter` makes an attempt to route for only those schemes which are configured. 
If you need to configure a new URL Scheme, you would need to do so in `DeepLinker+URLSchemes.swift` located in `WalmartPlatformExtensions`. Ensure that the scheme is also part of your info.plist.
Update the supported list [here](supported-list.md)

### Example

```swift
struct ExampleURLScheme: URLScheme {
    let host = "example"
    
    let description = "Example scheme to open example://path"
}
```

## Domain

The domains which are not universal links but are supposed to be part of allowed domains can be configured. A new domain can be added in `DeepLinker+Domains.swift` located in `WalmartPlatformExtensions`. By default, the subdomains are qualified as part of allowed list. However, this can be configured.

`DeepLinkRouter` will signal `externalURLHandler` if a URL matching the allowed domains is attempted to route. 
Update the supported list [here](supported-list.md)

### Example

```swift
struct ExampleDomain: Domain {
    let name = "example.com"
    let allowSubdomain = true
}
```

## QueryParameter

The query paramaters allows one to configure value so that the same will be matched against all the query parameters in the deeplink source. The matching parameters will be tracked as part of analytics. A new parameter can be added in `DeepLinker+QueryParameters.swift` located in `WalmartPlatformExtensions`. By default, the `qualifiesTracking` field will be set to false. However, this can be configured.

`DeepLinkRouter` will signal `preExecutionHandler` whenever a deeplink is successfully routed. As part of this, the `query` parameters in the resultant `DeepLink` will be matched against the configured parameters. if there is a successful match & qualifies for tracking, the `query` fields in the `DeepLink` will be tracked based on the `eventName` configured.
Update the supported list [here](supported-list.md)

### Example

```swift
struct SampleQuery: QueryParameter {
    let value = "sample"
    let description = "Sample query"
    let qualifiesTracking = true
    let eventName = "onClick"
}
```

Given a deeplink, say, `example://testRoute?option=all&source=sample`, the value `sample` matches with the query paramaters `option=all&source=sample`. Since `qualifiesTracking` is set to `true`, all the query parameters which are part of the deeplink will be tracked as part of `onClick` event.

**Note**: Ensure that the `eventName` matches one of the values in [ActionCategEnum](https://gecgithub01.walmart.com/CustomerBackbone/cbb-glass-stubgenerator-ios/blob/e3fa83c7ddc90190e79fcaf33e5015feaae942c2/sources/ActionCategEnum.swift#L5-L18) for successfull tracking 

## Supported List

The supported list of domains & schemes can be found [here](supported-list.md)

## Testing Deep Links

To test a Deep Link, you need to:

- Make sure to set up the Container properly before initializing `Routable` / executing route.
- Set up test hooks or variables to properly validate execution

The following is a snippet in the app with added description comments

```swift
class DebugPanelRoutableTests: XCTestCase {
    var container: Container!
    var debugPanelRoutable: DebugPanelRoutable!

    override func setUp() {
        super.setUp()

        // `deepLinkingContainer()` is available in the `DeepLinkingPlugin` tests and allows
        // for easily setting up a mock container with an added DeepLinker
        container = deepLinkingContainer()
        
        // initialize the routable before executing any routes
        debugPanelRoutable = DebugPanelRoutable(container: container)
    }

    func testDefaultRoute() throws {
        // route the deeplink
        _ = container.deepLinker.route(deepLink: DeepLink(url: URL(route: "debugPanel")))

        // validate conditions that the route triggers
        let debugPanelAPIMock = container.get(DebugPanelAPI.self) as! DebugPanelAPIMock

        try waitForCondition(debugPanelAPIMock.debugPanelShown)
        XCTAssertTrue(debugPanelAPIMock.debugPanelShown)
    }
}
```

## Debug Panel

A debug panel entry is available to test your  deep links and universal links
- [Open](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/debug-panel/index.md#how-to-open-the-debug-panel) the debug panel and navigate to `Deeplink 🔗 Playground 🛠`
- Enter the URL and tap on `Route` to check whether it can be routed
- The status is presented on the screen in the form of an alert or a message

## Dashboard
The diagnostic messages for the deep links can be viewed in [this](https://ce-logsearch01.prod.walmart.com/en-US/app/devices/glass_ios_diagnostic_message_dashboard?form.aVer=*&form.time-range.earliest=-7d%40h&form.time-range.latest=now&form.index=devices) dashboard

## Documenting Deep Links

Deep link URLs should be documented in [the list of supported links](supported-list.md).

## More Information

- [Mobile Universal & Deep Linking documentation](https://confluence.walmart.com/pages/viewpage.action?pageId=355386619#GlassMobileAppUniversal&DeepLinking-WhatshouldbeGlassUniversalLinkorDeeplinkforafeature.)
- [WalmartPlatform Deep Linking API](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/#deep-linking)
