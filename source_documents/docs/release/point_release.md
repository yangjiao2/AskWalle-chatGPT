---
title: Creating A Point Release
navTitle: Point Release
---

# Creating a Point Release
Sometimes you need to quick hotfix. Here are some steps on how to create one:


## Update Glass App

Using the existing `release/YY.VV` branch, you'll need to update the version
 number. No new release branch needs to be created.

Update the version number in the `glass-app` project.
* Choose the Walmart Project file
* Select Project settings 
* Select "Build Settings"
* Update Marketing version (`MARKETING_VERSION`).

At this point you may want/need to run the xcode proj cleanup script.
`./scripts/xcodeproj_verifications/_cleanup_projects.py`

Commit and push your fix make sure it lands to the correct release branch.


### Glass App Builds
Check on looper for builds of your release branch:
https://ci.mobile1.walmart.com/job/glass-mobile/job/glass-ios-app/

Then navigate to your release branch. A build with your changes should already
be running. If you need to do any custom builds you can use the UI buttons in
the top left to create the build you need.

A Release Candidate (RC) and Submission Candidate (SC) build should be created
and the result will be posted in `#builds-glass-ios`


### Submit for App Store Review
Create the new version in App Store Connect.
Once QA has validated the build submit it for App Store Review. Be sure to
select phased rollout.


### Release the app.
Once you have:
* A build created
* * Sign off From QA
* * App Store Review Complete
You'll need to work with the Program Team and QA to get a CRQ created for the
release. There may be requirements to let the VP know about the release.

During the CRQ window, release the app and monitor for crashes.
