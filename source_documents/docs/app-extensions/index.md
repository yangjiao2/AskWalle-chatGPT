---
title: App Extensions
navTitle: App Extensions
---

# Glass iOS App Extensions

- [What are app extensions?](#what-are-app-extesions)
- [How to integrate new App Extension?](#how-to-integrate-new-app-extension)
- [CI?](#how-to-update-CI-scripts-to-support-new-app-extension)
- [Verification](#how-to-verify-the-new-extension)

## What are app extensions?

App Extensions are an iOS feature that allows developers to extend the functionality and content of their app beyond the app itself, making it available to users in other apps or in the main operating system. Xcode provides various different App Extensions templates that can be integrated with the app as per usecase basis.
Glass iOS US app supports [6 extensions](https://gecgithub01.walmart.com/walmart-ios/glass-app/markets/usa/AppExtensions/) currently - Widgets, Rich Push Notifications, SiriIntents etc. 


This document highlights the process of integrating a new app extension, changes needed across CI scripts and App Store archiving and submission along with the newly added extension.


## How to integrate new App Extension

#### Plugin architecture for app-extension

1. Create a new plugin/use existing plugin to host all the UI and business logic needed for the extension.
2. Add a new target of desired extension template in the specific market project. Update the bundle identifier for the extension such that it is prefixed by the main app bundleId.
3. Add the plugin as a dependecy on the new extension target.
4. Create a new scheme to run the extenion independently for debugging.
5. It is a good practice to add a demo app to the plugin; this will enable you to add app-extension targets in the demo app making integration testing faster.

#### Why Plugin?
1. Overall aligns with the Glass plugin architecture where the extension logic and UI can be shared across markets.
2. Xcode does not allow to add unit tests to app-extensions. Adding the logic and UI in a plugin makes the extension code testable.

For ref - Check [Widgets Plugin](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/Widgets), [RichPush Plugin](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/RichPush)


## How to update CI scripts to support new app-extension
1. Work with MRE team to create new provisioning profiles for your extension. These will be stored and updated [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/BuildSupport).
2. You will need to update the following files in the Glass iOS App for looper to compile your new extension. Add two variables exporting the provisioning profile and the bundleId of the extension - ex: PROFILE_RICHPUSH_CONTENT and RICHPUSH_CONTENT_BUNDLE. You may refer this [PR](https://gecgithub01.walmart.com/walmart-ios/glass-app/pull/41790/files) to mimic the changes for your extension.
    1. .looper.compareappsize.yml
    2. .looper.parallel.yml
    3. .looper.sandbox.yml
    4. BuildSupport/archive-and-export-adhoc.sh
    5. BuildSupport/archive-and-export-thinning.sh
    6. BuildSupport/archive-and-export.sh
    7. BuildSupport/archive-pr-build.sh
    8. BuildSupport/exportEnterprisePlist.plist
    9. Plugins/Pharmacy/PharmacyUITests/E2ETests/BuildSupport/looper.pharmacyE2E.yml
    10. env/usa/market.properties
    11. env/usa/quick_submission_candidate.properties
    12. env/usa/release_mode_qa.properties
    13. env/usa/submission_candidate.properties
3. Create/Update the entitlements(debug and prod both) for your extension. Use exported variables created in the step above to denote the profile and bundleId in the extension's build settings.
You may refer these [PR1](https://gecgithub01.walmart.com/walmart-ios/glass-app/pull/41802/files) and [PR2](https://gecgithub01.walmart.com/walmart-ios/glass-app/pull/41805/files) to mimic the changes for your extension.
4. Update the [build number generator script](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/scripts/set_build_number.rb#L33) to automatically update the extension's build and version number to match the host app and other extensions. 
5. You will need MRE team's help to export the variables created in step 2 in [MRE repo](https://gecgithub01.walmart.com/mobile-platform/mre-tools/pull/493/files).
6. Push these changes and hope your PR passes CI! 

## How to verify the new extension?

### Local verification
1. Select the new scheme created for the extension and build.
2. You should be able to run the extension independently on simulator/device. (PS: This would need the host app to be installed on the sim/device beforehand)
3. You can also run the main app and verify the extesnion integration. 


### Debug/RC builds
1. Generate a sandbox build from [#builds-glass-ios-sandbox](https://walmart.slack.com/archives/C0133D3GG1X).
2. If the looper scripts are not correctly setup to support the new extension, sandbox build will throw errors indicating them. If so, follow the CI steps from above again.
3. Once the build is successfully generated, you can verify the app and extension functionalities on your device.

### Testflight/SC builds
1. It is a good practice to verify that the new extension is not breaking the app store archive of the app. This will cause issues only when preparing for app store submissions and would not affect debug/RC builds. 
2. Work with the MRE team to test an app archive using Release profiles(just archive, don't export or upload to testflight). This will ensure the app submission goes smoothly next cycle.
3. In case the archive fails, try checking if changes were made to all looper scripts as mentioned above.

