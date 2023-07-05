# Multi Market Deeplink Routables

- Authors: [Drake Yundt](https://gecgithub01.walmart.com/d0y00qn)
- Decision: Option 1 Variant A. But while implementing will look into variant B. 

## Introduction

With multiple markets entering the glass ecosystem, we need a expandable way to support differences in deeplink paths for routables in different markets. 

## Motivation

Although markets like Canada and Mexico will be reusing Glass code and features, their back end systems and structure may lead them to having different deeplink paths
for the features in question. For example `walmart://account` will become `walmart.ca://myaccount`. While the scheme will be handled via a configuration injected into
the DeepLinker, the paths also need to be 

### Guiding Principles

- Routables must be configurable. 
- API must be clean and easy to use.
- Amount of refactoring required should be taken into consideration.

## Considered Options

### Option 1: App's will directly inject their supported Routable objects as protocols. 
Host applications will directly inject their supported routable objects as implementations of a protocol interface. Standard concrete implementations will be provided, which will optionally support configurations.

#### Variant A: 
Standard routable object implementations will continue to live in the Deeplinking plugin but will exposed publicly and require injection. 

##### Pros:
- Minimal Change.
- Allows configuring of routables.
- Allows custom routable implementations to be injected should the host application need to do so.

##### Cons:
- Clean api but no text completion.

##### Example:

```swift
class Container {}
class DeepLink {}
class DeepLinkNavigationState {}

class Deeplinker {
    var routables: [Routable] = []

    func register(routables: [Routable]) {
        self.routables = routables
    }
}

protocol Routable {
    var actionRoutes: [String] { get }
    var waitForPreconditions: Bool { get }
    func registerRoutes()
    func handleDeepLink(_ deepLink: DeepLink, state: DeepLinkNavigationState) -> Bool
}

protocol FeatureRoutableA: Routable {
    // Additional Requirements declared.
}

class StandardFeatureRoutableA: Routable {
    struct Configuration {
        let customPath: String
    }

    private var configuredRoutes = [
        "path1",
        "path2"
    ]

    override var actionRoutes: [String] { configuredRoutes }

    init(container: Container, configuration: Configuration) {
        super.init(container: container)

        configuredRoutes.append(configuration.customPath)
    }

    required init(container: Container) {
        super.init(container: container)
    }
}

protocol FeatureRoutableB: Routable {
    // Additional Requirements declared.
}

class StandardFeatureRoutableB: Routable {
    override var actionRoutes: [String] {
        [
           "path1",
           "path2"
       ]
    }
}

let container = Container()
let deeplinker = Deeplinker()

deeplinker.register(
    routables: [
        StandardFeatureRoutableA(
            container: container,
            configuration: .init(customPath: "customFeaturePath")
        ),
        StandardFeatureRoutableB(container: container)
    ]
)
```

#### Variant B: 
Routable objects will continue to live in the Deeplinking plugin but a enum based wrapper will be exposed and used to specify which routable objects the app supports
while also taking configuration objects as associated variables to the enum case.

##### Pros:
- Small Change
- Allows configuring of routables.
- Deeplinking api would have text completion thanks to the enum wrapper.
- Allows custom routable implementations to be injected should the host application need to do so.

##### Cons:
- Enum may be unnecessary and require additional code in the deeplinker.

##### Example:

```swift
class Container {}
class DeepLink {}
class DeepLinkNavigationState {}

class Deeplinker {
    enum Route {
        case featureA(FeatureRoutableA)
        case featureB(FeatureRoutableB)
    }

    var routables: [Routable] = []

    func register(routes: [Route]) {
        routes.forEach { route in
            switch route {
            case .featureA(let routable):
                routables.append(routable)
            case .featureB(let routable):
                routables.append(routable)
            }
        }
    }
}

protocol Routable {
    var actionRoutes: [String] { get }
    var waitForPreconditions: Bool { get }
    func registerRoutes()
    func handleDeepLink(_ deepLink: DeepLink, state: DeepLinkNavigationState) -> Bool
}

protocol FeatureRoutableA: Routable {
    // Additional Requirments declared.
}

class StandardFeatureRoutableA: Routable {
    struct Configuration {
        let customPath: String
    }

    private var configuredRoutes = [
        "path1",
        "path2"
    ]

    override var actionRoutes: [String] { configuredRoutes }

    init(container: Container, configaration: Configuration) {
        super.init(container: container)

        configuredRoutes.append(configaration.customPath)
    }

    required init(container: Container) {
        super.init(container: container)
    }
}

protocol FeatureRoutableB: Routable {
    // Additional Requirments declared.
}

class StandardFeatureRoutableB: Routable {
    override var actionRoutes: [String] {
        [
           "path1",
           "path2"
       ]
    }
}

let container = Container()
let deeplinker = Deeplinker()

deeplinker.register(
    routes: [
        .featureA(StandardFeatureRoutableA(configuration: .init(customPath: "customFeaturePath"), container: container)),
        .featureB(StandardFeatureRoutableB(container: container))
    ],
    using: container
)
```

#### Variant C: 
Standard routable object implementations will be moved to PluginAPIs, be exposed publicly and live next to their associated plugin API. They will then need to be injected into the Deeplinking 
plugin by the host application.

#### Pros:
- Allows configuring of routables.
- Routables would live close to the Plugin API definitions they utilize.
- Allows custom routable implementations to be injected should the host application need to do so.

#### Cons:
- More moving of code then the other options
- PluginApis would now contain logic (less than ideal)

#### Example:
This will look the same as Variant A, the only difference being where the Routable objects are defined.

### Option 2: Required routable configurations are injected into the Deeplinking plugin

#### Pros:
- Small change.
- Allows configuring of routables.

#### Cons:
- Configuration would be required even if the routable was not used by the host application
- Less clean api than other options.

#### Example:

```swift
class Container {}
class DeepLink {}
class DeepLinkNavigationState {}

class Routable {
    let container: Container

    // The routes for this particular route action
    var actionRoutes: [String] {
        []
    }

    var waitForPreconditions: Bool {
        true
    }

    required init(container: Container) {
        self.container = container

        registerRoutes()
    }

    private func registerRoutes() {
        container.deepLinker.addRoutes(actionRoutes, waitForPreconditions: waitForPreconditions)
        { [weak self] (deepLink, state) -> Bool in
            self?.handleDeepLink(deepLink, state: state) ?? false
        }
    }f

    func handleDeepLink(_ deepLink: DeepLink, state: DeepLinkNavigationState) -> Bool {
        false
    }
}

class Deeplinker {
    enum Route {
        case featureA(configuration: FeatureRoutableA.Configuration)
        case featureB
    }

    struct Configurations {
        let featureAConfiguration: FeatureRoutableA.Configuration
    }

    var routables: [Routable] = []

    func registerRoutes(with configurations: Configurations, using container: Container) {
        routables.append(FeatureRoutableA(container: container, configaration: configurations.featureAConfiguration))
        routables.append(FeatureRoutableB(container: container))
    }
}

class FeatureRoutableA: Routable {
    struct Configuration {
        let customPath: String
    }

    private var configuredRoutes = [
        "path1",
        "path2"
    ]

    override var actionRoutes: [String] { configuredRoutes }

    init(container: Container, configaration: Configuration) {
        super.init(container: container)

        configuredRoutes.append(configaration.customPath)
    }

    required init(container: Container) {
        super.init(container: container)
    }
}

class FeatureRoutableB: Routable {
    override var actionRoutes: [String] {
        [
           "path1",
           "path2"
       ]
    }
}

let container = Container()
let deeplinker = Deeplinker()

deeplinker.registerRoutes(
    with: .init(
        featureAConfiguration: .init(customPath: "customFeaturePath")
    ),
    using: container
)
```

## Recommended Solution

Option 1 Variant B: This solution would provide the optimal path forward. Not only would the API remain clean, the Routable object be configurable and the host application be
able to choose exactly which routables they support, but the enum wrapper would allow for very easy integration due to the added benefit of text completion in XCode.
