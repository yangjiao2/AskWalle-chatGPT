---
title: Architecture Overview
navTitle: Overview
---

# Architecture Overview

## App Project Structure

Overview of the Glass app's project structure

- `Configuration` - Xcode build configuration files
- `Frameworks` - Core libraries such as WalmartPlatform
- `Modules` - App-specific shared libraries
- `Plugins` - Plugin modules
  - `AppRoot` - Configures root view controller
  - `PluginAPIs` - Collection of Plugin APIs
  
## Frameworks

### Plugin Modules

Each feature is integrated in the app as a framework called a plugin module. See the [Plugin System](the-plugin-system.md) for more information.

### App-specific Shared Libraries

- `WalmartPlatformExtensions` - Glass-specific extensions on WalmartPlatform types

### Core Libraries

- `GlassUI` - Living Design components and UI utilities
  - Source: https://gecgithub01.walmart.com/walmart-ios/glass-platform/tree/development/WalmartPlatform
  - Documentation: [https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/platform/ui/](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/ui/)
- `WalmartPlatform` - core capabilities like networking, analytics, logging, and deep linking
  - Source: https://gecgithub01.walmart.com/walmart-ios/glass-platform/tree/development/GlassUI
  - Documentation: [https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/)
- `TestUtilities` - A set of utilities to help with unit testing
  - Source: https://gecgithub01.walmart.com/walmart-ios/glass-platform/tree/development/TestUtilities
  - Documentation: TBD
