# Holiday Splash Dynamic Asset Approach

 - Authors: [Hemanth Kumar](https://gecgithub01.walmart.com/h0m07gr)
 - Status: Complete
 - Decision: Finalized [Option 2: CCM used to control enable and disable the feature, Configurations are pushed through service api](images/holidaysplash/adr-tempo-holiday-splash-app-dynamic-asset.png)

 ## Introduction

 As part of the holiday season in 2022, Holiday Splash screen is designed to load the splash screen asset (lottie animation) 
 from the application bundle/resource(as static) which is built as part of the application ipa.

 ## Motivation

 If we want to change holiday splash screen animation, It requires a new version of the lottie animation 
 file incorporated into the application bundle and build and deploy a new version of the application to 
 App store with the updated asset. This has become a tedious process for the engineering and release teams. 
 Also it's very hard for the UX team to test their new animations before pushing to production.

 ## Considered Options

 ### Option 1: CCM will be updated through the BE when changes are pushed

 CCM is used to turn on/off the holiday splash screen completely and configurable parameters are retrieved through BE api, 
 and the ccm will be updated when the backend pushes the changes.

 To overcome the limitations of the mentioned static approach, It is required to re-architect to display the holiday splash screen with dynamic assets (lottie supported animation assets) during the configured interval.

 ```json dynamic splash asset configuration through CCM
 {
  "platform.event.holiday.splash.isEnabled": {
    "value": true,
    "minVersion": "24.0"
  },
  "platform.event.holiday.splash": [
    {
      "name": "Holiday Early Access Splash",                             // a.
      "id": "uuid of the splash animation",                              // b.
      "startDateTime": "Start Date in UTC format",                       // c.
      "endDateTime": "End Date in UTC format",                           // d.
      "assetUrl": "https://splashscreenasset.holiday.com/holiday1.json", // e.
      "displayTimeInterval": "display time interval in mins",            // f.
      "priority": 0                                                      // g.
    },
    {
      "name": "Holiday Black Friday Splash",
      "id": "uuid of the splash animation",
      "startDateTime": "Start Date in UTC format",
      "endDateTime": "End Date in UTC format",
      "assetUrl": "https://splashscreenasset.holiday.com/holiday2.json",
      "displayTimeInterval": "display time interval in mins",
      "priority": 0
    },
    {
      "name": "Holiday Cyber Monday Splash",
      "id": "uuid of the splash animation",
      "startDateTime": "Start Date in UTC format",
      "endDateTime": "End Date in UTC format",
      "assetUrl": "https://splashscreenasset.holiday.com/holiday3.json",
      "displayTimeInterval": "display time interval in mins",
      "priority": 0
    }
  ]
}

 Notes:
 a. Name of the splash screen
 b. Splash animation identifier
 c. The start date of the splash screen shown.
 d. The end date of the splash screen shown.
 e. The url for the Lottie file published, Should be unique.
 f. The display time interval used to decide next splash screen.
 g. Optional Integer property, used to determine which splash is shown when multiple overlap.  
 Lowest number is shown first, so long as it is eligible to be shown.
 ```
 Refer the below diagrams for download and dynamic asset and display flow:
 
 [During App startup](images/holidaysplash/adr-ccm-holiday-splash-dynamic-asset.png)
 
 [During App Foreground](images/holidaysplash/adr-ccm-holiday-splash-app-foreground.png)

 #### Pros:
     No overhead of updating CCM
     No overhead of handling holiday splash asset configuration and deployment
 #### Cons:
     Complex CCM maintenance for which CCM is not meant for doing it.
     Network failures may cause for not displaying the required holiday splash.

 ### Option 2: CCM used to control enable and disable the feature Configurations are pushed through service api

 In this approach only the enable/disable is controlled by ccm; 
 The dynamic splash configurations are fetched through the service api cached local for the first time,
 in next subsequent launch will reach from cache and display the splash screen.

 ```json CCM configuration for enable/disable of the holiday splash

 "platform.event.holiday.splash.isEnabled": {
  "value": true,
  "minVersion": "24.0"
 }

 ```

 ```json Below is the Configuration expected to receive through servie api:

 {
  "holidaySplash": [
    {
      "name": "Holiday Early Access Splash",                             // a.
      "id": "uuid of the splash animation",                              // b.
      "startDateTime": "Start Date in UTC format",                       // c.
      "endDateTime": "End Date in UTC format",                           // d.
      "assetUrl": "https://splashscreenasset.holiday.com/holiday1.json", // e.
      "displayTimeInterval": "display time interval in mins",            // f.
      "priority": 0                                                      // g.
    },
    {
      "name": "Holiday Black Friday Splash",
      "id": "uuid of the splash animation",
      "startDateTime": "Start Date in UTC format",
      "endDateTime": "End Date in UTC format",
      "assetUrl": "https://splashscreenasset.holiday.com/holiday2.json",
      "displayTimeInterval": "display time interval in mins",
      "priority": 0
    },
    {
      "name": "Holiday Cyber Monday Splash",
      "id": "uuid of the splash animation",
      "startDateTime": "Start Date in UTC format",
      "endDateTime": "End Date in UTC format",
      "assetUrl": "https://splashscreenasset.holiday.com/holiday3.json",
      "displayTimeInterval": "display time interval in mins",
      "priority": 0
    }
  ]
}
  
 Notes:
 a. Name of the splash screen
 b. Splash animation identifier
 c. The start date of the splash screen shown.
 d. The end date of the splash screen shown.
 e. The url for the Lottie file published, Should be unique.
 f. The display time interval used to decide next splash screen.
 g. Optional Integer property, used to determine which splash is shown when multiple overlap.  
 Lowest number is shown first, so long as it is eligible to be shown.

 ```
 In this approach, the holiday splash screen asset is configurable at the backend side and will be downloaded through an api 
 based on the configuration set.

 In this approach we will have 2 end points:
     1. To fetch the holiday splash configuration details
     2. To download the lottie file assets

 Refer the below diagrams for download and dynamic asset and display flow:
 
 [During App startup](images/holidaysplash/adr-tempo-holiday-splash-app-dynamic-asset.png)
 
 [During App Foreground](images/holidaysplash/adr-tempo-holiday-splash-app-foreground.png)

 #### Pros:
     1. Completely service driven
     2. No static asset maintenance.
     3. Customizations/configurations can be pushed anytime
     4. Feature enable/disable through CCM, so that we can disable the feature on any disasters.
     5. Well tested environment and consistent with other features in the application
 #### Cons:
     1. Network failures may cause for not displaying the required holiday splash.

 As part of the Backend service, Below options are studied:

 #### 1. Tempo service + Pronto for Asset content management
 This service is very well being used in the application. Glass app all home items are being loaded through Tempo.
 It can well supply configurations through Tempo and Tempo can contain the url of the dynamic asset which is deployed on to Pront service
 [Pronto Ref Document](https://confluence.walmart.com/pages/viewpage.action?spaceKey=PGPPRONTO&title=Pronto+FAQ)

 #### 2. Backend API from BE layer:
 This will be a new service to be developed. Additional overhead and cost involved in development of the service and maintenance. 
 This has a dependency on Orchestration and BE teams.

 #### 3. Pronto service integration for BE  for asset management
 This service is used for content management.

 [Pronto Ref Document](https://confluence.walmart.com/pages/viewpage.action?spaceKey=PGPPRONTO&title=Pronto+FAQ)

 #### 4. Develop Custom service
 Developing custom service by platform team.

 Based on the above study, For displaying the splash screen, Tempo + Pronto are considered as the value added option than other options.
 Tempo provides the very well-tested environment with the configuration details as many feature in the app are already depending on Tempo, where as pronto is for asset management.

 ### Platform restrictions
 1. No custom animations, we need to use  only suggested animations in to support in iOS : https://github.com/airbnb/lottie/blob/master/supported-features.md
 2. Should always align with Lottie framework version updated in the app(iOS Current version: 4.0.0)
 3. Should work in all Walmart app supported devices(iOS - iPhone and iPad), possibly different Lottie files for specific to iPhone and iPad 
 4. Since the splash screen always updates from the cache, if an update pushed to the app, the splash will start displaying from the subsequent launches
 5. Frame rate should be < 30 ps

 ### Assumptions
 1. Max 3 seconds of animation duration, if animation delay increases could cause the delay in displaying Home screen.
 2. The size should be <= ~1mb file was tested so far, Suggested to maintain the same file size.
 3. If the file size configured are supported by Pronto service
 4. We should perform cleanup on any json which is no longer represented in the CCM return
 5. Splash last-displayed dates need to be cached on a per-splash basis, so that we can support multiple splash screens whose dates overlap

 ### Open questions discussed: 

##Q: Should the display logic live in Onboarding or Animation plugin?
Ideally we would have some sort of "App Ready" controller which would allow for registration of events such as the presentation of a Splash Screen, or anything else which should occur post-onboarding and/or on app launch (if onboarded) and/or on app foreground (if onboarded).

##Q: Should we support display of the splash on foregrounding?
Yes.  Even if we disable it internally (not saying we will), we should still build this out to support it if needed.

##Q: Do we have any functionality when tapping on the splashscreen (like deeplinking to deals page)?
One concern is that we already provide tap-to-dismiss behavior on the Splash Screen.  If we add the ability to configure this behavior, it may result in confusion for users.
One solution might be to add a label which conveys the tab behavior, to the bottom of the screen.  (eg: a semi-transparent string that reads "Tap to Dismiss"). If overriding behavior is provided, we could also require that a replacement string be provided along with it.  (eg: "Tap to View Deals"). This would help to convey the resulting action to our users and reduce overall confusion.

##Q: How to prevent crash happening?
Create a Crash Logger (plugin?) which locally caches some number of crashes along with their session lengths (and any other info which we may find to be useful – but maybe not too much)
If it crashed on the last session, within a certain amount of time, we should skip the splash on the subsequent launch


