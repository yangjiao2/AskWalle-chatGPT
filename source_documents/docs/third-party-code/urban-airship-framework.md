---
title: Urban Airship Framework
navTitle: Urban Airship Framework
---

# Urban Airship

- [Urban Airship](#urban-airship)
  - [About](#about)
  - [How Are They Integrated?](#how-are-they-integrated)
  - [Current Version](#current-version)
  - [Importing all Urban Airship modules at once](#importing-all-urban-airship-modules-at-once)
  - [Creating looper Jobs SDK versions (only the first time)](#creating-looper-jobs-sdk-versions-only-the-first-time)
  - [Updating/Uploading SDK versions](#updatinguploading-sdk-versions)
  - [Point of contact](#point-of-contact)
  - [More Information](#more-information)

## About

Glass leverages [Urban Airship](https://docs.airship.com/platform/ios/getting-started/) for marketing campaigns to send push notifications and rich notifications. Glass iOS app uses **xcframeworks** of [Urban Airship SDK](https://github.com/urbanairship/ios-library/releases) through [frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks) pulled from [static artifacts](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/airship) as dependencies.

## How Are They Integrated?

Urban Airship SDK is integrated with the main Walmart.ca target.

- **Urban Airship Core**
  Push messaging features including channels, tags, named user and default actions. This is the main framework used to receive push notifications. This is a dynamic framewokr and is embedded and linked within the main target. In order to make use of it in the code it is necessary to import it as (`import AirshipCore`). This framework is only referenced within [AirshipProvider](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Providers/Notifications/AirshipProvider/AirshipProvider/Sources/Provider/AirshipProvider.swift) which will be the responsible of handling the push notifications.
- **Urban Airship Basement**
  Required by AirshipCore. This framework is implicitly imported when importing AirshipCore. i.e. `import AirshipCore`.

## Current Version

Version [16.7.0](https://github.com/urbanairship/ios-library/releases) is used in development

## Importing all Urban Airship modules at once

Another option to use the SDK is to import everything at once.
```
https://repository.walmart.com/content/repositories/pangaea_releases/com/airship/airship/16.7.0/airship-16.7.0.zip
```
However, in order to avoid interface segregation, independent frameworks are being used.
```
https://repository.walmart.com/content/repositories/pangaea_releases/com/airship/core/16.7.0/core-16.7.0.zip
https://repository.walmart.com/content/repositories/pangaea_releases/com/airship/basement/16.7.0/basement-16.7.0.zip
```

## Creating looper Jobs SDK versions (only the first time)

Create a new job in looper in order to be able to deploy that artifact. in [.looper.yml](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/blob/main/.looper.yml) add the corresponding job.

For example in order to add AirshipCore.xcframework and AirshipBasement.xcframework jobs the following lines are necessary
```
  Airship-Basement(ui Airship Basement):
    - (name Deploy Framework) ./deploy-framework.sh com.airship basement 16.7.0 repository/com/airship/basement/16.7.0/AirshipBasement.xcframework.zip
  Airship-Core(ui Airship Core):
    - (name Deploy Framework) ./deploy-framework.sh com.airship core 16.7.0 repository/com/airship/core/16.7.0/AirshipCore.xcframework.zip
```

## Updating/Uploading SDK versions

The following steps explain how to update Urban Airship SDK: AirshipBasement,xcframework and AirshipCore.xcframework. The same procedure applies for updating the the compilation of frameworks.

- **AirshipCore.xcframework**
  
  1) Download the necessary framework compilation from the [releases page](https://github.com/urbanairship/ios-library/releases).
  
  2) Unzip the compilation of frameworks and select the one that is going to be updated from the list of available frameworks.
  
  3) Checkout the `main` branch of **[static-artifact-deployer](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer)** and create a directory in [airship](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/airship/airship) matching the release version, ie, 16.7.0 for the compilation of frameworks.
  Another option is to create a directory based on the framework final end, for example AirshipCore would be core. Repeat this for all the individual frameworks e.g. [core](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/airship/core) and create the corresponding directory 16.7.0, [basement](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/airship/core) and create the corresponding directory 16.7.0

  4) Copy the corresponding framework(s) e.g. core-16.7.0.xcframework.zip as downloaded and paste it in this directory 16.7.0. The same process applies for the compilation of frameworks.

  5) Commit the change, get the PR reviewed and merge this change in `main` branch

  6) Open [Deploy Static Framework CI job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/) and click on the corresponding job e.g.**Airship Core** to deploy the latest framework. Once the job succeeds, scroll to `Build History`, open the latest job, navigate to details by clicking `Build Output`. Under the `Deploy Framework`, look for the uploaded URL. It should appear something like this - 

     ```
     Uploaded: https://repository.walmart.com/content/repositories/pangaea_releases/com/airship/core/16.7.0/core-16.7.0.zip (15 MB at 4.2 MB/s) 
     ```

  7) Copy this URL and update it under [Frameworks](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Frameworks). Make sure not to erase the end of file charecter in that file.

  8) After every `glass environment setup` the frameworks will be downloaded from the artifacts repository.

## Point of contact

- Primary contact - [@walmart-ios/glass-canada-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-canada-ios/members)
- Walmart Canada Leads - [Alexandre Oliveira](https://walmart.slack.com/archives/D031G8LPUGH) & [Makeshwaran Sampath](https://walmart.slack.com/archives/D02TK712VME)
- SDK/Framework related queries - support.airship.com

## More Information

- [Urban Airship Integration Document](https://docs.airship.com/platform/)
- [Urban Airship iOS Developer Guide](https://docs.airship.com/platform/ios/getting-started/)
