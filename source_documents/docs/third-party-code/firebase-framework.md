---
title: Firebase Framework
navTitle: Firebase Framework
---

# Firebase

- [About](#about)
- [How Are They Integrated?](#how-are-they-integrated)
- [Current Version](#current-version)
- [Updating SDK versions](#updating-sdk-versions)
- [Frameworks in Different Markets](#frameworks-in-different-markets)
- [Static Frameworks](#static-frameworks)
- [Point of contact](#point-of-contact)
- [More Information](#more-information)

## About

Firebase is an app development platform with tools to build, grow and monetize our Glass app. More information about Firebase can be found on the [official Firebase website](https://firebase.google.com/). Glass iOS app uses different **xcframeworks** of Firebase platform and pulled from [Firebase Carthage Binary](https://github.com/firebase/firebase-ios-sdk/blob/master/Carthage.md) as dependencies.

## How Are They Integrated?

Firebase SDK's integrated with the main Walmart(as per the Market) target and CrashLogger plugin. 

- **FirebaseAnalytics.xcframework**
  The SDK automatically captures certain key events and user properties, and we can define our own custom events to measure the things that uniquely matter to Glass App. This must always be included, since automatically captures certain key events. It is referred internally within Firebase SDK's. 
- **FirebaseCore.xcframework**
  This is the core SDK which enables connecting to multiple Firebase dependency, is embedded and linked in the main target.
- **FirebaseCoreInternal.xcframework**
  This is a prerequisite for FirebaseCore. It is embedded and linked in the main target. It is referred internally within FirebaseCore.
- **FirebaseCrashlytics.xcframework**
  This is a lightweight, realtime crash reporter that helps to track, prioritize, and fix stability issues that erode our Glass app quality. It is embedded and linked in the main target.
- **FirebaseInstallations.xcframework**
  This is a service that allows to manage the installation of Glass app on a user's device. It is embedded and linked in the main target.
- **FBLPromises.xcframework**
  This is a modern framework that provides a synchronization construct for Swift and Objective-C. It is embedded and linked in the main target.
- **GoogleDataTransport.xcframework**
  This is a prerequisite for Firebase SDKs. It is embedded and linked in the main target. It is referred internally within Firebase SDKs.
- **GoogleUtilities.xcframework**
  This is a prerequisite for Firebase SDKs. It is embedded and linked in the main target. It is referred internally within Firebase SDKs.
- **nanopb.xcframework**
  This is a prerequisite for Firebase SDKs. It is embedded and linked in the main target. It is referred internally within Firebase SDKs.
- **FirebaseMessaging.xcframework**
  Firebase Cloud Messaging (FCM) is a cross-platform messaging solution, is a prerequisite for Firebase SDKs. It is embedded and linked in the main target.
- **FirebaseRemoteConfig.xcframework**
  This is a cloud service that lets change the behavior and appearance of the app. It is embedded and linked in the main target.
- **FirebaseStorage.xcframework**
  This is a Cloud Storage, is embedded and linked in the main target.

## Current Version

Version [10.5.0](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/BinaryDependencySpecification/FirebaseCrashlyticsBinary.json#L7) is used in development

## Updating SDK versions

Following steps explains updating Firebase SDKs

  1) Download the latest binaries:
      - [FirebaseAnalyticsBinary.json](https://dl.google.com/dl/firebase/ios/carthage/FirebaseAnalyticsBinary.json)
      - [FirebaseCrashlyticsBinary.json](https://dl.google.com/dl/firebase/ios/carthage/FirebaseCrashlyticsBinary.json)
      - [FirebaseMessagingBinary.json](https://dl.google.com/dl/firebase/ios/carthage/FirebaseMessagingBinary.json)
      - [FirebaseRemoteConfigBinary.json](https://dl.google.com/dl/firebase/ios/carthage/FirebaseRemoteConfigBinary.json)
      - [FirebaseStorageBinary.json](https://dl.google.com/dl/firebase/ios/carthage/FirebaseStorageBinary.json)

  2) Copy the downloaded binary json files and paste it in this directory [BinaryDependencySpecification](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/BinaryDependencySpecification)

  3) Update the [Cartfile](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Cartfile#L5...L9) with latest version of Firebase SDKs as specified in **BinaryDependencySpecification**.

  4) Run the this command `glass environment setup --update` to resolve the latest version of Firebase SDKs in [Cartfile.resolved](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Cartfile.resolved#L11...L15) 

  5) Compile the app to ensure there are no new warning/compile issues generated due to these new versions of Firebase SDKs.

  6) Create a PR with the changes in _BinaryDependencySpecification_, _Cartfile_ and _Cartfile.resolved_.

## Frameworks in Different Markets 

Following table summarises the presence of various frameworks in different markets

|      Provider Name      |         US         |       Canada       |       Mexico       |       Chile        |        B2B         |       Bodega       |
| :---------------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: |
| FirebaseAnalytics       | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FirebaseCore            | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FirebaseCoreInternal    | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FirebaseCrashlytics     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FirebaseInstallations   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FBLPromises             | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| GoogleDataTransport     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| GoogleUtilities         | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| nanopb                  | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: |
| FirebaseMessaging       |         :x:        |        :x:         | :white_check_mark: | :white_check_mark: |         :x:        | :white_check_mark: |
| FirebaseRemoteConfig    |         :x:        |        :x:         | :white_check_mark: |        :x:         |         :x:        | :white_check_mark: |
| FirebaseStorage         |         :x:        |        :x:         | :white_check_mark: |        :x:         |         :x:        | :white_check_mark: |

## Static Frameworks

 Note that the Firebase frameworks in the distribution include static libraries. While it is fine to link these into apps, it will generally not work to depend on them from wrapper dynamic frameworks.

## Point of contact

- Primary contact - [@walmart-ios/glass-platform-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-platform-ios/members)

## More Information
- [Add Firebase to your Apple project](https://firebase.google.com/docs/ios/setup)
- [Firebase Crashlytics](https://firebase.google.com/docs/crashlytics/get-started?platform=ios)
- [Google Analytics](https://firebase.google.com/docs/analytics/get-started?platform=ios)
- [Firebase Carthage](https://github.com/firebase/firebase-ios-sdk/blob/master/Carthage.md)
- [Firebase Release Notes](https://firebase.google.com/support/releases)
