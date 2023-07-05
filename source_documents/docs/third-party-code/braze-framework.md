---
title: Braze Framework
navTitle: Braze Framework
---

# Appboy/Braze

- [About](#about)
- [How Are They Integrated?](#how-are-they-integrated)
- [Current Version](#current-version)
- [Updating SDK versions](#updating-sdk-versions)
- [Point of contact](#point-of-contact)
- [More Information](#more-information)

## About

Glass leverages [Braze](https://www.braze.com/docs/) for marketing campaigns to send push notifications and rich notifications. Glass iOS app uses **xcframeworks** of [Appboy_iOS_SDK](https://github.com/appboy/appboy-ios-sdk/releases) and [AppboyPushStoryFramework](https://github.com/appboy/appboy-ios-sdk/releases) through [frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks) pulled from [static artifacts](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/appboy) as dependencies. [SDWebImage](https://gecgithub01.walmart.com/walmart-ios/glass-sdwebimage) is a prerequisite for Appboy_iOS_SDK.

## How Are They Integrated?

Appboy/Braze SDK is integrated with the main Walmart target. 

- **Appboy_iOS_SDK.xcframework**
  This is the core SDK to handle [push + rich notifications](https://www.braze.com/docs/developer_guide/platform_integration_guides/ios/push_notifications/). Appboy_iOS_SDK, being a dynamic framework, is embedded and linked in the main target. It is referred (`import Appboy_iOS_SDK`) in the Notifications plugin, where Appboy related push notifications are handled.
- **SDWebImage**
  This is a prerequisite for Appboy_iOS_SDK. SDWebImage, being a dynamic framework, is embedded and linked in the main target. It is referred internally within Appboy_iOS_SDK. 
- **AppboyPushStoryFramework.xcframework**
  This handles [Push Stories](https://www.braze.com/docs/developer_guide/platform_integration_guides/ios/push_story/). AppboyPushStoryFramework, being a static framework, is linked in the main target. It is referred (`import AppboyPushStory`) in the WalmartNotificationContent extension, where Appboy notification content is handled.

## Current Version

Version [4.4.1](https://github.com/Appboy/appboy-ios-sdk/releases/tag/4.4.1) is used in development

## Updating SDK versions

Following steps explains updating SDK versions of Appboy_iOS_SDK.xcframework, SDWebImage.xcframework and AppboyPushStoryFramework.xcframework

- **Appboy_iOS_SDK.xcframework**

  1) In order to update the version to, say, **x.y.z**, ensure that the Appboy_iOS_SDK.xcframework.zip is attached in the [release page](https://github.com/Appboy/appboy-ios-sdk/releases/) under the assets of **x.y.z** tag and download the zip if present. Incase if the attachment is not found, reach out to support@braze.com for help. Note that Braze maintains two repos, [Appboy](https://github.com/Appboy/appboy-ios-sdk/releases/) and [Braze](https://github.com/braze-inc/braze-ios-sdk/releases/), but the contents and releases should remain identical. 

  2) Checkout the `main` branch of **[static-artifact-deployer](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer)** and create a directory in [iossdk](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/appboy/iossdk) matching the release version, ie, x.y.z

  3) Copy Appboy_iOS_SDK.xcframework.zip as downloaded in step 1) and paste it in this directory x.y.z

  4) Commit the change, get the PR reviewed and merge this change in `main` branch

  5) Open [Deploy Static Framework CI job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/) and click on **AppBoy iOS SDK** to deploy the latest framework. Once the job succeeds, scroll to `Build History`, open the latest job, navigate to details by clicking `Build Output`. Under the `Deploy Framework`, look for the uploaded URL. It should appear something like this - 

     ```
     Uploaded: https://repository.walmart.com/content/repositories/pangaea_releases/com/appboy/iossdk/x.y.z/iossdk-x.y.z.zip (15 MB at 4.2 MB/s) 
     ```

  6) Copy this URL and update it under [Frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks)

- **SDWebImage.xcframework**

  1) Checkout [glass-sdwebimage](https://gecgithub01.walmart.com/walmart-ios/glass-sdwebimage) and ensure the `main` branch is updated with the latest release on the original [SDWebImage](https://github.com/SDWebImage/SDWebImage/releases) repo.

  2) To build xcframework, open `SDWebImage.xcodeproj`, choose the `SDWebImage XCFramework` target. Then just click `Build` (Command + R). You'll start to build all individual platform's framework and produce a `xcframework` product on your `build` directory. After all the builds succeed, Finder will pop-up and select this file.

     Refer [this](https://github.com/SDWebImage/SDWebImage/wiki/Installation-Guide#build-sdwebimage-as-xcframework) guide for more details 

  3) In order to update the version to, say, **x.y.z**, checkout the `main` branch of **[static-artifact-deployer](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer)** and create a directory in [sdwebimage](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/sdwebimage) matching the release version, ie, x.y.z
     copy the SDWebImage.xcframework.zip obtained from step 2) and paste it in this directory x.y.z

  4) Commit the change, get the PR reviewed and merge this change in `main` branch

  5) Open [Deploy Static Framework CI job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/) and click on **SDWebImage** to deploy the latest framework. Once the job succeeds, scroll to `Build History`, open the latest job, navigate to details by clicking `Build Output`. Under the `Deploy Framework`, look for the uploaded URL. It should appear something like this - 

     ```
     Uploaded: 
     https://repository.walmart.com/content/repositories/pangaea_releases/com/sdwebimage/sdwebimage/x.y.z/sdwebimage-x.y.z.zip (9.9 MB at 1.8 MB/s)
     ```

  6) Copy this URL and update it under [Frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks)

