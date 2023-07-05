# Market Deep Link Extensions

 - Authors: [Hemanth Kumar](https://gecgithub01.walmart.com/h0k01sl)
 - Status: Pending
 - Decision: Pending

 ## Introduction

 Solution for allowing different markets to extend existing functionality by defining their own 
 deep link patterns for existing deep links.

 ## Motivation

 Glass has a set of default deep links that provide entry points into various features.  Because of
 various constraints, some markets are unable to redefine existing links to align with the current
 deep link paths defined in glass.  Therefore, we need to provide a mechanism to allow markets to extend
 Glass' current linking strategy to support their existing linking strategy.

 ### Guiding Principles

 - Changes should only have to occur in the modules owned by the market
 - Market Engineers need to know as little as possible about the underlying routing
 - Telemetry can be added at the root level to track any linking failure

 ## Considered Options

 ### Option 1. Provide routing on the backend that will redirect market URLs to Glass supported URLs

 This solution is driven purely from backend configuration.  URLs are rewritten into glass-supported
 URLs

 #### Pros:
 - No change is required to any front end code
 - Market configuration is handled solely by the market

 #### Cons:
 - Causes load issues on the backend resources to server and rewrite every URL that comes in
 - Does not support rewrites for app driven URLs.  For example, custom app schemes such as `walmart-ca://`
 can't be rewritten on the backend
 
 ### Option 2. For each individual markets having their own `DeepLinker` to perform their jobs similar to `DeepLinkRouter` in walmart platform (ex: `CustomDeepLinkRouter`).

 This solution is driven purely from each markets and their own implementations. 

 #### Pros:

 - This requires some cleanup in `DeepLinker`
 - Market configuration is handled solely by the market, 
 - Deeplinking business logic implementation responsibility entirely on each market

  #### Cons
 - No control on deeplinks and its business logic
 - Duplicate code platform vs each market

  #### Example

  ```
  extension Container {
     //Calling this method while initialising the container
     public func setupCustomDeepLinker() {
         registerProvidingChild(DeepLinker.self) { (container) in
             let router = CustomDeepLinkRouter()
             router.configurePreExecution(container: container)
             router.configureSuccessRouteHandler(container: container)
             router.configurePreconditionUI()
             router.configureUniversalLinks()
             router.configureAllowedDomains()
             router.configureURLSchemes()
             router.configureExternalURLHandler(container: container)
             router.configureFailedURLHandler(container: container)
             return router
         }
         // Manages Reflector Cookie
         registerProvidingChild(ReflectorManager.self, ReflectorManager.init)
     }
  }
  
  /// See `DeepLinker` for documentation.
  public class CustomDeepLinkRouter: DeepLinker {
     var tabBarControllerProvider: (() -> UITabBarController)? { get set }

     var preExecutionHandler: ((DeepLink) -> Void)? { get set }

     var successRouteHandler: ((DeepLink) -> Void)? { get set }

     var externalURLHandler: ((_ url: URL) -> Void)? { get set }

     var preconditionLoadingStateUIHandler: PreconditionLoadingStateUIHandler? { get set }

     var universalLinks: [UniversalLink] = []

     var urlSchemes: [URLScheme] = []

     var allowedDomains: [Domain] = []

     var unableToRouteHandler: ((_ url: DeepLink) -> Void)?

     var debugInfo: [RouteDebugInfo]

     func canRoute(deepLink: DeepLink) -> Bool { }

     func route(deepLink: DeepLink) -> Bool { }

     func addRoute(_ routePattern: String,
                 waitForPreconditions: Bool,
                 handler: ((DeepLink, DeepLinkNavigationState) -> Bool)?) { }

     func addDeeplinkPreconditionHandler(identifier: String, handler: @escaping DeepLinkPreconditionHandler) { }

     func removeDeeplinkPreconditionHandler(identifier: String) { }
  }
  ```


### Option 3. Each Market Implements `CustomDeeplinkRouteProvider` for all links that need to be rewritten
Each market would supply a CustomDeeplinkRouteProvider for the links they'd want to rewrite into links supported by glass.

```

//Helps to each market to define and implement its own deeplink domain, universal links and schemes
public protocol CustomDeeplinkRouteProvider {

    /// Defines the domains as part of allowed list
    var domains: [Domain] { get }

    // Defines the configuration for handling a universal links.
    var universalLinks: [UniversalLink] { get }

    /// Defines the URL schemes
    var urlSchemes: [URLScheme] { get }
}

//Implementing CustomDeeplinkRouteProvider protocol
public struct DummyDeeplinkRouteProviding: CustomDeeplinkRouteProvider {
    public var domains: [Domain] { [CADomain()] }
    
    public var universalLinks: [UniversalLink] { [CAUniversalLink()] }
    
    public var urlSchemes: [URLScheme] { [] }

    public init() { }
}
```

#### Pros
 * Easy to instrument with telemetry

#### Cons
 * A bug could inadvertently overwrite a default supported URL
 * Required few platform change
```
//Register the container dependency
container.register(CustomDeeplinkRouteProvider.self, { DummyDeeplinkRouteProviding() })

//import the changes to from CustomDeeplinkRouteProvider to DeepLinker
public extension DeepLinker {
    func configureCustomDeeplinkRouteProvider(container: Container) {
        let extraDomainProvider: CustomDeeplinkRouteProvider = container.get()
        self.allowedDomains.append(contentsOf: extraDomainProvider.domains)
        self.universalLinks.append(contentsOf: extraDomainProvider.universalLinks)
        self.urlSchemes.append(contentsOf: extraDomainProvider.urlSchemes)
    }
}

//change the varable walmartSchemeURL to function to accept custom hosts, and invoke this method from UniversalLink subclass under method deepLinkURL on each market respectively

func walmartSchemeURL(with customHost: String = "") -> URL? {
    //////
    //////
    } else if absoluteString.contains("\(scheme)\(customHost)/") {
        absolutePath = absoluteString.replacingOccurrences(of: "\(scheme)\(customHost)/",
                                                           with: "")
    }

    return URL(string: "\(Constants.oneHallwayScheme)://\(absolutePath)")
}
```
## Recommended Solution

 Option 3 - Each Market Implements `CustomDeeplinkRouteProvider` for all links that need to be rewritten which allows markets to maximise configurability of supported links while still enforcing compile time checks to ensure correctness and consistency.
