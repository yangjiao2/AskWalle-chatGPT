---
title: Glass iOS Strings - Localization Build Tooling Setup
navTitle: Glass iOS Strings - Localization Build Tooling Setup
---

# Glass iOS - Localization Build Tooling Setup

This tutorial outlines the steps for publishing content to the Translation Management System (TMS) 
and downloading localized content for your application. Please make sure you have completed the [Migration Tutorial](strings-migration-tutorial.md) before completing this one.

## Setting up your plugin to use the Localization build Tooling:

### Summary of steps
- Step 1. Add the localization capability to your plugin or app's glassconf file.
- Step 2. Add validate strings phase execution script.

### Tutorial assumptions

***The commands in this tutorial assume the following:***

- You are on the VPN.
- Your project already matches the [organizational structure](strings-project-structure.md) required
  by the tooling.  If this is not the case, please see the [migration tutorial](strings-migration-tutorial.md).
- Your plugin's strings have already been converted to ICU as needed. 
  If this is not the case, please see the [migration tutorial](strings-migration-tutorial.md)

### Step 1. Add the localization capability to your plugin or app's glassconf file.
- Run `glass configuration add-capability --capability localization --plugin {YourPlugin}`
- The previous command will have added the the following stub to your glassconf file:
```json
"capabilities" : [
    {
      "resourceDirectoryPath" : "",
      "supportedLanguages" : [ ],
      "type" : "localization"
    }
  ]
```
- Fill in both the resourceDirectoryPath and supportedLanguages fields
  - resourceDirectoryPath should be set to relative path to your plugins root resource directory.
  - supported languages should be set to the following (This is subject to change as new languages are added):
  ```json
  "supportedLanguages" : [
        "en",
        "en-US",
        "en-CA",
        "es-419",
        "es-MX",
        "fr-CA"
      ]
  ```

### Step 2. Run the localization steps.

Now that your plugin is setup to use the localization tooling, you can proceed to run the steps outlined in the [localizing new content tutorial](strings-localize-new-content.md), to get your plugin localized for the first time!

## Tutorial complete
###### [Return to index](index.md)
