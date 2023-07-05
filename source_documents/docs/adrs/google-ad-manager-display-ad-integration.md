# Google Ad Manager SDK integration

- Authors: [Utkarsh Sharma](https://gecgithub01.walmart.com/vn52736), [Mohd Salman](https://gecgithub01.walmart.com/m0k049y)
- Status: Finalized
- Decision: Option 2

## Introduction
Solution for integrating Google Ad Manager(GAM) for international market. Google Ad Manager will be responsible for fetching display AD

Canada and Mexico markets should be able to fetch Ads from the GAM sdk without affecting other markets.
Current situations is each market uses SWAG server for fetching Ads.

- US : SWAG server
- Canada : GAM
- Mexico : GAM

##Motivation

US has got conditional approval from Google for S2S Ads integration, which is not applicable for international markets. Hence we have to integrate Google Ad Manager in client side for fetching display ADs.

## Considered Options

### Option 1- All providers (SDKs) in the Ads plugin

1. Add GoogleMobileAds sdk in the Frameworks folder and import it inside the Ads plugin.
2. Create market level environment variable in AdsConfiguration which will distinguish whether we need to call GAM Library or Swag server.
3. Create custom native ADs format in Google Ad Manager for fetching the display ADs.
4. Handle view and click impression from client side.
5. Create campaign in GAM dashboard.
6. Create a CCM Switch to change flow between SWAG vs GAM

#### Pros:

- Easy to integrate GAM Library in every market.

#### Cons:

- All markets need to import google sdk. so US is also dependent on GAM library which increases the app size.

### Option 2 - Integrate GAM Library in separate plugin

##### Pros:

- It doesn't affect the current implementation of US market.
- US market will not depend on GAM Library. Hence app size will not increase for US market.
- Reusable between different markets

##### Cons:
- Refactoring takes some extra effort

## Recommended Solution

Option 2 - Every market uses Ads plugin which is responsible of shared logic and only the GAM Ads plugin is injected as a collection when registering Ads plugin in container app. So markets don't require to import all frameworks, only what they need.

##### Example:

```
Walmart/
├─ Plugins/
│  ├─ Ads/
│  │  ├─ Modules/
│  │  │  ├─ GAMAds/
│  │  │  │  ├─ Frameworks
│  │  │  │  │  ├─ GoogleMobileAds
│  │  │  │  ├─ Modules
│  │  │  │  ├─ Project
│  │  │  │  ├─ ...
```

GAM Ad is a project inside ADs Plugin, but it is not a dependency of Ads target. Every market app depends on Ads Plugin. This way every market build has only the SDKs they need.

###### WalmartApp (CA and MX)

```swift
...
import Ads
import GAMAds
...

final class BootstrapCoordinator: Coordinator<Void> {
    override func start() {
      ...
        container.registerProvidingChild(GAMAdsAPI.self, GAMAdsPluginAPI.init)
      ...
    }
}
```

###### PluginAPIs Plugin

```swift

public protocol GAMAdsAPI {

    func getAdsForAdType(completionHandler:@escaping ((AdModel.DisplayAd?) -> Void))
    func performClickImpressionOnAdElement(element:String)

}
```

### Reference Links

- [Confluence Link](https://confluence.walmart.com/display/OIT/Glass+-+Display+Ads+ADR+for+Mobile+Apps)
- [GAM Link](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/quick-start)