- **AppboyPushStoryFramework.xcframework**

  1) In order to update the version to, say, **x.y.z**, ensure that the AppboyPushStoryFramework.xcframework.zip is attached in the [release page](https://github.com/Appboy/appboy-ios-sdk/releases/) under the assets of **x.y.z** tag and download the zip if present. Incase if the attachment is not found, reach out to support@braze.com for help. Note that Braze maintains two repos, [Appboy](https://github.com/Appboy/appboy-ios-sdk/releases/) and [Braze](https://github.com/braze-inc/braze-ios-sdk/releases/), but the contents and releases should remain identical. 

  2) Checkout the `main` branch of **[static-artifact-deployer](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer)** and create a directory in [pushstory](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/appboy/pushstory) matching the release version, ie, x.y.z

  3) Copy AppboyPushStoryFramework.xcframework.zip as downloaded in step 1) and paste it in this directory x.y.z

  4) Commit the change, get the PR reviewed and merge this change in `main` branch

  5) Open [Deploy Static Framework CI job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/) and click on **AppBoy PushStory** to deploy the latest framework. Once the job succeeds, scroll to `Build History`, open the latest job, navigate to details by clicking `Build Output`. Under the `Deploy Framework`, look for the uploaded URL. It should appear something like this - 

     ```
     Uploaded: 
     https://repository.walmart.com/content/repositories/pangaea_releases/com/appboy/pushstory/x.y.z/pushstory-x.y.z.zip (2.6 MB at 672 kB/s)
     ```

  6) Copy this URL and update it under [Frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks)
  
- **ABKPageView.nib**

  1. As per the [Braze integration document](https://www.braze.com/docs/developer_guide/platform_integration_guides/ios/push_notifications/push_story/#manual-integration), `ABKPageView.nib` is required for Push stories. Inorder to update the latest nib corresponding to the release version, say, **x.y.z**, download [AppboyPushStory.zip](https://github.com/appboy/appboy-ios-sdk/releases) tagged under the release **x.y.z**. 
  2. Unzip the downloaded `AppboyPushStory.zip` and copy `ABKPageView.nib` found under `Resources` folder
  3. Paste the updated ABKPageView.nib in [WalmartNotificationContent/Resources/](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/WalmartNotificationContent)

## Point of contact

- Primary contact - [@walmart-ios/glass-platform-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-platform-ios/members)
- CRM lead - Namrita Mittal (Email - namrita.mittal@walmart.com/ Slack - @namritamittal)
- SDK/Framework related queries - support@braze.com
- Slack channel for general discussions - #glass-braze-push

## More Information

- [Braze integration document](https://confluence.walmart.com/pages/viewpage.action?pageId=453984780)
- [Braze developer guide](https://www.braze.com/docs/developer_guide/platform_integration_guides/ios/initial_sdk_setup/installation_methods/manual_integration_options/)

