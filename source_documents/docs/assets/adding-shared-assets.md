# Shared Asset Guidelines

This document outlines the guidelines that should be followed when adding new shared assets to the glass project.

## What is an "asset"?

An asset is a resource that appears in an [Asset Catalog](https://developer.apple.com/documentation/xcode/managing-assets-with-asset-catalogs).

## Adding a New Shared Asset

When adding a new image asset, you should first follow the guidelines [here](image-asset-guidelines.md) for ensuring the asset does not already exist in the project, is sized correctly, and is in the correct format.

## When should an asset be added to a shared module?

An asset should be added to a shared module if that asset needs to be accessed by 2 or more features. If you are looking for guidance related to adding an asset that will be used only in your feature, then [this document](image-asset-guidelines.md) may be more appropriate.

In Glass, there are 2 modules where shared assets can live:

1. [GlassUI](#glassui)
2. [LivingDesign](#livingdesign)

Guidance below will help you determine which module is most appropriate for your new asset.

## As a Feature, how should I share my assets with other features?

If you have one or more assets that need to be used with other features, then adding them to one of the modules mentioned above will make them accessible to any other features that may want to use them.

1. First, determine where the asset should live. In general, if the asset can be used generally in the app, and doesn't have any feature specific branding or business logic, then it would be a good candidate for [inclusion into GlassUI](#glassui) or [LivingDesign](#livingdesign).
2. Once you have chosen a module for your asset, follow the guidance below for specifics on adding your asset to the most appropriate module.

### GlassUI

GlassUI contains assets and UI elements that are common amongst Glass features, but are not officially part of the Living Design specification.

An asset should be added to GlassUI if it fits the following conditions:

1. The asset can be used generally, and doesn't contain any feature specific branding or styling (e.g. a generic icon or error image asset).
2. If the asset provides a functionality (like a button or some other UI element) then it should not contain any feature specific business logic. It should be self-contained with a clear and generic usage API.
3. (Optional) LivingDesign decides not to accept it to their library.

#### Adding to GlassUI

#### Add to Asset Catalog

Inside the GlassUI project you'll find the `GlassAssets` asset library. Add your asset to this asset library. If your asset is related to some UI for errors, then add it to the `Error` folder, otherwise add it to the `Glass` folder.

#### Update Asset Enum

The asset should be accessed through strong typing, such as an enum.

```swift
public enum GlassIcon CaseIterable {
    case account
    case appleCare
    case myNewAsset
}
```

#### Use the Asset

To use the asset from your plugin, import GlassUI and reference the newly added enum case:

```swift
import GlassUI

let image: UIImage = GlassIcon.myNewAsset.image
```

If your plugin has a sample app, make sure you configure your sample app target to include GlassUI:

1. Add GlassUI to the app target's `Target Dependencies` and `Link Binary With Libraries` builds phases
2. Add the GlassUIResources bundle to your app target's Copy Bundle Resources build phase

#### Getting sign-off for a new GlassUI asset

Getting sign-off for adding your asset to GlassUI is done by creating a PR with your new asset added to the module. A Platform team member will review the PR and if everything looks good, you'll get an approval.

GlassUI is maintained by the iOS Platform team. If you have a question or run into any issues when adding your asset, reach out to them using the @glass-platform-ios slack handle.

### LivingDesign

[Living Design](https://livingdesign.walmart.com) is maintained by the Living Design team, and contains a standardized set of fonts, icons, components and usage guidelines that collectively define Walmart's design language across web, mobile, and various design tools.

Living Design defines Walmart's officially approved design language across all of our platforms, and as such, there is an approval process that must be adhered to in order to get a new asset added to the LD library.

Note: If you're looking to add a new icon to the project, then the LD Community Icons library is likely what you're looking for. More details about [Community Icons can be found here](https://livingdesign.walmart.com/community-and-subsystems/community-icons/).

#### Getting sign-off for a new LD asset

Contributing to Living Design is a bit more involved than other modules, but can be worth it if you wish this asset to be maintained to a high standard going forward.
- If you are contributing a new icon, then follow the process defined here: [How to contribute a new icon](https://livingdesign.walmart.com/foundations/icons/contributing/#how-to-contribute-a-new-icon).
- If you are contributing a new component (like a button or other UI element) then look [here](https://livingdesign.walmart.com/resources/contributing/components/) for information on how to do that.

# Useful Links

1. [Living Design](https://livingdesign.walmart.com)
2. [Living Design Contributions](https://livingdesign.walmart.com/resources/contributing/)
3. [Adding or updating an LD icon](https://gecgithub01.walmart.com/LivingDesign/icons/blob/main/CONTRIBUTING.md#how-to-contribute)
