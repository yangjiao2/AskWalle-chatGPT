---
title: How to CCM
navTitle: CCM
---

# How to CCM (Cloud Content Configuration)

This guide provides an overview of how to implement CCM in a plugin module.

- [Best Practices](#best-practices)
- [Adding a New Flag](#adding-a-new-flag)
- [Creating a CCM Model](#creating-a-ccm-model)
- [Subscribing to CCM Updates](#subscribing-to-ccm-updates)
- [Market Scopes](#market-scopes)
- [Glass iOS CCM Flags](#glass-ios-ccm-flags)
- [More Information](#more-information)

## Best Practices 

The following are some best practices when adding a new CCM flag.

### Sync with Android

Collaborate with your green bubble friend and use the same CCM flag name; this simplifies production changes, product communication, and keeps the platforms in alignment.

### Incremental development

The `glass-app` repository is constantly changing from a myriad of contributors committing code daily. Therefore, long-lived feature branches are not only discouraged, but disallowed except in extenuating circumstances. If you're working on a new feature, it is recommended to develop your feature incrementally through a series of small to medium sized PRs into the main `development` branch. To ensure your feature is not exposed to users until it is completed, add a CCM flag with a default value (e.g. `false`) that hides the new feature. When adding the CCM flag, PR the addition separately from other code changes. There is no need to deploy the CCM flag to production until the feature is ready for release since the default value hides the experience. When the feature is ready, simply update the flag in production with an appropriate `minVersion`. 

## Deprecations 

- `CCModel`
- `CCMModelDefaultsProvider`

## Adding a New Flag

First, add a new flag in the CCM Portal. See the [Mobile CCM Documentation](https://gecgithub01.walmart.com/glass/docs/blob/master/docs/mobile/ccm/index.md) for information on adding a new flag in the CCM portal.

### CCMDefaultableModel

The `CCMDefaultableModel` protocol provides a `defaultCCM` value during the decoding process. This obviates the need to create multiple initializers. The `defaultCCM` value is injected when requesting a publisher for the model.

- Create a struct with `public` properties that represent all of the CCM flags in your Plugin, e.g. `MyPluginCCM`. Use nested structs, e.g. `MyPluginCCM.SubFeatureOne`, to namespace related CCM flags.
- Provide default values inline to each of the properties on the CCM object and provide documentation that mentions the default value. 
- Conform top-level and nested strucs to `CCMDefaultableModel` in an extension and implement the `init(decoder: Decoder, defaultCCM: MyPluginCCM)` similar to the example below. This should be the **only** initializer you should _need_ to create. 
- Add a `static` `defaults` property that is initialized with the default values using the synthesized initializer. An outside consumer can obtain a mutable copy of `defaults` and modify the it to fit their needs.

#### Example:

```swift
public struct MyPluginCCM {

    /// Whether `MyPlugin` is enabled.
    public var isEnabled: Bool = true

    public var subFeatureOne = SubFeatureOne()
    public var subFeatureTwo = SubFeatureTwo()

    public struct SubFeatureOne {

        /// Whether SubFeatureOne is enabled.
        public var enabled: Bool = false

        /// Determines the number of times a message is displayed to the user.
        public var messageCount: Int = 2
    }

    public struct SubFeatureTwo {

        /// Whether SubFeatureTwo is enabled.
        public var enabled: Bool = true
    }
}

// MARK: - MyPluginCCM

extension MyPluginCCM: CCMDefaultableModel {

    public static let defaults = Self()

    private enum CodingKeys: String, CodingKey {
        case isEnabled = "feature.enabled"
    }

    public init(decoder: Decoder, defaultCCM: MyPluginCCM) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        self.init(
            isEnabled: container.decodeVersioned(.isEnabled) ?? defaultCCM.isEnabled,
            subFeatureOne: try SubFeatureOne(decoder: decoder, defaultCCM: defaultCCM.subFeatureOne),
            subFeatureTwo: try SubFeatureTwo(decoder: decoder, defaultCCM: defaultCCM.subFeatureTwo)
        )
    }
}

// MARK: - MyPluginCCM.SubFeatureOne

extension MyPluginCCM.SubFeatureOne: CCMDefaultableModel {

    public static let defaults = Self()

    private enum CodingKeys: String, CodingKey {
        case enabled = "feature.subFeatureOne.enabled"
        case messageCount = "feature.subFeatureOne.messageCount"
    }

    public init(decoder: Decoder, defaultCCM: MyPluginCCM.SubFeatureOne) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        self.init(
            enabled: container.decodeVersioned(.enabled) ?? defaultCCM.enabled,
            messageCount: container.decodeVersioned(.messageCount) ?? defaultCCM.messageCount
        )
    }
}

// MARK: - MyPluginCCM.SubFeatureTwo

extension MyPluginCCM.SubFeatureTwo: CCMDefaultableModel {

    public static let defaults = Self()

    private enum CodingKeys: String, CodingKey {
        case enabled = "feature.subFeatureTwo.enabled"
    }

    public init(decoder: Decoder, defaultCCM: MyPluginCCM.SubFeatureTwo) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        self.init(
            enabled: container.decodeVersioned(.enabled) ?? defaultCCM.enabled
        )
    }
}
```

## Subscribing to CCM Updates

Finally, use [`CCMSource`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Protocols/CCMSource.html) to create a [publisher](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Protocols/CCMSource.html#/s:15WalmartPlatform9CCMSourceP9publisher3for7Combine12AnyPublisherVyqd__s5Error_pGqd__m_tAA8CCMModelRd__lF) the decodes the model upon a successful CCM update. The initial value is based on the last fetched CCM data.

**Important:** Previous CCM responses are cached to disk and loaded on app launch.

**Important:** If you subscribe to the CCM publisher for updates, the updates are published on an **arbitrary** queue.

### CCMDefaultableModel

It is **highly** recommended to obtain a _single ccm publisher_ and inject it throughout your code. You can store it a child container or save a reference to it on your PluginAPI. The example below saves a reference to the publisher on `MyPluginAPI`. The default CCM configuration should come from your Plugin's configuration object. 

```swift
public final class MyPluginAPI {

    public struct Configuration {
        public var ccm: MyPluginCCM

        public init(ccm: MyPluginCCM = .defaults) {
            self.ccm = ccm
        }
    }
    
    let ccm: AnyValuePublisher<MyPluginCCM, Never>

    public init(container: Container, configuration: Configuration = Configuration()) {
        self.container = container
        // The publisher maintains the most-recently fetched CCM value.
        // Accessing `ccm.value` is thread-safe.
        self.ccm = container.get(CCMSource.self).publisher(for: MyPluginCCM.self, defaults: configuration.ccm)
    }
```

## Unit Testing

Unit testing CCM is straightforward and requires only **two tests**:

1. Verify the JSON value takes precedence over the default value during the decoding process. You should also verify any data transformations are performed correctly (e.g. dates, comma delimited string to array, etc).
2. Verify the default CCM values are used when the JSON values are not present. 

### Example

```swift
final class InAppMessagingCCMTests: XCTestCase {

    func test_decoding_whenCCMValuesArePresentInJSON_JSONValuesAreUsedDuringDecoding() throws {
        // Given
        let jsonData = """
        {
            "platform.iam.callout.enabled": { "value": true },
            "platform.iam.callout.maxDisplay": { "value": 1337 },
            "platform.iam.callout.maxTotal": { "value": 1993 },
            "platform.iam.callout.windowHours": { "value": 4 },
            "platform.iam.diagnostics.failedRules.enabled": { "value": false }
        }
        """.data(using: .utf8)!

        // When
        let ccm = try JSONDecoder().decode(InAppMessagingCCM.self, from: jsonData)

        // Then
        XCTAssertTrue(ccm.callout.enabled)
        XCTAssertEqual(ccm.callout.maxDisplay, 1337)
        XCTAssertEqual(ccm.callout.maxTotal, 1993)
        XCTAssertEqual(ccm.callout.windowInterval, 14400)
        XCTAssertFalse(ccm.diagnostics.failingRulesEnabled)
    }

    func test_decoding_CCMValuesAreNotPresent_defaultsAreUsedDuringDecoding() throws {
        // Given
        let emptyJSON = "{}".data(using: .utf8)!
        let defaults = InAppMessagingCCM(
            callout: InAppMessagingCCM.Callout(
                enabled: false,
                maxDisplay: 7113,
                maxTotal: 2022,
                windowInterval: 100
            ),
            diagnostics: InAppMessagingCCM.Diagnostics(failingRulesEnabled: true)
        )

        // When
        let ccm: InAppMessagingCCM = try JSONDecoder.decodeCCM(data: emptyJSON, defaults: defaults)

        // Then
        XCTAssertFalse(ccm.callout.enabled)
        XCTAssertEqual(ccm.callout.maxDisplay, 7113)
        XCTAssertEqual(ccm.callout.maxTotal, 2022)
        XCTAssertEqual(ccm.callout.windowInterval, 100)
        XCTAssertTrue(ccm.diagnostics.failingRulesEnabled)
    }
}
```

#### Understanding the Client CCM Caching Logic

CCM flags are fetched via a ce-ccm-bootstrap call, and are then cached and accessed by the app and stored on the device.

There are 4 triggers that might cause a CCM refresh (that same bootstrap call), otherwise the cached data is used.

### Market Scopes
#### Overview
With the new bootstrap service launching in the 22.38 release, multiple markets are now fully supported for app versions 22.38 and higher.
It is critical to understand how your CCM updates in both prod and staging will affect all markets. If you update a value directly in the root scope, understand that it will be inherited by all child scopes.

The US app now reads the `WALMART-US-B2C` scope in versions 22.38 and higher, the Mexico app reads the `WALMART-MX-B2C` scope (MX and CA have always used this bootstrap service so it's always worked this way), and so on. These market "child" scopes all inherit from the "root" scope, which can be viewed by clicking the config in walmart-mobile-ios.

> Pro-tip: Use the "View inherited" check box to see what your child scope is inheriting from the root scope.

##### PROD ENVIRONMENT SCOPES

root: US app production config for app versions under 22.38. All child scopes inherit from this for all versions
prod: Can be ignored completely

`WALMART-US-B2C`: US app production config for app versions 22.38 and higher, inherits from root

`WALMART-CA-B2C`: Canada app production config, inherits from root

`WALMART-MX-B2C`: Mexico app production config, inherits from root

##### STG ENVIRONMENT SCOPES

When working with staging CCM, please use https://admin.tunr.walmart.com/services/walmart-mobile-ios/service-configs/NON-PROD-1.0/configs

Docs on updating CCM2: https://confluence.walmart.com/pages/viewpage.action?pageId=509928400#CCM2UIUserGuide-ViewandUpdateServiceConfigurations

Start by adding the ccm key and value to the `Config Definition` tab properties if it does not already exist. 
To add the key to the correct scope go to `Config Overrides` tab, select the correct scope. Select properties and toggle the edit in the header for the scope. Finally select the import key for new keys and update the value. Make sure to save your changes when finished. CRQ is not required for staging changes.

`ADMIN_UI-WALMART-US-B2C-*-*`: US app staging config for app versions 22.38 and higher, inherits from root.

`ADMIN_UI-WALMART-US-B2B-*-*`: Upcoming b2b app staging config. Contact the B2B team for the correct scope.

`ADMIN_UI-WALMART-CA-B2C-*-*`: Canada app staging config, inherits from root.

`ADMIN_UI-WALMART-MX-B2C-*-*`: Mexico app staging config, inherits from root.

#### Process: Updating a CCM flag that already exists in the root scope

##### Preferred method [US app version 22.38 and higher only]:
1. Rather than updating the root scope value, override it directly in the child scope you want to alter.
2. Post a message in the #glass-mobile-ccm-updates channel using the CCM Changed Workflow. The Workflow will be accessible from #glass-mobile-ccm-updates

> NOTE: Values updated in WALMART-US-B2C will only be seen by app versions 22.38 and above.

##### If you need to update a flag to be read on earlier versions of the US app, you'll need to follow a different process:
1. Update all child scopes except for WALMART-US-B2C to the same value as the root scope
2. Update the WALMART-US-B2C scope with the change you need reflected in the US app.
3. Update the root scope with the change you need reflected in the US app
4. Post a message in the #glass-mobile-ccm-updates channel using the CCM Changed Workflow. The Workflow will be accessible from #glass-mobile-ccm-updates

#### Process: Adding a brand new CCM flag

##### Preferred method [US app version 22.38 and higher only]:
1. Generally, set your default value in the app to be disabled, or something that you know will work for every market.
2. Don't touch the root scope config, and instead set value overrides directly in the child scope.
3. Post a message in the #glass-mobile-ccm-updates channel using the CCM Changed Workflow. The Workflow will be accessible from #glass-mobile-ccm-updates

> NOTE: New flags added to WALMART-US-B2C will only work on app versions 22.38 and above.

##### If you need to add a flag to be read on earlier versions of the US app, you'll need to follow a different process:
1. Set the default in the app to be disabled or something that will work for every market.
2. Update all child scopes except for WALMART-US-B2C to the same value as the app default
3. Update the WALMART-US-B2C scope with the change you need reflected in the US app.
4. Update the root scope with the change you need reflected in the US app
5. Post a message in the #glass-mobile-ccm-updates channel using the CCM Changed Workflow. The Workflow will be accessible from #glass-mobile-ccm-updates

#### Process: Updating a CCM flag that only exists in one or more child scopes and NOT root [US app version 22.38 and higher only]:
1. Update in the child scopes as needed

#### Process: CRQs
For CCM CRQ's:
Ensure you specify in the CRQ description the changes that will need to be made for each market according to the process above.

#### Get help
If you need a CCM consultation, please don't hesitate to reach out to the platform team.

## Glass iOS CCM Flags

CCM flags used in Glass are listed [here](ccm-flags.md)

## More Information

- [Mobile CCM documentation](https://gecgithub01.walmart.com/glass/docs/blob/master/docs/mobile/ccm/index.md)
- [WalmartPlatform Quimby API](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/#quimby)
