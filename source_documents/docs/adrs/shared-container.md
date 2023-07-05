# Share container with app, extensions, and dependencies

- Authors: Hemanth Kumar
- Decision: Option 2

# Context

We want some type of shared assembly that sets up a container with all of the platform's dependencies. Currently, each market and extension 
has to remember which Platform dependencies exist and remember to register them.

We often encounter situations where new dependencies are added to the GlassAppProvider.walmartApp container.
1. This causes certain codepaths to trigger a force unwrap through `container.get()`.
2. This causes the container used by Intents & IntentsUI to miss these dependencies, leading to a crash when one of these codepaths is triggered.
3. This creates a need for sharing the same container across extensions, the main app target, its dependencies.

# Goal

Provide a shared platform container with all its dependencies registered to all across the app and its dependencies to avoid any discrepancies

# Options

## Option 1: Walmart platform allows injecting its dependencies [Similarly dependency-injection-vs-validation ADR: https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/adrs/dependency-injection-vs-validation.md]

This solution will ensure all dependencies will have all required interface dependencies with the platform.

#### Pros:
- All dependencies of the platform are managed in a single place, no more sudden surprise for its dependencies.
- Avoids duplication of code

#### Cons:
- It makes all dependencies of the platform available for the platform available regardless of their usage.

#### Example:

