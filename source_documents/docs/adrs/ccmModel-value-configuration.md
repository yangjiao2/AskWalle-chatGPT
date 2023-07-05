# CCM Model Value Configuration

- Authors: [Hemanth Kumar](https://gecgithub01.walmart.com/h0k01sl), [John Liedtke](https://gecgithub01.walmart.com/j0l04ck)
- Initial Decision: Option 2 with Option 1
- Revised Decision: Option 3

## Introduction

With multiple markets entering the glass ecosystem, we need to determine the best possible way to allow for CCM Model Value configuration within our many plugins.

## Motivation

Currently, they are provided as hard-coded values stored within internal static structs (enums, actually), but we need to be able to inject them through some sort of configuration.

The complication arises from our `quimby.publisher(...)` method not really providing for the sort of configurations that we would like. 

The goal of this ADR is to determine the best potential ways for achieving this level of configuration all throughout the Glass App.

## Considered Options

### Option 1: Constants injection to a CCMModels when initializing the plugins 

- `CCMModel` will have a static variable of type `associatedtype` for accepting the defaults.

#### Example:

```swift
public protocol CCMModel: Decodable, Equatable {

    /// Defaults will specify the struct associated with CCMModel, for providing all default values
    associatedtype Defaults
    static var defaults: Defaults { get }

    /// The `CCMModel` initialized with appropriate default values.
    init()
}
```

- Each the custom CCMModel will have its own implementation for defaults and make CCMModel's to be public so that each markets can override the required defaults.
#### Example:

```swift
public struct AccountCCMModel: CCMModel, Decodable {

    /// Determines whether account deletion is enabled in Account
    let isAccountDeletionEnabled: Bool

    public struct Defaults {
        let isAccountDeletionEnabled: Bool

        public init(isAccountDeletionEnabled: Bool = false) {
            self.isAccountDeletionEnabled = isAccountDeletionEnabled
        }
    }

    public internal(set) static var defaults: Defaults = .init()

    enum CodingKeys: String, CodingKey {
        case isAccountDeletionEnabled = "account.EAE.accountDeletion.isEnabled"
    }

    public init() {
        self.init(
            isAccountDeletionEnabled: Self.defaults.isAccountDeletionEnabled
        )
    }

    init(isAccountDeletionEnabled: Bool?) {

        self.isAccountDeletionEnabled = isAccountDeletionEnabled ?? Self.defaults.isAccountDeletionEnabled
    }

    /// Attempt to decode from JSON or fallback to default value
    public init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        let isAccountDeletionEnabled: Bool? = container.decodeVersioned(.isAccountDeletionEnabled)

        self.init(
            isAccountDeletionEnabled: isAccountDeletionEnabled
        )
    }
}
```

- The plugin initializer will accepts `Defaults` of respective ccm model as a parameter to override the defaults.

#### Example:

```swift

public final class AccountPlugin {

    ////Class properties

    public init(container: Container,
                configuration: AccountPluginConfiguration,
                accountCCMModelDefaults: AccountCCMModel.Defaults = .init()) {
        AccountCCMModel.defaults = accountCCMModelDefaultsProvider
        self.container = container
        self.configuration = configuration
    }
}

```

- Each market will be initializing the plugins with respective `Defaults` for overriding the CCMModel defaults
#### Example:

```swift

extension BootstrapCoordinator {
       private func registerAccountAPIs() {
        container.registerProvidingChild(AccountPlugin.self) {
            return AccountPlugin(
                container: $0,
                configuration: .standard(environment: $0.legacyEnvironment),
                accountCCMModelDefaults: .init(isAccountDeletionEnabled: true)
            )
        }
    }
}
```

#### Pros:
- Easy customization for the constants for all CCM Models in each market-specific

#### Cons:
- If we have too many ccmModels associated with plugin, managing them would be more difficult
- Rewrite plugins to accept multiple ccm defaults

### Option 2: To solve the issue of managing multiple ccm models associated in Option 1,  here we are wrapping up all the ccm model defaults into a single struct from option 1, and can initialise by each market to override defaults for whichever the required values.

- `CCMModel` will have a static variable of type `associatedtype` for accepting the defaults.

#### Example:
```swift
public protocol CCMModel: Decodable, Equatable {

    /// Defaults will specify the struct associated with CCMModel, for providing all default values
    associatedtype Defaults
    static var defaults: Defaults { get }

    /// The `CCMModel` initialized with appropriate default values.
    init()
}
```

- Similar to Option 1, Each the CCMModel will have its own `Defaults` and corresponding shared instance, and make CCMModel's to be public so that each markets can override the requred defaults.

#### Example:
Model 1
```swift

public struct AdsCCMModel: Codable, CCMModel {
    let isMoatAnalyticsEnabled: Bool

    public struct Defaults {
        let isMoatAnalyticsEnabled: Bool

        public init(isMoatAnalyticsEnabled: Bool = false) {
            self.isMoatAnalyticsEnabled = isMoatAnalyticsEnabled
        }
    }

    public internal(set) static var defaults: Defaults = .init()

    public init() {
        self.init(enabled: Self.defaults.isMoatAnalyticsEnabled)
    }

    init(enabled: Bool?) {
        self.isMoatAnalyticsEnabled = enabled ?? Self.defaults.isMoatAnalyticsEnabled
    }

    public init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.init(enabled: container.decodeVersioned(.isMoatAnalyticsEnabled))
    }

    private enum CodingKeys: String, CodingKey {
        case isMoatAnalyticsEnabled = "ads.moatTracking.enabled"
    }
}
```

Model 2
```swift
public struct MarqueeDeDupeCCMModel: CCMModel, Equatable {

    let isMarqueeDeDupeEnabled: Bool

    public struct Defaults {
        let isMarqueeDeDupeEnabled: Bool

        public init(isMarqueeDeDupeEnabled: Bool = true) {
            self.isMarqueeDeDupeEnabled = isMarqueeDeDupeEnabled
        }
    }

    private enum CodingKeys: String, CodingKey {
        case isMarqueeDeDupeEnabled = "ads.marqueeDeDupe.enabled"
    }

    public internal(set) static var defaults: Defaults = .init()

    public init() {
        self.init(isMarqueeDeDupeEnabled: nil)
    }

    init(isMarqueeDeDupeEnabled: Bool?) {
        self.isMarqueeDeDupeEnabled = isMarqueeDeDupeEnabled ?? Self.defaults.isMarqueeDeDupeEnabled
    }

    public init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)

        self.init(isMarqueeDeDupeEnabled: container.decodeVersioned(.isMarqueeDeDupeEnabled))
    }
}
```

- A new protocol for each plugin for accociated all ccmModel's defaults

#### Example:
```swift
public protocol PluginAPI {

    /// CCMModelDefaults will specify the struct associated with CCMModel, for providing all default values
    associatedtype CCMModelDefaults
    var defaults: CCMModelDefaults { get }
}
```

- Each Plugin will have its own `PluginDefaults`, having all the defaults associated with it CCMModels, and a variable to update the defaults of CCMModels defauls when its updated from initialiser.

#### Example:
```swift
public class AdsPluginAPI: PluginAPI {

    public struct CCMModelDefaults {

        public init(adsCCMModelDefaults: AdsCCMModel.Defaults = .init(),
                    marqueeDeDupModelDefaults: MarqueeDeDupeCCMModel.Defaults = .init()) {
            AdsCCMModel.defaults = adsCCMModelDefaults
            MarqueeDeDupeCCMModel.defaults = marqueeDeDupModelDefaults
        }
    }

    private internal(set) var defaults: CCMModelDefaults {
        didSet {
            AdsCCMModel.defaults = defaults.adsCCMModelDefaults
            MarqueeDeDupeCCMModel.defaults = defaults.marqueeDeDupModelDefaults
        }
    }

    public init(configuration: AdsPluginConfiguration,
                    defaults: CCMModelDefaults = .init()) {
        self.adsPluginAPICCMModelDefaults = adsPluginAPICCMModelDefaults
        self.container = configuration.container
        self.networkService = adsNetworkService
        self.defaults = defaults
    }
}
```

- Each market will be initializing the plugins with respective `PluginCCMModelDefaults` for overriding the required CCMModel defaults
#### Example:
```swift

extension BootstrapCoordinator {

       private func registerAdsPluginAPIs() {

        let defaults: AdsPlugin.CCMModelsDefaults = .init(
            adsCCMModelDefaults: .init(isMoatAnalyticsEnabled: true),
            marqueeDeDupeDefaults: .init(isMarqueeDeDupeEnabled: false))
        container.registerProvidingChild(AdsAPI.self) {
            AdsPluginAPI(configuration: AdsConfiguration(container: $0),
                        adsPluginAPICCMModelDefaults:defaults)
        }
    }
}

```
#### Pros:
- Each market will be independent on each ccm model defaults and will have better control

#### Cons:
- Rewrite all CCMModels and plugins to support new default providers 

### Option 3: Override defaults to Quimby by passing the Json String on its initializer
- Update the Quibymanager initializer with QuimbyDefaultsProvider protocol, which provides the defaultJsonString(initial cache, can be injected) of the Quimby, which will initialize with default response data

#### Example:
```swift
public protocol QuimbyDefaultsProvider {
    static var defaultJSONString: String { get }
}

public final class QuimbyManager {
    let initialDefaultsProvider: QuimbyDefaultsProvider?
 
    public init(
            configuration: Configuration,
            networking: Networking,
            analyticsProvider: @escaping () -> AnalyticsTracking,
            initialDefaultsProvider: QuimbyDefaultsProvider? = nil) {
         self.initialDefaultsProvider = initialDefaultsProvider
         //Other properties
      }
    }
```

- A new method which will merge the defaults injected via `initialDefaultsProvider` with the latest response received from service, use network response as a higher priority with keys are in defaults if any keys are missing in network response then use the data for that key in `initialDefaultsProvider`.

#### Example:
```swift
extension QuimbyManager {
    func mergeResponse(responseData: Data, withDefaults defaultsJSONString: String) { 
           // merging logic    
    }
}
```

- Implement `QuimbyDefaultsProvider` in each market, with defaults, and pass it when initializing the Quimby

#### Example:
```swift
public class QuimbyDefaultsProviding : QuimbyDefaultsProvider {
    //All the ccm that needs to override 
    static var defaultJSONString: String {
            ""{\"ccm\": {\"ccmKey1\": {\"value\": true},\"ccmKey2\": {\"value\": \"Test String\"},\"ccmKey3\": {\"value\": 100}}}""    
    }
}

extension Container {
    func setupQumby() {
            registerProvidingChild(QuimbyProvider.self, { QuimbyProvider(container: $0,                                                                    initialDefaultsProvider: QuimbyDefaultsProviding()) })

    }
}
```

#### Pros:
- Easy to override all ccm models with a single step

#### Cons:
- Having default ccm's to be invoked in a single place would violate the single responsibility principle
- Also managing them at feature/plugin would be more difficult

### Option 4: On option 1, Platform provide a mechanism/API handle the defauts
- A new protocol on `CCMModel` will encorage each CCMModel's to conform to `CCMModelWithDefault` so that it can be exposed for accepting defaults

#### Example:
```swift
public protocol CCMModelWithDefault: CCMModel {
     static var defaultModel: Self { get set }
}
```

- `DefaultCCMProvider` will provide a machanism in register a ccmModel defaults

#### Example:
```
public protocol DefaultCCMProvider {
     func register<Model>(model: Model, for type: Model.Type) throws where Model : CCMModelWithDefault
}
```

- An implementation of `DefaultCCMProvider` is`QuimbyDefaultCCMProvider` helps in managing the defaults

#### Example:
```swift
public class QuimbyDefaultCCMProvider: DefaultCCMProvider {
    var registry: [String: Any] = [:]

     /// one time setting of static var
    public func register<Model>(model: Model, for type: Model.Type) throws where Model : CCMModelWithDefault {
         guard registry[String(describing: type)] == nil else { return /* make this throw */ }
         registry[String(describing: type)] = model
         Model.defaultModel = model
     }

    public init() { }
 }
```

- Register `QuimbyDefaultCCMProvider` as part of container
#### Example:
```swift
    container.registerProvidingChild(DefaultCCMProvider.self) { _ in QuimbyDefaultCCMProvider.init() }
```

- During bootstrapping set any default models as desired
#### Example:
```swift
    private func registerDefaultCCMModels() {
        let defaultCCMProvider = container.getIfRegistered(DefaultCCMProvider.self)
        try? defaultCCMProvider?.register(model: .init(enabled: 4),
                                          for: AttributionCCMModel.self)
    }
```

#### Pros:

- Easy to manage all ccm's at market level
- Can implement incrementally

#### Cons:

- Violate the single responsibility principle

## Recommended Solution

Option 2: Option 2 with Option 1, as this pattern helps the compile time validations that swift provides, the option 4 where checking at runtime is more prone to errors at markets level, 
The migration can be done by a new protocol with inheriting CCMModel , which would be `CCMModelDefaultsProvider`, from that all required CCMModels can be confirmed to this protocol can be used for defaults
Once all the ccmModels are migrated then we can replace the name to `CCMModel`

## Addendum

The following is a third option proposed by John Liedtke. The third option was not proposed initially since it did not originally seem feasible. 

### Option 3, inject default CCM during decoding process

The third option obviates the need for setting a global variable on the CCM model by providing the default CCM value during the decoding process. 

**Steps**

1. Update the CCM model to conform to `CCMDefaultableModel`.
2. Use the provided default CCM value during the decoding process. 
3. Update Plugin configuration to include a CCM default value.
4. Retrieve CCM publisher at Plugin Init, passing in the CCM default value from (3). 

#### Example

```swift
/// A model whose properties correspond to cloud configuration management (CCM) flags.
public protocol CCMDefaultableModel: Decodable, Equatable {

    /// The model initialized with appropriate default values.
    static var defaults: Self { get }

    /// Creates a new instance by decoding from the given decoder and using the provided defaults for missing
    /// flags.
    init(decoder: Decoder, defaultCCM: Self) throws

}

extension CCMDefaultableModel {

    public init(from decoder: Decoder) throws {
        try self.init(decoder: decoder, defaultCCM: Self.defaults)
    }
}

public struct MyPluginCCM {

    /// Whether `MyPlugin` is enabled.
    public var isEnabled: Bool = true

    public var subFeatureOne = SubFeatureOne()
    public var subFeatureTwo = SubFeatureTwo()

    public struct SubFeatureOne {

        /// Whether SubFeatureOne is enabled.
        public var enabled: Bool = false

        /// Determines the number of times a message is displayed to the user
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
        case isEnabled = "feature.subFeatureOne.enabled"
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

// retrieving CCM

public final class MyPluginAPI {

    public struct Configuration {
        public var ccm: InAppMessagingCCM = .defaults

        public init(ccm: InAppMessagingCCM = .defaults) {
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
}

```

**Pros**

- Obviates global mutable state (i.e. global mutable variables). 
- One initializer, synthesized initializer is maintained. 
- Local reasoning about the default values. The default values are set where the property is declared. 
- Straightforward unit testing, the defaults can be even be mocked! Additionally, only two types of unit tests are required, one test for validating the defaults and another to verify the JSON value takes precedence. 


**Cons**

* Migration of existing CCM models that have already been converted during the internationalization process.


#### Implementation 

Option 3 has already been implemented in this [PR](https://gecgithub01.walmart.com/walmart-ios/glass-app/pull/22261).


