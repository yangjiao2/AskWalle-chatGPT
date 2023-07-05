---
title: Contribution Guidelines
navTitle: Contributing
---

# Contribution Guidelines

This document contains a list of contribution guidelines specific to the iOS app. Please be sure to also read the [Mobile Contribution Guidelines](https://engineering.walmart.com/docs/mobile/mobile/contributing).

## API Design

- APIs should strive to follow the [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)

## Platform APIs

- Contributors are expected to be familiar with [WalmartPlatform](https://gecgithub01.walmart.com/walmart-ios/glass-platform/)'s capabilities and must use the platform's APIs for all of the core capabilities that it offers (networking, analytics, deep linking, etc).

## Plugin Modules

- Every feature must be implemented within a [plugin module](../architecture/plugin-modules.md).
- Every plugin module in the app is required to have a code owner associated with it in the  `.github/CODEOWNERS.MD`.
- Plugin modules cannot import other plugin modules.
- Plugin modules must communicate with other plugin modules via [Plugin APIs](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/Plugins/PluginAPIs/PluginAPIs/APIs)
- Plugin modules must localize strings. We should not be using String literals. 

## UI

- All UI code is required to be constructed programmatically in code without using Interface Builder (Storyboards and Xibs). See the [FAQs doc](../faqs/index.md#why-cant-we-use-interface-builder) for more info.
- Features should use the Living Design UI components (GlassUI library) as much as possible. If the designs can't be implemented using GlassUI, point out the discrepancy to your designer and ask that the design either be made consistent with the sticker sheet or that the change be socialized with the UX platform team.

## Unit Testing

- Every module is expected to maintain at least 75% test coverage. 
- Module changes cannot be merged if the module does not meet the 75% test coverage requirement.
- Unit tests cannot make requests over the network
- [TestUtilities](https://gecgithub01.walmart.com/walmart-ios/glass-platform/tree/development/TestUtilities) should be used as much as possible to avoid repeating common unit test boilerplate
