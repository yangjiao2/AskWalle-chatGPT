
# Generic Plugins

- Authors: [Alexandre Santos](https://gecgithub01.walmart.com/a0s028v) / [Mak](https://gecgithub01.walmart.com/m0s01p7)
- Decision: Problem 1: Configurations Protocol, Problem 2: Dependency Documentation, Problem 3: Optional plugins

## Introduction

A solution for converting the current Plugins into market-agnostic Plugins.

## Motivation

The current plugins are often making assumptions or using APIs that are specific to the Walmart US market, but with the goal of reusing features in different markets, it will be necessary to refactor the plugins in a way that they become market-agnostic, relying only on their configuration setup to execute their feature code.

These are the current problems identified so far:

1. [Plugins have hard-coded configurations, like server address.](#problem-1-hard-coded-configurations)
1. [Lack of clarity on the inter-dependency graph between the plugins](#problem-2-service-locator), since plugins usually get a `Container` instance ([Service Locator](https://en.wikipedia.org/wiki/Service_locator_pattern)) from where they try to resolve its dependencies directly. This may cause issues to consumers of isolated features, since those are hidden dependencies that are resolved only at runtime.
1. [Plugins have hard dependencies](#problem-3-market-dependent-plugin) that sometimes won't be a hard requirement for other markets (e.g: Walmart+ plugin in the `CheckoutPluginAPI` implementation)

## Problem 1: Hard-Coded Configurations

Plugins have hard-coded configurations, like server address, URLs, etc.

### Considered Options

#### Option 1: Configurations Struct/Protocol

1. Create a configuration struct or protocol for each plugin implementation in its public interface.
1. This struct/protocol would include:
	1. Local feature flags (e.g: enable/disable a UI feature)
	1. Environment details (e.g: host address)
1. Use dependency injection to inject the implementation of the configurations protocol to each plugin in its `init` method.

> *Note*: The team agreed to primarily use structs to expose the plugin configurations and in more complex cases, where more flexibility might be required, we can use a protocol to expose the public interface while providing a default struct that conforms to it.

##### Pros

- Allows other markets to quickly configure and reuse existing plugin features.
- Easy to extend the configurations protocol without making breaking API changes.
- Makes it easier to catch breaking changes, since they could be caught at compile time.
- Easy to mock for automated tests.

##### Cons

- Requires refactoring in every plugin.
- Refactoring will include breaking API changes to the public interfaces.

#### Option 2

Add a build-time configuration (`xcconfig`) file for market specific flags/values to each plugin.

##### Pros

- Less code changes when compared to option 1
- xcconfig files would be set at the container app level

##### Cons

- Incompatible with a dependency manager system, like Swift Package Manager
- Harder to mock for automated tests

### Recommended Solution

We recommend option 1, since it's compatible with any type of artifact distribution and would allow a compile-time "validation" of compatibility.

### Decision

The team agreed on Option 1.  However configuration will occur via `struct`, unless otherwise needed.

## Problem 2: Service Locator

The current strategy for [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) is by using a `Container`, which acts as a [Service Locator](https://en.wikipedia.org/wiki/Service_locator_pattern), from where plugins try to resolve their dependencies directly. This may cause issues to consumers of these plugins, since those are hidden dependencies that are resolved only at runtime.

### Considered Options

#### Option 1: Dependency Injection (Constructor)

Plugins would expose their dependencies in their `init` method, which would have parameters for each protocol type they depend on.

##### Pros

- Clear communication of required/optional dependencies to users of the module.
- Easier to catch issues at compile time.
- Easier to inject mocked dependencies for automated tests.

##### Cons

- Requires more refactoring of existing API interfaces.

#### Option 2: Dependency Injection + Configurations Protocol

Plugins would expose their dependency in a "Configurations Protocol", which would be injected in their `init` method.

##### Pros

- Keeps a simple interface to the Plugin `init` method.
- Can include other flags and configurations in the same protocol.

##### Cons

- Protocol implementations would have to be provided by every container app using the plugin.

#### Option 3: Dependency Documentation

Plugins would have their list of dependencies documented, so that any new team integrating that plugin would have a good idea of
what dependencies would be required and which ones would be optional.

##### Pros

- No code refactoring, since it would only require documentation.

##### Cons

- Not easy to catch inconsistencies, since code and documentation are not linked together.
- Not easy to automate checks.

### Recommended Solution

We agreed on option 3, as it would require no refactoring. 

### Decision

The team agreed on having two spikes:
One to investigate the best approach for injecting dependencies: [Link](https://jira.walmart.com/browse/CEPG-59680)
One to investigate the best approach of validating the plugin dependencies: [Link](https://jira.walmart.com/browse/CEPG-59676)

## Problem 3: Market Dependent Plugin

Code assumes all plugins are registered during startup. This is OK if we have only one market. 
Some market might not require few Plugins. For Ex: Walmart+ is not required for Canada. 
If Canada app does not register Walmart+, other plugin that is dependent on Walmart+ plugin will fail at runtime and crash the App.


#### Option 1: Plugin's are made optional

1. We should stop using `Container.get<Interface>` and start using `Container.getIfRegistered<Interface>`. This returns an optional Interface?
1. Market specific plugin's are made optional.

##### Pros

- Makes the plugin code more robust to handle missing optional dependencies.

##### Cons

- Requires refatoring.
- Requires handling and testing the scenarios where the optional dependencies are not registered.

#### Option 2: Mock Plugin's

1. When a market does not need a plugin, a mocked plugin is injected into the container.
1. When a plugin sees a mocked plugin the UI is not rendered on the screen.

##### Pros

- Quick fix approach.

##### Cons

- Team should be educated on this approach during onboarding.

### Recommended Solution
We recommend option 1, Plugin's are made optional.

### Decision

The team has agreed on Option 1.  However, down-stream impact should first be coordinated at a product level, to ensure now-plugins still behave in expected ways.

## Future Enhancement

We also need to consider the exposition of these plugins (feature modules) as isolated artifacts via a dependency manager like [Swift Package Manager](https://developer.apple.com/documentation/swift_packages) or [CocoaPods](https://cocoapods.org).

### Reference Links

- Inversion of Control: https://en.wikipedia.org/wiki/Inversion_of_control and https://martinfowler.com/bliki/InversionOfControl.html
- Dependency Injection: https://martinfowler.com/articles/injection.html
- [Service Locator Pattern](https://en.wikipedia.org/wiki/Service_locator_pattern)
- [Apple SPM](https://developer.apple.com/documentation/swift_packages)
- We initially had exported the "Glass Platform" components with SPM and documented it here: [SPM Dependency](https://confluence.walmart.com/display/CAMAPP/SPM+Dependency+Management). However, glass platform has been moved to the glass app mono repo.
