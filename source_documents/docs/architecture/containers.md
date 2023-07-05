---
title: Architecture - Containers
navTitle: Containers
---

# Containers

- [What Is A "Container"?](#what-is-a-container)
- [Container Instances](#container-instances)
- [Lazy Initialization](#lazy-initialization) (todo)

## What Is A "Container"?

A `Container` is a [Container class object](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Classes/Container.html) that lazily creates and maintains references to other objects which have been previously registered for a given associated public protocol.  It returns those references generically, based on the Type that the reference is being set or passed into.

### Container Examples

Here we use the container to register a plugin's implementation, an instance of the `FeedbackPluginAPI` class, for a given interface, the `FeedbackAPI` protocol.

```swift
// register `FeedbackPluginAPI` as the implementation for the `FeedbackAPI` interface
let container = Container()
container.register(FeedbackAPI.self, FeedbackPluginAPI.init)
```

Now we can use the container to get an instance of the feedback plugin.

```swift
// retrieve plugin instance
let feedbackAPI: FeedbackAPI = container.get()
```

The container keeps interfaces decoupled from their underlying implementations. Consumers of the `FeedbackAPI` don't have to care about which instance is providing the implementation. This allows you to inject mock instances for unit tests and demo apps.

## Container Instances

- The Walmart app target's default container object is instantiated in main AppDelegate and it's reference is passed to all the registered plugins. Contains implementations of platform APIs and plugin APIs.
- `Container.mock` - A mock container used by demo apps and test targets. Contains mock implementations of platform APIs and plugin APIs 

### Walmart App Container

The app creates a `Container` instance that serves as a registry of plugin implementations and their corresponding interfaces. The app container registers implementations for all platform APIs such as networking and analytics.

```swift
public extension Container {
    /// The shared instance for the app.
    public static let walmartApp: Container
    
    
    /// The app's analytics instance
    public var analytics: Analytics {
        self.get()
    }
    
    /// The app's standard networking instance
    public var networking: Networking {
        self.get()
    }
}
```

This container is used to register and access plugin instances and is passed to every plugin's initializer when running the main app target.

### Mock Container

A mock container is provided by the [PluginAPIsMocks](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/PluginAPIs/PluginAPIsMocks), a framework containing mock implementations for every Plugin API in the app.

The mock container is used to reference mock implementations of plugin APIs in test targets and demo apps.

```swift
public extension Container {
    static var mock: Container {
        let container = Container()

        container.register(Analytics.self, AnalyticsManager.init)
        container.register(PluginAPIs.FeedbackAPI.self, FeedbackAPIMock.init)

        return container
    }
}
```

## Lazy Initialization

TODO: Discuss best practices for lazily referencing container objects.

```
private lazy var fieldValidator: FieldValidator = container.get()

```
