---
title: Glass iOS Strings - Localizing Content Guide
navTitle: Glass iOS Strings - Localizing Content Guide
---

# Glass iOS - Localizing Content Guide

This tutorial outlines the steps for localizing content in your plugin. Please make sure you have performed all the steps in the [build tooling tutorial](strings-localization-tutorial.md) before proceeding with this one.

## Localizing new content going forward:

### Summary of steps
- Step 1: Convert/Add new string.
- Step 2: Create and merge your PR.

### Tutorial assumptions

***The steps in this tutorial assume the following:***

- You are on the VPN.
- Your plugin's strings have already been converted to ICU as needed. 
  If this is not the case, please see the [migration tutorial](strings-migration-tutorial.md).
- Your plugin has already been setup to use the Localization build tooling. 
  If this is not the case, please see the [localization tutorial](strings-localization-tutorial.md).
  
### Step 1: Convert/Add new string
Convert the new/modified string to ICU format if needed, and add them to the english(default) Localizable.strings file.

### Step 2: Create and merge your PR.

Developers are no longer required to publish and download strings to and from the Crowdin TMS. So you can just add your english copy and merge your PR, CI tooling will take care of the rest.

***NOTE: There is a 48 hours SLA for new translations once they hit the TMS. Can take ~72 hours though.***

### How does automated strings work? 

The Glass platform now handles all publishing and downloading of localized strings automatically via CI, so our developers no longer need to worry about it. This works via the following processes:

**Development Branch**
1. The engineer merges their new string(s) into development after adding them to the `en.lproj/Localizable.strings` of their plugin.
2. Their job is done.
3. Two cron jobs are kicked off on the `development` branch every 6 hours:
   1. Publish Strings:
      1. `glass strings publish -e development` is run
      2. GlassCLI fetches all GlassConf files with the localization capability
      3. GlassCLI kicks off publishing in parallel for all configurations:
         1. The Crowdin version of the localization file is downloaded and merged with developments local file to create a master copy. This is done to prevent the accidental erasure of strings.
         2. The master copy of the strings file is published to crowdin to add the new strings.
         3. TMS translators translate the new content in all languages (This is done asynchronously and will have the aforementioned 24hr SLA)
         4. GlassCLI resets the local copy of the file as to not add an unnecessary strings. 
         5. Publishing is complete for this plugin.
      
   2. Download Strings:
      1. `glass strings download -e development` is run
      2. GlassCLI downloads all strings files for iOS from Crowdin
      3. GlassCLI copies all downloaded strings files over to their respective plugin's resource directories. This is based on their GlassConf and will only happen for the those with the localization capability.
      4. GlassCLI creates a new branch `automatic/strings/development`
      5. GlassCLI commits all new strings file changes, restoring any changes that are not Localizable.strings files.
      6. GlassCLI opens a PR and merges it into the `development` branch using two service accounts, one to create the PR and another to approve it.
      7. If the PR fails to merge, the Platform team is notified.
      8. Upon rebasing with development, the engineer will see their newly translated strings.

**Release Branches**
1. There are two ways for a string to need translation in a release branch:
   1. The engineer merges their new string(s) into a `release` branch after adding them to the `en.lproj/Localizable.strings` of their plugin. This will only happen in the case of a PCF.
   2. A `release` branch is cut before all content in development has been translated.
3. Two cron jobs are kicked off on every `release` branch every 6 hours:
   1. Publish Strings:
      1. `glass strings publish -e release` is run
      2. GlassCLI fetches all GlassConf files with the localization capability
      3. GlassCLI kicks off publishing in parallel for all configurations:
         1. The Crowdin version of the localization file is downloaded and merged with developments local file to create a master copy. This is done to prevent the accidental erasure of strings.
         2. The master copy of the strings file is published to crowdin to add the new strings.
         3. TMS translators translate the new content in all languages (This is done asynchronously and will have the aforementioned 24hr SLA)
         4. GlassCLI resets the local copy of the file as to not add an unnecessary strings. 
         5. Publishing is complete for this plugin.
      
   2. Download Strings:
      1. `glass strings download -e release` is run
      2. GlassCLI checks if there are missing translations for any plugins. If not, it exits early as there is nothing today and this release branch is ready to ship.
      3. GlassCLI downloads all strings files for iOS from Crowdin
      4. GlassCLI copies all downloaded strings files over to their respective plugin's resource directories. This is based on their GlassConf and will only happen for the those with the localization capability.
      5. GlassCLI creates a new branch `automatic/strings/{insert-release-branch-here}`
      6. GlassCLI commits all new strings file changes, restoring any changes that are not Localizable.strings files.
      7. GlassCLI opens a PR and merges it into the `release` branch using two service accounts, one to create the PR and another to approve it.
      8. If the PR fails to merge, the Platform team is notified.
      9. GlassCLI checks if there are missing translations for any plugins. If not, it sends a notification to  #i18n-glass-mobile-auto-alerts that the release branch successfully downloaded all strings.
      10. If missing strings are detected, GlassCLI sends an alert to #i18n-glass-mobile-auto-alerts notifying @glass-localization-support that there are missing strings in a release branch that require escalated translation. 
      11. The @glass-localization-support group translates the missing strings and either restarts the looper job or waits for the next one to run. 
      12. The process repeats until all strings are downloaded and their are no missing translations. 

**RC and SC builds**
RC builds will not be blocked by missing translations but SC builds will. This will make it impossible to ship a build to the app store that is missing translations. 

## Tutorial complete
###### [Return to index](index.md)
