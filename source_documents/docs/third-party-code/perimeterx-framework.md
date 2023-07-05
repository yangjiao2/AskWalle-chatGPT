---
title: PerimeterX Framework
navTitle: PerimeterX Framework
---

# PerimeterX

- [About](#about)
- [How Are They Integrated?](#how-are-they-integrated)
- [Current Version](#current-version)
- [Updating SDK versions](#updating-sdk-versions)
- [Point of contact](#point-of-contact)
- [More Information](#more-information)

## About

[PerimeterX](https://github.com/PerimeterX/px-iOS-Framework) is used In Glass for bot detection and mitigation. The PerimeterX iOS framework works by constantly profiling and evaluating device behavior to ensure that the connections to Mobile APIs and services are genuine. 

## How Are They Integrated?

**PerimeterX.xcframework**

PerimeterX, being a dynamic framework, is embedded and linked in the main Walmart target. It is referred (`import PerimeterX_SDK.xcframework`) in the BotDetection plugin, where captcha related functionalities are handled.

## Current Version

Version [2.2.3](https://github.com/PerimeterX/px-iOS-Framework/releases/tag/2.2.3) is used in development

## Updating SDK versions

Following steps explains updating SDK versions of PerimeterX_SDK.xcframework

- **PerimeterX_SDK.xcframework**

  1) In order to update the version to, say, **x.y.z**, ensure that the PerimeterX.xcframework is available in the [repository page](https://github.com/PerimeterX/px-iOS-Framework/tree/master/PerimeterX.xcframework) and download the folder if present. Right click on the downloaded folder and opt `Compress PerimeterX_SDK.xcframework` to create a zip file.  Incase if the attachment is not found, reach out to [Point of contact](#point-of-contact) for help.

  2) Checkout the `main` branch of **[static-artifact-deployer](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer)** and create a directory in [PerimeterXSDK](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/perimeterx/PerimeterXSDK) matching the release version, ie, x.y.z

  3) Copy PerimeterX_SDK.xcframework.zip as downloaded in step 1) and paste it in this directory x.y.z

  4) Commit the change, get the PR reviewed and merge this change in `main` branch

  5. Open [Deploy Static Framework CI job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/) and click on **PerimeterX SDK** to deploy the latest framework. Once the job succeeds, scroll to `Build History`, open the latest job, navigate to details by clicking `Build Output`. Under the `Deploy Framework`, look for the uploaded URL. It should appear something like this - 

     ```
     Uploaded: 
     https://repository.walmart.com/content/repositories/pangaea_releases/com/perimeterx/PerimeterXSDK/x.y.z/PerimeterXSDK-x.y.z.zip (2.9 MB at 1.6 MB/s)
     ```

  6) Copy this URL and update it under [BinaryDependencySpecification](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/BinaryDependencySpecification/PerimeterXSDK.json#L2) and update the version of [Cartfile.resolve](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Cartfile.resolved#L15). Then run "glass environment setup" to reflect that changes locally. And Commit the change, get the PR reviewed and merge this change in `Development` branch of "glass-app".


## Point of contact

- Primary contact - [@walmart-ios/glass-platform-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-platform-ios/members)
- PerimeterX torbit config/captcha behavior - Danilo Ubaid (Email - Danilo.Ubaid@walmart.com/ Slack - @dubaid)
- SDK/Framework related queries -https://pxi.slack.com/archives/C7H0MV85N (Requires an invite - contact Danilo Ubaid)
- Slack channel for general discussions - #perimeterx-glass, #perimeterx-mobile

## More Information

- [Mocking captcha](https://confluence.walmart.com/pages/viewpage.action?pageId=369036657#PerimeterXintegrationinGlass(iOS)-Mockingcaptcha)
