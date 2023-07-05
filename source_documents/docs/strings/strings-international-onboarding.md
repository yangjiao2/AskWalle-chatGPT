---
title: Glass iOS Strings - Migrating your plugin to ICU
navTitle: Glass iOS Strings - Migrating your plugin to ICU
---

# International Markets: Customizing your allowed Locales

This document contains the steps needed to setup your international market's Glass App with the locales it supports.

## Configuring your supported locales:

### Summary of steps
- Step 1: Add the special private interface import to access privileged WalmartPlatform APIs.
- Step 2: Call the Strings setup method to configure the allowed locales.

### Step 1: Import

In order to access the StringsAPI's configuration methods, you must import WalmartPlatform using @\_spi(PrivilegedPlatformAPI). @\_spi is special annotation that allows modules to restrict access to specific apis using a keyword. While it is an experimental swift offering it is unlikely to change, that said it is not recommended to use outside of Platform offerings. 

```swift
@_spi(PrivilegedPlatformAPI) import WalmartPlatform
```

### Step 2: Call the setup method

To configure your allowed locals call the static `String.setupStringsAPI` method. This method will set your allowed locales for release versions of your app. However, to support development and testing, debug versions will continue to attempt resolution of all locale types regardless of their inclusion in the allow list.

It is recommended to call this method as early as possible in your app's startup, your app delegate is a great place to do so.

```swift
String.setupStringsAPI(
    with: .init(
        allowedLocales: [
            .init(identifier: "en"),
            .init(identifier: "en-US")
        ],
        fallbackLocale: .init(identifier: "en-US")
    )
)
```

###### [Return to index](index.md)