```swift
// A Platform Dependancies is responsible for capturing all its dependencies to be able to setup the shared platform container dependencies
public struct PlatformDependancies {
    let quimbyAuthProvider: ((Container) -> QuimbyAuthProviding)
    let quimbyCartProvider: ((Container) -> QuimbyCartProviding)
    let userAgentProvider: ((Container) -> UserAgentProviding)
    let analyticsStoreIDProvider: ((Container) -> AnalyticsStoreIDProviding)
    let analyticsMembershipProvider: ((Container) -> AnalyticsMembershipProviding)
    let analyticsCustomerInfoProvider: ((Container) -> AnalyticsCustomerInfoProviding)
    let analyticsLocationInfoProvider: ((Container) -> AnalyticsLocationInfoProviding)
    let addressConfigurationAPI: ((Container) -> AddressConfigurationAPI)
    let userDefaultsRegistrationAPI: ((Container) -> UserDefaultsRegistrationPlatformAPI)
    let analyticsPrivider: ((Container) -> Analytics)
    let storageAPIProvider: ((Container) -> StorageAPI)
    let analyticsSessionProvider: ((Container) -> AnalyticsSessionProvider)
    let appProcessInfoProvider:((Container) -> AppProcessInfo)
    let deepLinkerProvider:((Container) -> DeepLinker)
    let environmentAPIProvider:((Container) -> EnvironmentAPI)
    let errorCoordinatorProvider: ((Container) -> ErrorCoordinatorProducing)
    let locationUpdatesProvider:((Container) -> LocationUpdatesProvider)
    let cookieStorageAPIProvider: ((Container) -> CookieStorageAPI)
    let urlSessionConfigurationAPIProvider: ((Container) -> URLSessionConfigurationAPI)
    let preferencesStorageProvider: ((Container) -> PreferencesStorage)
    let networkInformationProvider: ((Container) -> NetworkInformationProvider)
    let notificationPermissionProvider: ((Container) -> NotificationPermissionProvider)
    let locationPermissionProvider: ((Container) -> LocationPermissionProvider)
    let appInfoAPIProvider: ((Container) -> AppInfoAPI)
    let startupPreferenceAPIProvider:((Container) -> StartupPreferenceAPI)
    let applicationURLHandlerProvider:((Container) -> ApplicationURLHandler)
    let videoPermissionProvider: ((Container) -> VideoPermissionProvider)
    let retryNotifyingFactoryProvider: ((Container) -> RetryNotifyingFactory)
    let reflectorManagerProvider: ((Container) -> ReflectorManager)
    let networkingProvider: ((Container) -> Networking)
    let mediaNetworkingProvider: ((Container) -> MediaNetworking)

    public init(analyticsStoreIDProvider: @escaping ((Container) -> AnalyticsStoreIDProviding),
                analyticsMembershipProvider: @escaping ((Container) -> AnalyticsMembershipProviding),
                analyticsCustomerInfoProvider: @escaping ((Container) -> AnalyticsCustomerInfoProviding),
                analyticsLocationInfoProvider: @escaping ((Container) -> AnalyticsLocationInfoProviding),
                addressConfigurationAPI: @escaping ((Container) -> AddressConfigurationAPI),
                quimbyAuthProvider: @escaping ((Container) -> QuimbyAuthProviding),
                quimbyCartProvider: @escaping ((Container) -> QuimbyCartProviding),
                userAgentProvider:@escaping ((Container) -> UserAgentProviding),
                userDefaultsRegistrationAPI:@escaping ((Container) -> UserDefaultsRegistrationPlatformAPI),
                analyticsPrivider:@escaping ((Container) -> Analytics),
                storageAPIProvider:@escaping ((Container) -> StorageAPI),
                analyticsSessionProvider:@escaping ((Container) -> AnalyticsSessionProvider),
                appProcessInfoProvider:@escaping ((Container) -> AppProcessInfo),
                deepLinkerProvider:@escaping ((Container) -> DeepLinker),
                environmentAPIProvider:@escaping ((Container) -> EnvironmentAPI),
                errorCoordinatorProvider:@escaping ((Container) -> ErrorCoordinatorProducing),
                locationUpdatesProvider:@escaping ((Container) -> LocationUpdatesProvider),
                cookieStorageAPIProvider:@escaping ((Container) -> CookieStorageAPI),
                urlSessionConfigurationAPIProvider:@escaping ((Container) -> URLSessionConfigurationAPI),
                preferencesStorageProvider:@escaping ((Container) -> PreferencesStorage),
                networkInformationProvider:@escaping ((Container) -> NetworkInformationProvider),
                notificationPermissionProvider:@escaping ((Container) -> NotificationPermissionProvider),
                locationPermissionProvider: @escaping ((Container) -> LocationPermissionProvider),
                appInfoAPIProvider:@escaping ((Container) -> AppInfoAPI),
                startupPreferenceAPIProvider:@escaping ((Container) -> StartupPreferenceAPI),
                applicationURLHandlerProvider:@escaping ((Container) -> ApplicationURLHandler),
                videoPermissionProvider:@escaping ((Container) -> VideoPermissionProvider),
                retryNotifyingFactoryProvider:@escaping ((Container) -> RetryNotifyingFactory),
                reflectorManagerProvider: @escaping ((Container) -> ReflectorManager),
                networkingProvider:@escaping ((Container) -> Networking),
                mediaNetworkingProvider:@escaping ((Container) -> MediaNetworking)) {
        self.analyticsStoreIDProvider = analyticsStoreIDProvider
        self.analyticsMembershipProvider = analyticsMembershipProvider
        self.analyticsCustomerInfoProvider = analyticsCustomerInfoProvider
        self.analyticsLocationInfoProvider = analyticsLocationInfoProvider
        self.quimbyAuthProvider = quimbyAuthProvider
        self.quimbyCartProvider = quimbyCartProvider
        self.addressConfigurationAPI = addressConfigurationAPI
        self.userAgentProvider = userAgentProvider
        self.userDefaultsRegistrationAPI = userDefaultsRegistrationAPI
        self.analyticsPrivider = analyticsPrivider
        self.storageAPIProvider = storageAPIProvider
        self.analyticsSessionProvider = analyticsSessionProvider
        self.appProcessInfoProvider = appProcessInfoProvider
        self.deepLinkerProvider = deepLinkerProvider
        self.environmentAPIProvider = environmentAPIProvider
        self.errorCoordinatorProvider = errorCoordinatorProvider
        self.locationUpdatesProvider = locationUpdatesProvider
        self.cookieStorageAPIProvider = cookieStorageAPIProvider
        self.urlSessionConfigurationAPIProvider = urlSessionConfigurationAPIProvider
        self.preferencesStorageProvider = preferencesStorageProvider
        self.networkInformationProvider = networkInformationProvider
        self.notificationPermissionProvider = notificationPermissionProvider
        self.locationPermissionProvider = locationPermissionProvider
        self.appInfoAPIProvider = appInfoAPIProvider
        self.startupPreferenceAPIProvider = startupPreferenceAPIProvider
        self.applicationURLHandlerProvider = applicationURLHandlerProvider
        self.videoPermissionProvider = videoPermissionProvider
        self.retryNotifyingFactoryProvider = retryNotifyingFactoryProvider
        self.reflectorManagerProvider = reflectorManagerProvider
        self.networkingProvider = networkingProvider
        self.mediaNetworkingProvider = mediaNetworkingProvider
    }
}

final class GlassAppProvider {
    private static let mainApp: Container = {
        let container = Container()

        let analyticsInfoProvider = AnalyticsFeatureInfoProvider(container: container)
        let dependancies = PlatformDependancies(analyticsStoreIDProvider: { _ in
            analyticsInfoProvider
        }, analyticsMembershipProvider: { _ in
            analyticsInfoProvider
        }, analyticsCustomerInfoProvider: { _ in
            analyticsInfoProvider
        }, analyticsLocationInfoProvider: { _ in
            analyticsInfoProvider
        }, addressConfiguration: { _ in
            addressConfigurationPlatformAPI.usa
        }, quimbyAuthProvider: { container in
            QuimbyAuthProvider(container: container)
        }, quimbyCartProvider: { container in
            QuimbyCartProvider(container: container)
        }, userAgentProvider:{ _ in
            GlassUserAgent()
        }, userDefaultsRegistrationAPI: { container in
            UserDefaultsRegistrationPlatformAPI(configuration: .usa, container: container)
        }, analyticsPrivider: { container in
            let analyticsSessionProvider: AnalyticsSessionProvider? = container.getIfRegistered()
            let manager = AsyncAnalyticsManager(sessionProvider: analyticsSessionProvider)
            manager.addAniviaIntent(container: container,
                                    configuration: .usa)
            return manager
        }, storageAPIProvider: { _ in
            StoragePlatformAPI()
        }, analyticsSessionProvider: { container in
            WalmartPlatform.AnalyticsSession(container: container)
        }, appProcessInfoProvider:{ _ in
            ProcessInfo.processInfo
        }, deepLinkerProvider: { container in
            let router = DeepLinkRouter()
            container.setupDeeplinkerDependancies(deeplinker: router)
            return router
        }, environmentAPIProvider: { container in
            EnvironmentPlatformAPI(container: container, configurations: .usa)
        }, errorCoordinatorProvider: { container in
            ErrorCoordinatorManager(container: container)
        }, locationUpdatesProvider: { _ in
            LocationUpdates(allowsBackgroundLocationUpdates: false)
        }, cookieStorageAPIProvider: { container in
            CookieStoragePlatformAPI(configuration: .usa, container: container)
        }, urlSessionConfigurationAPIProvider: { container in
            URLSessionConfigurationPlatformAPI(container: container,
                                               configuration: .usa)
        }, preferencesStorageProvider: { _ in
            return Preferences(configuration: .usa)
        }, networkInformationProvider: { _ in
            NetworkInformationMonitor()
        }, notificationPermissionProvider: { _ in
            NotificationPermission(options: [.alert, .sound, .badge])
        }, locationPermissionProvider: { _ in
            do {
                let permission = try LocationPermission(locationPermissionLevel: .whenInUse)
                return permission
            } catch {
                fatalError(error.localizedDescription)
            }
        }, appInfoAPIProvider: { container in
            AppInfoPlatformAPI(configuration: .usa)
        }, startupPreferenceAPIProvider: { container in
            StartupPreferencePlatformAPI(container: container)
        }, applicationURLHandlerProvider: { _ in
            UIApplication.shared
        }, videoPermissionProvider: { _ in
            do {
                let videoPermission = try VideoPermission()
                return videoPermission
            } catch {
                fatalError(error.localizedDescription)
            }
        }, retryNotifyingFactoryProvider: { container in
            RetryNotifierFactory(ccmSource: container.get())
        }, reflectorManagerProvider: { container in
            ReflectorManager(container: container)
        }, networkingProvider: { container in
            container.networkingProvider()
        }, mediaNetworkingProvider: { container in
            container.mediaNetworkProvider()
        })

        container.setupPlatformDependancies(dependancies: dependancies)

        return container
    }()
}

public extension Container {
    public func setupPlatformDependancies(dependancies: PlatformDependancies) {
        register(UserAgentProviding.self, dependancies.userAgentProvider )
        register(UserDefaultsRegistrationAPI.self, dependancies.userDefaultsRegistrationAPI)
        register(StorageAPI.self, dependancies.storageAPIProvider)
        register(AnalyticsStoreIDProviding.self, dependancies.analyticsStoreIDProvider )
        register(AnalyticsMembershipProviding.self, dependancies.analyticsMembershipProvider )
        register(AnalyticsCustomerInfoProviding.self, dependancies.analyticsCustomerInfoProvider )
        register(AnalyticsLocationInfoProviding.self, dependancies.analyticsLocationInfoProvider )
        register(Analytics.self, dependancies.analyticsPrivider)
        register(AnalyticsTracking.self) { $0.get(Analytics.self) }
        registerProvidingChild( AnalyticsSessionProvider.self, dependancies.analyticsSessionProvider)
        register(AppProcessInfo.self, dependancies.appProcessInfoProvider)
        registerProvidingChild(DeepLinker.self, dependancies.deepLinkerProvider)
        registerProvidingChild(ReflectorManager.self, dependancies.reflectorManagerProvider)
        register(EnvironmentAPI.self, dependancies.environmentAPIProvider)
        register(AddressConfigurationAPI.self, dependancies.addressConfigurationAPI )
        register(ErrorCoordinatorProducing.self, dependancies.errorCoordinatorProvider)
        register(LocationUpdatesProvider.self, dependancies.locationUpdatesProvider)
        register(CookieStorageAPI.self, dependancies.cookieStorageAPIProvider)
        register(URLSessionConfigurationAPI.self, dependancies.urlSessionConfigurationAPIProvider)
        register(PreferencesStorage.self, dependancies.preferencesStorageProvider)
        register(Networking.self, dependancies.networkingProvider )
        register(MediaNetworking.self, dependancies.mediaNetworkingProvider)
        register(NetworkInformationProvider.self, dependancies.networkInformationProvider)
        register(NotificationPermissionProvider.self, dependancies.notificationPermissionProvider)
        register(LocationPermissionProvider.self, dependancies.locationPermissionProvider)
        register(AppInfoAPI.self, dependancies.appInfoAPIProvider)
        register(StartupPreferenceAPI.self, dependancies.startupPreferenceAPIProvider)
        register(ApplicationURLHandler.self, dependancies.applicationURLHandlerProvider)
        register(VideoPermissionProvider.self, dependancies.videoPermissionProvider)
        register(RetryNotifyingFactory.self, dependancies.retryNotifyingFactoryProvider)
        register(QuimbyAuthProviding.self, dependancies.quimbyAuthProvider )
        register(QuimbyCartProviding.self, dependancies.quimbyCartProvider )
    }
}
```

