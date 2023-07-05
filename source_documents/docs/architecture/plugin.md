---
title: Plugin
navTitle: Plugin
---

[« Architecture Documentation - Table of Contents](index.md)

# Plugin

Note: This document details a component of [The Plugin System](the-plugin-system.md).

- [What Is A "Plugin"?](#what-is-a-plugin)
- [Creating A Plugin](#creating-a-plugin)

## What Is A "Plugin"?

A Plugin is an Xcode Project which contains the following components:

- [Plugin Module](plugin-modules.md) (required)
- [Plugin API(s)](plugin-api.md)
- Unit Testing target(s)
- UI Testing target(s)

### Plugin Examples

Looking through the Walmart App, here are a few examples of Plugins which are currently in use, each residing within the Plugins directory in the root directory of the project:

- [`Authentication`](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/Authentication/)
- [`CXO`](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/CXO/)
- [`Home`](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/Home/)
- [`PurchaseHistory`](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/PurchaseHistory/)
- [`Search`](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/Search/)

See [the Plugins directory](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/) for more.

## Creating A Plugin

Plugins are generated automatically while following the process for creating a [Plugin Module](plugin-modules.md).  See: [Creating A Plugin Module](plugin-modules.md#creating-a-plugin-module) for more details.

