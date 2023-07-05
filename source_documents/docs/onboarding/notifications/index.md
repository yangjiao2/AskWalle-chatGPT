---
title: Notifications
navTitle: Notifications
---

# Notifications

- [Remote Notifications](#remote-notifications)
- [Providers](#providers)
- [Creating Providers](#creating-providers)
- [Provider Example](#provider-example)
- [Handling Notifications](#handling-notifications)
- [Registering Providers](#registering-providers)
- [Mock UNNotification & UNNotificationResponse](#mock-unnotification-&-unnotificationresponse)
- [Providers in Different Markets](#providers-in-different-markets)
- [Dashboard](#dashboard)

## Remote Notifications

The remote notifications are supported in Glass through [Notifications](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/Notifications) plugin. The remote notifcations can get delivered from APNS through multiple sources. The notification payload needs to be handled by individual providers

## Providers

[Providers](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Providers/Notifications) can be conceptualized as Plugins which caters the functionality related to different sources of remote notifications. Providers for notifications confirms to [NotificationsProvider](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/PluginAPIs/PluginAPIs/APIs/Notifications/NotificationsProvider.swift) to handle responsibilities like setup, registration, unregistration, parsing the notification payload, etc. Providers can depend on some of the common modules such as PluginAPIs, WalmartPlatform. Interaction with other modules should be done only through interfaces exposed through PluginAPIs.

## Creating Providers

- Provider, say `MyNotificationProvider`,  can be created by following the steps mentioned [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/architecture/plugin-modules.md#creating-a-plugin-module)  
- The new module which gets generated needs to be moved from [Plugins](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins) to [Providers/Notifications](Providers/Notifications) 
- Confirm `MyNotificationProvider` to  [NotificationsProvider](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/PluginAPIs/PluginAPIs/APIs/Notifications/NotificationsProvider.swift) 
- Similar to Plugins, the providers should derive the build settings from `.xcconfig`
- ⚠️ Inclusion of test plans as part of PR checks is tracked [here](https://jira.walmart.com/browse/CEMRE-1591).

## Provider Example

```swift
public class MyNotificationProvider {
    public init(container: Container) {
        // Handle initialisation
    }
}

extension MyNotificationProvider: NotificationsProvider {

    public func apnsDeviceToken(token: String) {
        // Access to APNS device token
    }

		public func setup(
        with application: UIApplication,
        launchOptions: [UIApplication.LaunchOptionsKey : Any]?)
    {
        // Setup your provider (optional)
    }
}
```

## Handling Notifications

The notification from every source should have a unique identifier to differenciate the notification payload from other sources. It is the responsibility of the providers to check for the presence of unique identifier in the payload, parse and pass the completion wherever necessary. Following methods provide access to the remote notification payload  


```swift
extension MyNotificationProvider: NotificationsProvider {

    public func notificationCenterWillPresent<T>(
        with provider: NotificationCenterProvider,
        notification: T,
        completionHandler: @escaping NotificationPresentationOptions,
        notificationPayloadHandler: @escaping ProcessedNotification) where T : UserNotification
    {
        let userInfo = notification.request.content.userInfo
     	  // Check for unique identifier which differentiates from other sources in payload
        guard yourUniqueIdenfierPresent(in: userInfo) else {
            return
        }
        // Handle notification and pass completion if necessary
    }

    public func notificationCenterDidReceive<T>(
        with provider: NotificationCenterProvider,
        response: T,
        completionHandler: @escaping () -> Void,
        notificationPayloadHandler: @escaping ProcessedNotification) where T : UserNotificationResponse
    {
      	let userInfo = response.notification.request.content.userInfo
     	  // Check for unique identifier which differentiates from other sources in payload
        guard yourUniqueIdenfierPresent(in: userInfo) else {
            return
        }
        // Handle notification and pass completion if necessary
    }
}
```

## Registering Providers

Providers can be registered during the startup in respective `BootstrapCoordinator` as part of registration of `NotificationsAPI`

```swift
container.registerProvidingChild(NotificationsAPI.self) {
            let notificationsAPI = NotificationsPluginAPI(container: $0)
            // Register notification providers
            let providerOne = ProviderOne(container: $0)
            let providerTwo = ProviderTwo(
                container: $0,
                configuration: MyConfiguration(environmentAPI: $0.get())
            )
            notificationsAPI.register(providers:[providerOne, providerTwo])
            return notificationsAPI
}
```



## Mock UNNotification & UNNotificationResponse 

In order to mock `UNNotification` & `UNNotificationResponse` to handle custom notifications payloads in your providers, as part of test strategy, following mock objects can be used
- [MockNotification](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/Modules/TestUtilities/MockSupport/Mocks/MockUserNotificationResponse.swift)

- [MockUserNotificationResponse](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/Modules/TestUtilities/MockSupport/Mocks/MockUserNotificationResponse.swift)

 ```swift
import MockSupport

let userInfo: [AnyHashable: Any] = [
  "title": "Walmart",
  "body": "Your order is ready for pickup!"
]
// Creates mock UNNotification
let mockNotification = MockNotification.mock(userInfo)

// Creates mock UNNotificationResponse
let mockNotificationResponse = MockUserNotificationResponse.mock(userInfo)
 ```

## Providers in Different Markets 

Following table summarises the presence of various providers in different markets

|                        Provider Name                         |         US         |       Canada       |       Mexico       |
| :----------------------------------------------------------: | :----------------: | :----------------: | :----------------: |
| [Appboy/Braze](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/third-party-code/braze-framework.md) | :white_check_mark: |        :x:         |        :x:         |
| [DMP+Comms](https://confluence.walmart.com/pages/viewpage.action?pageId=582746949) | :white_check_mark: |        :x:         | :white_check_mark: |
|                        Urban Airship                         |        :x:         | :white_check_mark: |        :x:         |
|                           Firebase                           |        :x:         |        :x:         | :white_check_mark: |

## Dashboard

The dashboard which tracks various events and states related to push notifications can be found [here](https://ce-anivia-az.prod.walmart.com/en-US/app/devices/glass_ios_push_notifications_dashboard)
