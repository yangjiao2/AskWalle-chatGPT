# Dependency Injection vs Validation

- Authors: [Drake Yundt](https://gecgithub01.walmart.com/d0y00qn)
- Decision: Option 3 - Hybrid approach.

## Introduction

With multiple markets entering the glass ecosystem, a better way to manage dependency injection is needed to make each 
plugin's dependency requirements more explicit. 


## Motivation

Almost every plugin in the Glass ecosystem depends on various other plugins. The current approach to injecting dependencies
is through the container. This leads to runtime crashes when a dependency is missing and a need to resolve issues 
far too late in the feed back cycle, as well as an uncertainty on what dependencies are required by a specific Plugin 
during integration. 

### Guiding Principles

- Plugins should be explicit about their required dependencies without being coupled to a concrete implementations. 
- There should not be any uncertainty about what dependencies are required by a specific plugin.
- Missing dependencies should be caught as early in the feedback cycle as possible.

## Considered Options

### Option 1: Plugins will declare a Dependencies object to be injected into the concrete implementation of their public API

This solution will ensure all plugins are explicit about their dependencies while also reducing the time to discover issues 
caused by missing dependencies. It will also keep the public api footprint of the plugin small without running into the problem of 
an over bloated initializer. The dependency object will be initialized with dependency resolvers to insure we do not over complicate the 
dependency graph.

#### Pros:
- Dependencies are explicit.
- Added and removed dependencies will cause compile failures, leading to quicker discovery and resolution.

#### Cons:
- It is still possible for plugins to be missing from the container at resolutions time.

#### Example:

```swift
public protocol PluginA {
    func doThings()
    func doSomethingWithB()
    func doSomethingWithC()
}

public protocol PluginB {
    func doThings()
    func doSomethingWithA()
    func doSomethingWithC()
}

public protocol PluginC {
    func doThings()
    func doSomethingWithA()
    func doSomethingWithB()
}

public class PluginAPIA: PluginA {
    public class Dependencies {
        lazy var pluginB: PluginB = pluginBResolver()
        lazy var pluginC: PluginC? = pluginCResolver()

        private let pluginBResolver: () -> PluginB
        private let pluginCResolver: () -> PluginC?

        public init(
            pluginBResolver: @escaping () -> PluginB,
            pluginCResolver: @escaping () -> PluginC?)
        {
            self.pluginBResolver = pluginBResolver
            self.pluginCResolver = pluginCResolver
        }
    }

    private let dependencies: Dependencies

    public init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }

    public func doThings() {
        print("\(String(describing: self)) DID SOMETHING VERY IMPORTANT")
    }

    public func doSomethingWithB() {
        let pluginB = dependencies.pluginB

        pluginB.doThings()
    }

    public func doSomethingWithC() {
        let pluginC = dependencies.pluginC

        pluginC?.doThings()
    }
}

public class PluginAPIB: PluginB {
    public class Dependencies {
        lazy var pluginA: PluginA = pluginAResolver()
        lazy var pluginC: PluginC = pluginCResolver()

        private let pluginAResolver: () -> PluginA
        private let pluginCResolver: () -> PluginC

        public init(
            pluginAResolver: @escaping () -> PluginA,
            pluginCResolver: @escaping () -> PluginC)
        {
            self.pluginAResolver = pluginAResolver
            self.pluginCResolver = pluginCResolver
        }
    }

    private let dependencies: Dependencies

    public init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }

    public func doThings() {
        print("\(String(describing: self)) DID SOMETHING VERY IMPORTANT")
    }

    public func doSomethingWithA() {
        let pluginA = dependencies.pluginA

        pluginA.doThings()
    }

    public func doSomethingWithC() {
        let pluginC = dependencies.pluginC

        pluginC.doThings()
    }
}

public class PluginAPIC: PluginC {
    public class Dependencies {
        lazy var pluginA: PluginA = pluginAResolver()
        lazy var pluginB: PluginB = pluginBResolver()

        private let pluginAResolver: () -> PluginA
        private let pluginBResolver: () -> PluginB

        public init(
            pluginAResolver: @escaping () -> PluginA,
            pluginBResolver: @escaping () -> PluginB)
        {
            self.pluginAResolver = pluginAResolver
            self.pluginBResolver = pluginBResolver
        }
    }

    private let dependencies: Dependencies

    public init(dependencies: Dependencies) {
        self.dependencies = dependencies
    }

    public func doThings() {
        print("\(String(describing: self)) DID SOMETHING VERY IMPORTANT")
    }

    public func doSomethingWithA() {
        let pluginA = dependencies.pluginA

        pluginA.doThings()
    }

    public func doSomethingWithB() {
        let pluginB = dependencies.pluginB

        pluginB.doThings()
    }
}

let container = Container()
container.register(PluginA.self) {
    PluginAPIA(
        dependencies: .init(
            pluginBResolver: { container.get() },
            pluginCResolver: { container.getIfRegistered() }
        )
    )
}

container.register(PluginB.self) {
    PluginAPIB(
        dependencies: .init(
            pluginAResolver: { container.get() },
            pluginCResolver: { container.get() }
        )
    )
}

container.register(PluginC.self) {
    PluginAPIC(
        dependencies: .init(
            pluginAResolver: { container.get() },
            pluginBResolver: { container.get() }
        )
    )
}
```

### Option 2: Plugins will declare the types they are dependent on, which will then be validated at run time.

This solution will add a much needed runtime check to ensure every plugin has it's required dependencies loaded in the container. 
This check will occur during runtime just after after app launch and bootstrapping completes. 

#### Pros:
- Adds an additional check to insure all required plugins have been registered at launch time, reducing the time taken to catch 
issues caused by missing dependencies. 

#### Cons:
- Only adds a runtime check. 

#### Example:

```swift
public protocol DependencyProviding {
    var dependencies: [Any.Type] { get }
}

public protocol PluginA: DependencyProviding {
    func doThings()
}

public protocol PluginB: DependencyProviding {
    func doThings()
}

public protocol PluginC: DependencyProviding {
    func doThings()
}

public class PluginAPIA: PluginA {
    public var dependencies: [Any.Type] = [
        PluginB.self,
        PluginC.self
    ]

    public func doThings() {
        print("\(String(describing: self)) DID SOMETHING VERY IMPORTANT")
    }
}

// Simplified for sake of example.
public class Container {
    private var registry: [String: Any] = [:]

    public func register<Interface: DependencyProviding>(
        _ interface: Interface.Type,
        _ provider: @escaping () -> Interface)
    {
        // Resolution is deferred until needed in the real implementation via service states.
        registry[String(describing: interface)] = provider()
    }

    // Will be called after dependency registration and bootstrapping.
    public func validateAllDependencies() {
        registry.values.forEach { interface in
            if let dependentInterface = interface as? DependencyProviding {
                dependentInterface.dependencies.forEach { dependency in
                    precondition(
                        // Important to not load unresolved plugins at this point but to only insure 
                        // they are registered
                        registry[String(describing: dependency)] != nil,
                        "\(interface) has not been registered in the container."
                    )
                }
            }
        }
    }
}
```

### Option 3: A combination of both dependency injection and validation.

A combination of the above two solutions would combine the best of both worlds. Not only achieving some compile time checks
for when dependencies are added and removed, but also a run time check at app launch to ensure any dependencies missing 
from the container's registry are caught as early as possible in the development feedback cycle..

## Recommended Solution

Option 3, a combination of both solutions would provide the best of both worlds and better achieve our desired goal.
