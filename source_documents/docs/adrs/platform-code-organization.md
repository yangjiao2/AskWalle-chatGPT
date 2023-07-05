# Platform Code Organization

- Authors: John Liedtke
- Decision: Option 2

# Context

The goal of Mobile Platform is to provide app-agnostic APIs to power Walmart's native mobile apps such as the Walmart (Glass), Health & Wellness, and international applications. 

Today, Platform code resides in both the `Plugins` and `Platform` folders in the Glass monorepo. Having two homes for Platform code engenders confusion about code ownership and jeopardizes the goal of app-agnostic APIs. For example, it is unclear that Platform APIs residing in `Plugins` should not depend on *feature* PluginAPIs. For example, `Plugins/InAppMessaging` is a Platform API living in `Plugins` and it's awfully tempting for bourgeoning platform and feature developers to modify `InAppMessaging` to depend on or to account for specific *feature* needs. 

Additionally, `WalmartPlatform` is morphing into a grab bag of APIs, utilities, and sundry conveniences rather than clearly delineated responsibilities. 


# Goals

- Define a single home for Platform API code. 
- Break up `WalmartPlatform` into distinct APIs to separate responsibilities.  
- Speed up CI by running only relevant tests. e.g. an internal change to CCM should **not** run the entire app's test suite. 
- Migrate Platform capabilities to the "PluginAPI" model where capabilities are defined through an API protocol.
- Align with Android. Android considers capabilities such as Geofence, BaseScanner, Location, etc as Platform capabilities while iOS considers them Plugins (features). 

# Options

## Option 1: Relocate and Separate Platform APIs into Targets under WalmartPlatform 

The first option is to break up `WalmartPlatform` into separate targets that live under the `WalmartPlatform.xcodeproj` and relocate existing Platform APIs that reside in `Plugins` to the project. All Platform capabilities would define their API in an interface target, `PlatformAPIs`. In addition, all utilities, _core_ functionality (e.g. `Container`, `Coordinator`), and other code that does not fit the Plugin API model would move to a new `WalmartCore` module.

```

WalmartPlatform/
├─ [Targets]/
│  ├─ PlatformAPIs/
│  │  ├─ ConfigurationAPI.swift
│  │  ├─ ...
│  ├─ WalmartCore/
│  ├─ Configuration/
│  ├─ ConfigurationTests/
│  ├─ InAppMessaging/
│  ├─ InAppMessagingTests/
│  ├─ Location/
│  ├─ LocationTests/
│  ├─ Networking/
│  ├─ NetworkingTests/
│  ├─ .../
```

### Pros

- A single home for Platform code. 
- Platform capabilities are separated into frameworks with an accompanying API. This separates concerns and removes confusion by eliminating a monolith framework. 
- Utilities and core platform functionality have a clear home, WalmartCore, and are not conflated with Platform APIs. 
- Modifying the internal implementation does not result in running the entire app's test suite. 

### Cons

- Bloated Xcode project. Each target may require 1-3 additional targets for a sample apps and unit/UI tests.
- Differs from the current Plugin model where each capability has a separate Xcode project. This may cause confusion. 
- Lots of refactoring. 


## Option 2: Create `PlatformAPIs` and follow PluginAPI Pattern

The second option is to break up `WalmartPlatform` into separate Xcode projects that live under the `Platform` folder and relocate existing Platform APIs that reside in `Plugins` to the `Platform` folder. All Platform capabilities would define their API in an interface target, `PlatformAPIs`. In addition, all utilities, _core_ functionality (e.g. `Container`, `Coordinator`), and other code that does not fit the Plugin API model would remain in `WalmartPlatform`. `WalmartPlatform` would be renamed to `WalmartCore`

```
Walmart/
├─ Platform/
│  ├─ Modules/
│  │  ├─ GlassUI/
│  │  ├─ WalmartCore/
│  ├─ APIs/
│  │  ├─ PlatformAPIs/
│  │  ├─ Geofence/
│  │  ├─ Configuration/
│  │  ├─ InAppMessaging/
│  │  ├─ .../
├─ Plugins/
│  ├─ PluginAPIs/
│  ├─ Account/
│  ├─ Ads/
│  ├─ Amends/
│  ├─ .../
```

## Pros

- A single home for Platform code. 
- Platform capabilities are separated into distinct Xcode projects/frameworks with an accompanying API. This separates concerns and removes confusion by eliminating a monolith framework. 
- Utilities and core platform functionality have a clear home, WalmartCore, and are not conflated with Platform APIs. 
- Modifying the internal implementation does not result in running the entire app's test suite. 
- A familiar and battle-tested pattern (same as Plugins).
- Avoids a monolith Xcode project.

## Cons

- Lots of refactoring. 


## Recommendation 

We recommend proceeding with Option 2 because it follows the existing Plugin architecture and provides the same benefits as option 1. 


# Decision Outcome

The team has decided to proceed with Option 2.