### Option 2: Similar to Option 1, Walmart platform allows injecting its dependencies via configurations

To avoid any unwanted dependencies in the platform which are not required at the target level

#### Pros:
- All depedancies can be configurable

#### Cons:
- Requires lot of modifications on existing dependancies to be able to configurable

```swift
public struct PlatformConfig {
    let userDefaultsRegistrationAPIConfig: UserDefaultsRegistrationPlatformAPI.Configuration
    let aniviaSuperAttributesConfiguration: AniviaSuperAttributes.Configuration
    let environmentPlatformAPIAPIConfig: EnvironmentPlatformAPI.Configurations
    let urlSessionConfigurationPlatformAPIConfig: URLSessionConfigurationPlatformAPI.Configuration
    let networkingConfig: Networking.Configuration
    let appInfoPlatformAPIConfig: AppInfoPlatformAPI.Configuration
    let preferencesConfig: Preferences.Configuration
    let cookieStoragePlatformAPIConfig: CookieStoragePlatformAPI.Configuration

    public init(userDefaultsRegistrationAPIConfig: UserDefaultsRegistrationPlatformAPI.Configuration,
                aniviaSuperAttributesConfiguration: AniviaSuperAttributes.Configuration,
                environmentPlatformAPIAPIConfig: EnvironmentPlatformAPI.Configurations,
                urlSessionConfigurationPlatformAPIConfig: URLSessionConfigurationPlatformAPI.Configuration,
                networkingConfig: Networking.Configuration,
                appInfoPlatformAPIConfig: AppInfoPlatformAPI.Configuration,
                preferencesConfig: Preferences.Configuration,
                cookieStoragePlatformAPIConfig: CookieStoragePlatformAPI.Configuration) {
        self.userDefaultsRegistrationAPIConfig = userDefaultsRegistrationAPIConfig
        self.aniviaSuperAttributesConfiguration = aniviaSuperAttributesConfiguration
        self.environmentPlatformAPIAPIConfig = environmentPlatformAPIAPIConfig
        self.cookieStoragePlatformAPIConfig = cookieStoragePlatformAPIConfig
        self.urlSessionConfigurationPlatformAPIConfig = urlSessionConfigurationPlatformAPIConfig
        self.networkingConfig = networkingConfig
        self.preferencesConfig = preferencesConfig
        self.appInfoPlatformAPIConfig = appInfoPlatformAPIConfig
    }
}

extension Container {
    public func setupPlatformdependencies(with configuration: PlatformConfig) {
        register(UserDefaultsRegistrationAPI.self) {
            UserDefaultsRegistrationPlatformAPI(configuration: configuration.userDefaultsRegistrationAPIConfig,
                                                container: $0)
        }

        register(Analytics.self) { (container) in
            let analyticsSessionProvider: AnalyticsSessionProvider? = container.getIfRegistered()
            let manager = AsyncAnalyticsManager(sessionProvider: analyticsSessionProvider)
            manager.addAniviaIntent(container: container,
                                    configuration: configuration.aniviaSuperAttributesConfiguration)
            return manager
        }

        register(EnvironmentAPI.self) {
            EnvironmentPlatformAPI(container: $0,
                                   configurations: configuration.environmentPlatformAPIAPIConfig)
        }
        register(URLSessionConfigurationAPI.self) {
            URLSessionConfigurationPlatformAPI(container: $0,
                                               configuration: configuration.urlSessionConfigurationPlatformAPIConfig)
        }
        setupNetworking(setupTracking: true, configuration: configuration.networkingConfig)
        register(AppInfoAPI.self, {
            AppInfoPlatformAPI(configuration: configuration.appInfoPlatformAPIConfig) })

        register(CookieStorageAPI.self) {
            CookieStoragePlatformAPI(configuration: cookieStoragePlatformAPIConfig,
                                     container: $0)
        }
        setupPreferencesStorage(configuration: preferencesConfig)
    }
}

final class GlassAppProvider {
    private static let mainApp: Container = {
        let container = Container()
        let analyticsInfoProvider = AnalyticsFeatureInfoProvider(container: container)
        let config = PlatformConfig(userDefaultsRegistrationAPIConfig: .usa,
                                    aniviaSuperAttributesConfiguration: .usa,
                                    environmentPlatformAPIAPIConfig: .usa,
                                    urlSessionConfigurationPlatformAPIConfig: .usa,
                                    networkingConfig: .usa,
                                    appInfoPlatformAPIConfig: .usa,
                                    preferencesConfig: .usa,
                                    cookieStoragePlatformAPIConfig: .usa,)
        container.setupPlatformdependencies(with: config)
        return container
    }
}

```

### Option 3: Each market will have its own platform container dependencies, and share the file across the extension targets

#### Pros:
- Container dependencies are managed at the market level
- Less code modification

#### Cons:
- Need to duplicate based on the code changes in Platform for each market respectively.


## Recommended Solution

Option 2, walmart platform allows injecting its dependencies via configurations
