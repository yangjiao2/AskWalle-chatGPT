# Multi Provider Push Notifications

- Authors: [Diego Sepúlveda](https://gecgithub01.walmart.com/vn52glw), [Francisco Salinas](https://gecgithub01.walmart.com/vn5291h), [Patricio Sepúlveda](https://gecgithub01.walmart.com/p0s03q9), [Makeshwaran Sampath](https://gecgithub01.walmart.com/m0s01p7), [Alexandre Santos](https://gecgithub01.walmart.com/a0s028v)
- Decision: Option 5

## Introduction

Each market should be able to choose its push notification platform since there are contracts with external services and messaging platforms that are already implemented and connected with such services. This may be temporary until markets consider sharing messaging platforms.

Current situations is each market uses different framework for Push notification.

- US : App Boy/DMP
- Canada : Urban Airship
- Mexico : Firebase/DMP

## Motivation

From Glass we need to provide flexibility for each market to choose the push notification framework based on business strategy.

## Considered Options

### Option 1 - Modify only market’s App Delegate

#### Pros:

- Fast and simple, only affects the market app who needs the different push notifications service
- No need to refactor the current Notifications plugin

#### Cons:

- It doesn’t allow reuse between markets

### Option 2- All providers (SDKs) in the Notifications plugin

#### Pros:

- Reusable between different markets
- All the Notifications responsibility within the plugin

#### Cons:

- All markets need to import all the frameworks they don’t use
- Add complexity in the current Notifications plugin

### Option 3 - Abstract NotificationsProvider and inject implementation from market’s container apps (similar to Authentication plugin)

Decouple AppBoy from other components in Notifications plugin using an interface NotificationsProvider. The implementation is injected from the container app.

#### Pros:

- Reusable between different markets
- Less complexity inside the Notifications plugin

#### Cons:

- Maybe more work decoupling AppBoy

### Option 4 - Create additional plugins for different providers

#### Option 4.a - Duplicating shared logic

##### Pros:

- It doesn’t affect the current implementation
- Reusable between different markets

##### Cons:

- It doesn’t maximize reuse, some logic would need to be duplicated from current Notifications plugin (analytics, permissions, DMP related logic)

#### Option 4.b - Extracting shared logic as a library

##### Pros:

- Reusable between different markets

##### Cons:

- Need to refactor current implementation

### Option 5 - Create a wrapper (NotificationsProvider) to abstract Push Notification Services. Register the Notifications Plugin with one or more Providers

Add plugins for every Push Notification provider but keeping a Notifications plugin with the shared logic like analytics. Provider plugins need to implement a new protocol `NotificationsProvider` with methods like `registerForReceveingNotifications` and `handleNotification`.
Every market app would register only the plugins they need. A collection of providers is injected when registering the shared Notifications plugin, so the registration and notification handling can happen there without knowing the concretion of every `NotificationsProvider`.

![Multi Provider Push Notifications](images/multi-provider-push-notifications.png)

##### Pros:

- Every market uses only what they need
- Reusable between different markets
- All markets need to import only the required framework

##### Cons:

- Refactoring takes some amount of involved effort

## Recommended Solution

Option 5 - Every market uses Notifications plugin which is responsible of shared logic (analytics, user permissions, etc) and only the Provider plugins needed for that market, injected as a collection when registering Notifications plugin in container app.
This option allows flexibility to use different combinations of providers. The specific SDK and logic for registering and handling notifications is known only by the Provider plugin, so markets don't require to import all frameworks, only what they need.  
Other benefits are:

- If the format of some notification payload changes in the future, it will only require to change in the specific plugin for that Provider.
- If some market wants to leverage the same Notifications platform as other market, this option allows to do it only changing the dependencies and configurations (such as endpoints, for example).

##### Example:

```
Walmart/
├─ Plugins/
│  ├─ Notifications/
│  │  ├─ Providers/
│  │  │  ├─ AppboyNotificationsProvider
│  │  │  │  ├─ Frameworks
│  │  │  │  │  ├─ AppboyPushStoryFramework
│  │  │  │  │  ├─ Appboy_iOS_SDK
│  │  │  │  │  ├─ SDWebImage
│  │  │  │  ├─ Modules
│  │  │  │  ├─ Project
│  │  │  │  ├─ ...
│  │  │  ├─ FirebaseNotificationsProvider
│  │  │  │  ├─ Frameworks
│  │  │  │  │  ├─ FirebaseSDK
│  │  │  │  │  ├─ ...
│  │  │  ├─ UrbanAirshipNotificationsProvider
│  │  │  │  ├─ Frameworks
│  │  │  │  │  ├─ UrbanAirshipSDK
│  │  │  │  │  ├─ ...
│  │  │  ├─ DMPNotificationsProvider
│  │  │  │  │  ├─ ...
```

Providers are projects inside Notifications Plugin, but they aren't a dependency of Notifications target. Every market app depends on Notifications Plugin and the providers they want to register in Notifications. This way every market build has only the SDKs they need.

###### WalmartApp

```swift
...
import Notifications
import AppboyNotificationsProvider
import DMPNotificationsProvider
...

final class BootstrapCoordinator: Coordinator<Void> {
    override func start() {
      ...
      container.register(NotificationsAPI.self) {
          let notificationsAPI = NotificationsPluginAPI(container: $0, configuration: NotificationsConfiguration(container: $0))
          let appBoyProvider = AppboyNotificationsProvider()
          let dmpProvider = DMPNotificationsProvider()
          notificationsAPI.register(providers: [appBoyProvider, dmpProvider])
          return notificationsAPI
      }
      ...
    }
}
```

###### Notifications Plugin

```swift
public protocol NotificationsProvider {

    func handleNotification(...)
    func registerForReceveingNotifications(...)

}

public class NotificationsPluginAPI: NotificationsAPI {
    ...
    let providers: [NotificationsProvider]
    ...
    public func register(providers: [NotificationsProvider]) {
        self.providers = providers
    }
    ...
}

// Coordinator handle notification reception
public class NotificationsStartupCoordinator: Coordinator<Void> {
  let providers: [NotificationsProvider]

  init(container: Container,
       providers: [NotificationsProvider]) {
      ...
      self.providers = providers
  }

  func notificationProvider<T: UserNotification>(
        _ provider: NotificationCenterProvider,
        willPresent notification: T,
        withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void)
    {
        ...
        let userInfo = notification.request.content.userInfo

        for provider in configuration.providers {
            provider.handleNotification(...userInfo...)
        }
    }
}
```

###### FirebaseNotificationsProvider

```swift
// Firebase provider with its own implementation
public class FirebaseNotificationsProvider: NotificationsProvider {

  public func registerForReceivingNotifications() {
    ...
  }

  public func handleNotification(with userInfo: [AnyHashable : Any]) {
    let user = userInfo
    // detect payload and add logic here
    ...
  }
}
```

###### AppboyNotificationsProvider

```swift
// Appboy provider with its own implementation
public class AppboyNotificationsProvider: NotificationsProvider {

  public func registerForReceivingNotifications() {
    ...
  }

  public func handleNotification(with userInfo: [AnyHashable : Any]) {
    let user = userInfo
    // detect payload and add logic here
    ...
  }
}
```
