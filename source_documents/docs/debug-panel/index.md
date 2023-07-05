---
title: The Debug Panel
navTitle: Debug Panel
---

# The Debug Panel

- [What?](#what-is-the-debug-panel)
- [How?](#how-to-open-the-debug-panel)
- [Adding items](#adding-entries-to-the-debug-panel)
- [Deeplink support](#how-to-deeplink-to-debug-panel)

## What is the Debug Panel?

The debug panel is an overlay window that includes variety of useful functions. It provides capabilities to add Glass-Team-Specific entries, from app / plugin configuration, to view controller presenting, to service mocking. The entries are alphabetically sorted.

<details>
<summary>Analytics Super Attributes</summary>

This screen displays various analytics stats relating to the device, its environment, and the app's current state.

<img src="images/features/analytics_super_attributes.png?raw=true" alt="Analytics Super Attributes" width="320"/>

</details>

<details>
<summary>Select Environment</summary>


This screen allows you to switch between backend environments.  Note that the app will (and must) terminate in order for the changes to apply.

<img src="images/features/environment_select.png?raw=true" alt="Environment Selection" width="320"/>

<details>
<summary>Others</summary>


Other entries in this section may be specific to certain teams, or used to aid in certain workflows.  They are expected to change and grow as needed.

</details>
</details>

## How to open the Debug panel

It can be invoked in one of two ways:

1. By swiping right-to-left from the center right edge of the screen (useful on device)
2. By pressing "Control + D" on a hardware keyboard (useful in the Simulator)

Note: When swiping right-to-left, you need to be sure there isn't a horizontally-scrolling collection view, or some other form of scroll view, in that same position, or it will have priority on user interaction.

## Adding entries to the Debug Panel

Adding an entry to the Debug Panel is as simple as adding a new Module object to the `registerModules(container: Container)` function in [DebugPanelContainerViewController+Features.swift](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/DebugPanel/DebugPanel/Sources/ContainerViewController/DebugPanelContainerViewController+Features.swift).  You do this by creating a new module with DebugPanelModel(which includes an identifier, name, an optional description) and view controller provider, and appending it to the modules array.<br /><br />

<img src="images/new_entry.jpg?raw=true" alt="New Entry" width="320"/>

There are no restrictions on the appearance or behaviors of any view controller associated with a module, but we should make an effort to adhere to expected layouts and UIs, depending on the purpose.  For example, if you are adding additional settings, try to use the Settings app as inspiration, providing sectioned table cells with switches, checkmarks, and the like.  If the screen performs some behavior on load, or in response to some action, try to inform the user of exactly what they might expect, rather than hoping they can infer it from name alone.  If the screen provided is used more for convenience, demonstration, or some other purpose, then presenting it may be well enough, although adding an interstitial screen that explains its purpose may be desirable in some cases.

## How to deeplink to Debug Panel

Deeplink support to debug panel comes handy when dealing with automation. Debug panel can be opened by using a deeplink URL, **walmart://debugPanel**,  in Safari app on your simulator or through terminal using this command

```bash
xcrun simctl openurl booted "walmart://debugPanel"
```

Deeplink to a particular [identifier](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/PluginAPIs/PluginAPIs/APIs/DebugPanelAPI.swift#L31) under debug panel is also supported. Inorder to triger deep link to a particular [identifier](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/PluginAPIs/PluginAPIs/APIs/DebugPanelAPI.swift#L31), the path needs to be provided, ie, walmart://debugPanel/**debug-panel-identifier**

Ex: To open **Network Log** in Debug Panel, use the following deep link path

```bash
xcrun simctl openurl booted "walmart://debugPanel/networkLog"
```

Note that an incorrect identifier will just open the Debug Panel
