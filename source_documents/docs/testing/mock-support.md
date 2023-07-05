---
title: Mock Support
navTitle: Mock Support
---

# Mock Support

- [What is mock?](#what-is-mock)
- [Why do you want to mock?](#why-do-you-want-to-mock)
- [Available Mocks](#available-mocks)

## What is Mock?

Mocking is a key technique when it comes to writing unit tests in pretty much any language. When mocking an object, we are essentially creating a "fake" version of it - with the same API as the real one - in order to more easily be able to assert and verify outcomes in our test cases.

## Why do you want to mock?

Our app consists of many different components working together. Mocking is a technique to isolate dependencies so we can test one component at a time. Writing mocks to isolate other components so we can test the behavior of the subject under test. Mocking also allows us to introspect our mock objects to verify their observed behavior matches our expected behavior.

## Available Mocks

- [AppInfoAPIMock](#appInfoapimock)
- [FixtureMocking](#fixturemocking)
- [MockATTPermissionProvider](#mockattpermissionprovider)
- [MockAnalyticsIntent](#mockanalyticsintent)
- [MockAnalyticsSessionProvider](#mockanalyticssessionprovider)
- [MockAppVersionProvider](#mockappversionprovider)
- [MockApplicationURLHandler](#mockapplicationurlhandler)
- [MockBackgroundTaskManaging](#mockbackgroundtaskmanaging)
- [MockCCMSource](#mockccmsource)
- [MockClock](#mockclock)
- [MockCookieStorageAPI](#mockcookiestorageapi)
- [MockDeepLinker](#mockdeeplinker)
- [MockDeviceInfoProvider](#mockdeviceinfoprovider)
- [MockDownloadTaskResult](#mockdownloadtaskresult)
- [MockDownloadTask](#mockdownloadtask)
- [MockError](#mockerror)
- [MockExternalURLHandler](#mockexternalurlhandler)
- [MockGlassAppProvider](#mockglassappprovider)
- [MockLocationGeocoding](#mocklocationgeocoding)
- [MockLocationPermissionProvider](#mocklocationpermissionprovider)
- [MockNetworkInformationProvider](#mocknetworkinformationprovider)
- [MockNotificationPermissionProvider](#mocknotificationpermissionprovider)
- [MockPlacemark](#mockplacemark)
- [MockPreferencesStorage](#mockpreferencesstorage)
- [MockQuimbyProvider](#mockquimbyprovider)
- [MockQuimbySource](#mockquimbysource)
- [MockRemoteImageSource](#mockremoteimagesource)
- [MockRemoteMethod](#mockremotemethod)
- [MockRetryNotifierFactory](#mockretrynotifierfactory)
- [MockSystemGeocoder](#mocksystemgeocoder)
- [MockURLScheme](#mockurlscheme)
- [MockURLSessionConfigurationAPI](#mockurlsessionconfigurationapi)
- [MockUniversalLink](#mockuniversallink)
- [MockUserAgentProvider](#mockuseragentprovider)
- [MockVideoPermissionProvider](#mockvideopermissionprovider)
- [MockWindow](#mockwindow)
- [Mocking](#mocking)
- [PlatformMocks](#platformmocks)
- [StorageAPIMock](#storageapimock)

### AppInfoAPIMock

The API for accessing information about the current application.
The `AppInfoAPIMock.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/AppInfoAPIMock.swift).

#### Usage
```swift
private var container: Container {
    let container = Container(parent: .mock)
    container.register(AppInfoAPI.self) { AppInfoAPIMock() }
    return container
}
```
```swift
// ...
container.appInfoAPIMock.appStoreURL = URL(string: "test://")!
```
### FixtureMocking

A `Networking` instance that loads response data from resource files. To set up fixtures:
1. Create a "Fixtures" folder and add it as a _folder reference_ in in your project.
2. Add the "Fixtures" folder to your _tests_ target.

Then, for each test that uses a fixture, create a response file in the "Fixtures" folder.

For example, for the test function `MyTests.testSomeRequest()`, the response data should be in the file:
"Fixtures/MyTests/testSomeRequest/response.json". The `FixtureMocking.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/FixtureMocking.swift).

#### Usage
```swift
class MyTests: XCTestCase {
    func testSomeRequest() throws {
        let url = URL(fixturesFor: self)
        let value = try waitForValue(Networking.fixtures().dataTaskPublisher(for: url))
        // `value.data` is the contents of `Fixtures/MyTests/testSomeRequest/response.json`

        XCTAssertEqual(value.data, expectedData)
    }
}
```
### MockATTPermissionProvider

`MockATTPermissionProvider` Provides an interface for requesting permssion to get tracking data for ads.
The `MockATTPermissionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockATTPermissionProvider.swift).

#### Usage
```swift
container.register(ATTPermissionProvider?.self) { MockATTPermissionProvider.init() }
```

### MockAnalyticsIntent

`MockAnalyticsIntent` provides an analytics intent intended for testing. This can be added to a concrete implementation of `Analytics` to start collecting events. The `MockAnalytics.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockAnalytics.swift).

#### Usage
Sets up Mock Analytics in the container as follows:
`case .unitTest:` Configures mock analytics for unit testing using an associatated MockAnalyticsIntent that tests can use to verify the correct values are passed to the AnalyitcsManager.
```swift
let container = Container(parent: .mock)
let intent = MockAnalyticsIntent()
container.setupMockAnalytics(for: .unitTest(intent))
```

`case .consoleLogger:` Configures mock analytics for demo and test apps with a ConsoleLoggerIntent that will log analytics payloads to the console for easy verification of event payloads during development. You must still setup the LogHandler for the cosole.
```swift
let container = Container(parent: .mock)
container.setupMockAnalytics(for: .consoleLogger(logIdentifier: "MySuperRadFeature"))
LoggingSystem.bootstrap { label -> LogHandler in
    var result = OSLogHandler(label: label)
    result.logLevel = .debug
    return result
}

```
`case .noTracking:` Configures mock analytics with no additional tracking functionality.
```swift
let container = Container(parent: .mock)
container.setupMockAnalytics(for: .noTracking)
```

```swift
let container = Container(parent: .mock)
let intent = MockAnalyticsIntent()
container.setupMockAnalytics(for: .unitTest(intent))
```
### MockAnalyticsSessionProvider

`MockAnalyticsSessionProvider` A session provider used to record tracked event counts.
The `MockAnalyticsSessionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockAnalyticsSessionProvider.swift).

#### Usage
```swift
container.register(AnalyticsSessionProvider.self, MockAnalyticsSessionProvider.init)
```
```swift
let originalPayloadAttributes = coordinator.testHooks.payloadAttributes
let mockAnalytics = container.get() as AnalyticsSessionProvider as! MockAnalyticsSessionProvider
mockAnalytics.sessionId = "test_sid"
try waitForCondition(coordinator.testHooks.payloadAttributes.sessionId != originalPayloadAttributes.sessionId)
```
```swift
let expectation = expectation(description: #function)
let mockAnalyticsSession = MockAnalyticsSessionProvider()
mockAnalyticsSession.updateSessionHandler = { expectation.fulfill() }
// perform the action
wait(for: [expectation])
```
### MockAppVersionProvider

`MockAppVersionProvider` an interface to determine the app version. The `MockAppVersionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockAppVersionProvider.swift).

#### Usage
```swift
// Given
let inputPayload = """
{
...
}
"""
decoder.setAppVersionProvider(MockAppVersionProvider(major: 21, minor: 26, patch: 4))

// When
..
```
```swift
let versionProvider = MockAppVersionProvider(major: 21)
lazy var decoder: JSONDecoder = {
    let decoder = JSONDecoder()
    decoder.setAppVersionProvider(versionProvider)
    return decoder
}()
```
### MockApplicationURLHandler

`MockApplicationURLHandler` Provides an interface to mock URL handler to track the coordinator opening the URL. The `MockApplicationURLHandler.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockApplicationURLHandler.swift).

#### Usage
```swift
// Given
container.register(ApplicationURLHandler.self, { MockApplicationURLHandler() })
```
```swift
let url = URL(string: "http://www.mock.com/")!
let mockUrlHandler = MockApplicationURLHandler()

..
// Then
XCTAssertEqual(mockUrlHandler.openedURL, url)
XCTAssertEqual(mockUrlHandler.openCount, 1)
```
### MockBackgroundTaskManaging

`MockBackgroundTaskManaging` that manages executing a task in the background. The `MockBackgroundTaskManaging.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockBackgroundTaskManaging.swift).

#### Usage
```swift
let mockBackgroundManager = MockBackgroundTaskManaging()
..
operation.testHooks.setBackgroundTaskManager(mockBackgroundManager)

// Then
XCTAssertEqual(mockBackgroundManager.beginBackgroundTaskCount, 1)
XCTAssertEqual(mockBackgroundManager.endBackgroundTaskCount, 1)
```
### MockCCMSource

The source for obtaining Cloud Configuration Management (CCM) data. The MockCCMSource returns a registered mock publisher or a publisher with the default ccm model (`Model()`). The `MockCCMSource.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockCCMSource.swift).

#### Usage
```swift
class Middleware: NSObject {
    init(ccmSource: CCMSource) {
        self.ccmSource = ccmSource
    }
}
```
```swift
let ccm = CCM(isEnabled: true)
let mockCCMSource = MockCCMSource()
let publisher = Just(ccm).eraseToAnyValuePublisher()
mockCCMSource.register(publisher)

let middleware = Middleware(ccmSource: mockCCMSource)
// .. 
```
### MockClock

An artificial Clock type that allows the current time to be advanced manually. The default 'now' for the mock clock is 2019-05-19 23:06:40 +0000 (or 580,000,000 seconds since the reference date). The `MockClock.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockClock.swift).

#### Usage
```swift
private var mock: MockClock!

override func setUp() {
    super.setUp()
    mock = MockClock()
    Clock.TestHooks.using(customClock: mock)
}
```
```swift
let mockClock = MockClock()
Clock.TestHooks.using(customClock: mockClock)

timer = ClockTimer.scheduledTimer(
    withTimeInterval: 0.1,
    repeats: false)
{ (_) in
    actionPerformer.performAction()
}
XCTAssertTrue(timer is ClockTimer)
```
```swift
...
let mockClock = MockClock()
let startDate = Date(using: mockClock)

let endMockClock = MockClock()
endMockClock.advance(by: 3600 * 24 * 7)
let endDate = Date(using: endMockClock)
```
### MockCookieStorageAPI

API for retrieving the app groups `CookieStorage`. The `MockCookieStorageAPI.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockCookieStorageAPI.swift).

#### Usage 
```swift
private var container: Container {
    let container = Container(parent: .mock)
    container.register(CookieStorageAPI.self) { MockCookieStorageAPI() }
    return container
}

private var mockCookieStore: MockCookieStorage {
    container.get(MockCookieStorageAPI.self).mockStorage
}

// ..
XCTAssertTrue(mockCookieStore.storedCookies.isEmpty)
```
```swift
private var cookieStorage = MockCookieStorage()

override func setUp() {
    super.setUp()

    cookieStorage = MockCookieStorage()
    container = Container(parent: .mock)
    let cookieAPI = MockCookieStorageAPI()
    cookieAPI.mockStorage = cookieStorage
    container.register(CookieStorageAPI.self) { cookieAPI }
}

// Given
cookieStorage.addMockCookie(name: "mock", value: "mock id")
// ..
```
### MockDeepLinker

`MockDeepLinker` Provides an interface for handling mock deeplinks. The `MockDeepLinker.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockDeepLinker.swift).

#### Usage 
```swift
container.register(DeepLinker.self) { MockDeepLinker() }
```
```swift
// given
..
let deepLinker = MockDeepLinker()
container.register(DeepLinker.self) { deepLinker }
// when
// perform the deeplink ex; "testUrl"
..
//then
try waitForCondition(deepLinker.currentDeeplink != nil)
XCTAssertNotNil(deepLinker.currentDeeplink?.universalURL)
XCTAssertEqual(deepLinker.currentDeeplink?.url.absoluteString, "walmart://testUrl")
```

### MockDeviceInfoProvider

API for retrieving the device informations. The `MockDeviceInfoProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockDeviceInfoProvider.swift).

#### Usage 
```swift
let mockDeviceInfoProvider = MockDeviceInfoProvider()
mockDeviceInfoProvider.idfv = "564032c4-d474-11eb-b8bc-0242ac130003"
// ..
XCTAssertEqual(request.idfv, "564032c4-d474-11eb-b8bc-0242ac130003")
```
### MockDownloadTaskResult

A mock `DownloadTaskResult` to hold the result of URLSession's DownloadTask completion. The `MockDownloadTaskResult` can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockDownloadTask.swift).

#### Usage 
```swift
// Create a mock response
let mockURLResponse: URLResponse = HTTPURLResponse(url: URL(string: "https://local.com")!,
                                                   statusCode: 200,
                                                   httpVersion: nil,
                                                   headerFields: nil)!
// Create a mock download task result
let mockDownloadTaskResult = MockDownloadTaskResult(url: URL(string: "../tempDirectory/image.png")!,
                                                    response: mockURLResponse,
                                                    error: nil)
MockDownloadTask.downloadTaskResult(for: url, response: mockDownloadTaskResult)

let expectation = self.expectation(description: "downloadTask completion called")
networking.downloadTask(with: url, completionHandler: { (_, _, _) in
    expectation.fulfill()
}).resume()
wait(for: [expectation])
```
### MockDownloadTask

`MockDownloadTask` allows to mock URLSession DownloadTask. The `MockDownloadTask.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockDownloadTask.swift).

#### Usage 
```swift
let config = URLSessionConfiguration.default
config.protocolClasses = [MockDownloadTask.self]
let session = URLSession(configuration: config)
let networking = Networking(
    session: session,
    configuration: .init(
        isAPQClientEnabled: false,
        isAPQInterceptorEnabled: false
    )
)

....
let url = URL(mockingString: "test://")!

// Create a mock response
let mockURLResponse: URLResponse = HTTPURLResponse(url: URL(string: "https://local.com")!,
                                                   statusCode: 200,
                                                   httpVersion: nil,
                                                   headerFields: nil)!
// Create a mock download task result
let mockDownloadTaskResult = MockDownloadTaskResult(url: URL(string: "../tempDirectory/image.png")!,
                                                    response: mockURLResponse,
                                                    error: nil)
MockDownloadTask.downloadTaskResult(for: url, response: mockDownloadTaskResult)

let expectation = self.expectation(description: "downloadTask completion called")
networking.downloadTask(with: url, completionHandler: { (_, _, _) in
    expectation.fulfill()
}).resume()
wait(for: [expectation])
```
### MockError

`MockError` Provides an interface for handling mock error. The `MockError.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockError.swift).

#### Usage 
```swift
// When
let error = try waitForFailure(publisher) {
    upstream.send(completion: .failure(MockError()))
}

// Then
XCTAssertTrue(error is MockError)
```
```swift
..
mockService.error = MockError()
```
```swift
..
let error = MockError(message: "this is a test")
// When
..
```
### MockExternalURLHandler

`MockExternalURLHandler` Provides an interface to handle open URL in SafariViewController. The `MockExternalURLHandler.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockExternalURLHandler.swift).

#### Usage 
```swift
container.register(ExternalURLHandlerProtocol.self, MockExternalURLHandler.init)
```
```swift
let urlHandler = MockExternalURLHandler()
..

try waitForCondition(!urlHandler.invokedURLs.isEmpty)
let handledURL = try XCTUnwrap(urlHandler.invokedURLs.first)
XCTAssertEqual(handledURL.absoluteString, url.absoluteString)
XCTAssertEqual(urlHandler.openInSafariMethodCalled, 1)
```

### MockGlassAppProvider

Provides the main mock container used by the application. The `MockGlassAppProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockGlassAppProvider.swift).

#### Usage 
```swift
override class func setUp() {
    super.setUp()
    Container.configure(containerAppProvider: MockGlassAppProvider.self)
}
```
### MockLocationGeocoding

`MockLocationGeocoding` Provides an interface for mock geocoding based on a given location. The `MockLocationGeocoding.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockLocationGeocoding.swift).

#### Usage 
```swift
container.register(LocationGeocoding.self, { return MockLocationGeocoding() })
```

```swift
let expectedError2 = MockError.fail2
testHooks.setClient(to: MockLocationClient(result: .init(assigned: true)))
testHooks.setGeocoder(to: MockLocationGeocoding(.failure(expectedError2)))
let error2 = try waitForFailure(plugin.updateStoreForLocation())
XCTAssertEqual(error2 as? MockError, expectedError2)
```
### MockLocationPermissionProvider

API for `LocationPermission` allows for requesting permission to get location updates. `MockLocationPermissionProvider` provides a permission updates for testing. The `MockLocationPermissionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockLocationPermissionProvider.swift).

#### Usage 
```swift
private var container: Container {
    let container = Container(parent: .mock)
    container.register(LocationPermissionProvider.self) { MockLocationPermissionProvider.init() }
    return container
}

let someAPI = SomeAPI(locationPermissionProvider: MockLocationPermissionProvider())
```
### MockNetworkInformationProvider

A Mock `NetworkInformationProvider` that allows you to easily tune the `NetworkingInformation` using the `.mockNetworkingInfo` property. The `MockNetworkInformationProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockNetworkInformationProvider.swift).

#### Usage 
```swift
container.register(NetworkInformationProvider.self, MockNetworkInformationProvider.init)
```
```swift
var networkInformationProvider = MockNetworkInformationProvider()
..
networkInformationProvider.mockNetworkingInfo = .init(isReachable: true, interfaceType: .cellular)
```
### MockNotificationPermissionProvider

`MockNotificationPermissionProvider` Provides an interface to handle Notification permission. The `MockNotificationPermissionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockNotificationPermissionProvider.swift).

#### Usage 
```swift
container.register(NotificationPermissionProvider.self, { MockNotificationPermissionProvider() })
```
### MockPlacemark

`MockPlacemark` Provides a mock placemark data for a geographic location. The `MockPlacemark.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockPlacemark.swift).

#### Usage 
```swift
let geocoder = LocationGeocoder(geocoder: MockSystemGeocoder(.success([MockPlacemark()])))

let placemark = try waitForValue(geocoder.geocode(CLLocation(latitude: 1, longitude: 1)))

XCTAssertNotNil(placemark)
```

```swift
let utils = StoreInfoUtils()
placemark = MockPlacemark()
placemark!.location = CLLocation(latitude: 0, longitude: 0)
utils.locationGeocoder = LocationGeocoder(geocoder: MockSystemGeocoder(.success([placemark!])))
```
### MockPreferencesStorage

`MockPreferencesStorage` dealing with user data that would generally store in UserDefaults for testing. The `MockPreferencesStorage.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockPreferencesStorage.swift).

#### Usage 
Option 1
```swift
 container.register(PreferencesStorage.self, MockPreferencesStorage.init)
```
Option 2
```swift
 let preferencesStorage = MockPreferencesStorage()
 var somePreference = SomePreference()
 somePreference.isEnabled = true
 try? preferencesStorage.setUserDefaultsPreference(somePreference)
```
Option 3
```swift
 let preferencesStorage = MockPreferencesStorage()
 var somePreference = SomePreference()
 somePreference.isEnabled = true
 try? preferencesStorage.setKeychainPreference(somePreference)
```
### MockQuimbyProvider

Protocol to get Quimby authentication and cart context. The `MockQuimbyProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockQuimbyProvider.swift).

#### Usage 
```swift
var mockQuimbyProvider: MockQuimbyProvider!

override class func setUp() {
    super.setUp()
    let container = Container(parent: .mock)
    mockQuimbyProvider = .init()
    container.register(QuimbyAuthProviding.self) { self.mockQuimbyProvider }
    container.register(QuimbyCartProviding.self) { self.mockQuimbyProvider }
}

// ..
mockQuimbyProvider.mockAuthentication(true)
try waitForCondition(self.quimbySource.fetches == 1)

// ..
mockQuimbyProvider.mockAuthentication(false)
try waitForCondition(self.quimbySource.fetches == 1)
```
### MockQuimbySource

The source for obtaining `CCMModel`s. The `MockQuimbySource.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockQuimbySource.swift).

#### Usage 
```swift
private var mockQuimby: MockQuimbySource<SomeMockCCM>!

override class func setUp() {
    super.setUp()
    let container = Container(parent: .mock)
    mockQuimby = .init()
    container.register(QuimbySource.self, { self.mockQuimby })
}

// ..
var serverTimingCCM = SomeMockCCM(enabled: true)
_ = try waitForValue(mockQuimby.publisher(for: SomeMockCCM.self)) {
    self.mockQuimby.mockModelPublisher.send(serverTimingCCM)
}
```
```swift
container.register(QuimbySource.self, MockQuimbySource<SomeCCMModel>.init)
```
### MockRemoteImageSource

`MockRemoteImageSource` Provides an interface for handling image source. The `MockRemoteImageSource.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockRemoteImageSource.swift).

#### Usage 
```swift
let baseSource = MockRemoteImageSource()
let cachingSource = baseSource.caching()

let url = URL(string: "https://placehold.it/500x300")!

let value = try waitForValue(cachingSource.imagePublisher(for: url)) {
    baseSource.succeed(response: UIColor(red: 0x33/0xFF, green: 0x66/0xFF, blue: 0x99/0xFF, alpha: 1).image!)
}

XCTAssertEqual(value.pixels, [0x33, 0x66, 0x99, 0xFF])
```
```swift
..
container.setupRemoteImageSource()
RemoteImage.defaultSource = MockRemoteImageSource()
```

### MockRemoteMethod

`MockRemoteMethod` Provides an interface to more easily implement a mock (publisher-based) Network Client or Data Source. The `MockRemoteMethod.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockRemoteMethod.swift).

#### Usage 
```swift
let someMethod = MockRemoteMethod<RequestStruct, ResponseStruct>()

func some(request: RequestStruct) -> AnyPublisher<ResponseStruct, Error> {
    someMethod.publisher(for: request).eraseToAnyPublisher()
}
```

### MockRetryNotifierFactory

A factory that produces a publisher for retrying operations that have failed from a `RetryableError`. The `MockRetryNotifierFactory.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockRetryNotifierFactory.swift).

#### Usage 
```swift
var factory = MockRetryNotifierFactory()

// never attempt to retry
factory.retryPublisherOverride = Empty().eraseToAnyPublisher()

// ..
XCTAssertEqual(mocks.factory.retryPublisherCount, 1)
```
```swift
container.register(RetryNotifyingFactory.self, { MockRetryNotifierFactory() })
```
### MockSystemGeocoder

`MockSystemGeocoder` Provides an interface for mock up coordinate(with `Placemark`). The `MockSystemGeocoder.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockSystemGeocoder.swift).

#### Usage 
```swift
let geocoder = LocationGeocoder(geocoder: MockSystemGeocoder(.success([MockPlacemark()])))
let placemark = try waitForValue(geocoder.geocode(CLLocation(latitude: 1, longitude: 1)))
XCTAssertNotNil(placemark)
```
### MockURLScheme

`MockURLScheme` Configure mock URL scheme. The `MockURLScheme.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockURLScheme.swift).

#### Usage 
```swift
let linker: DeepLinker = DeepLinkRouter()
linker.urlSchemes = [MockURLScheme(name: "walmart", description: "Some test scheme")]
```
```swift
container.register(DeepLinker.self) {
     let router = DeepLinkRouter()
     router.tabBarControllerProvider = { tabBarController }
     router.urlSchemes = [
         MockURLScheme(name: "walmart", description: "walmart scheme"),
         MockURLScheme(name: "walmart", description: "walmart scheme")
     ]
     router.universalLinks = [MockUniversalLink(host: "beta.walmart.com",
                                                requiresRedirect: false,
                                                shouldOpenExternally: false)]
     return router
}
```

### MockURLSessionConfigurationAPI

API for creating URLSessionConfigurations. The `MockURLSessionConfigurationAPI.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockURLSessionConfigurationAPI.swift).

#### Usage 
```swift
override func setUp() {
    super.setUp()
    // ..

    let configurationAPI = MockURLSessionConfigurationAPI(protocolClasses: [Mocking.self])
    container.register(URLSessionConfigurationAPI.self) { configurationAPI }
    let configuration = configurationAPI.createURLSessionConfiguration()
    session = URLSession(configuration: configuration)
    // ..
```
```swift
container.register(URLSessionConfigurationAPI.self) { MockURLSessionConfigurationAPI() }
```
### MockUniversalLink
`MockUniversalLink` Defines the configuration for handling a universal link. The `MockUniversalLink.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockUniversalLink.swift).
#### Usage 
```swift
let linker = DeepLinkRouter()
linker.urlSchemes = [MockURLScheme(name: "tests", description: "Some test scheme")]
linker.universalLinks = [MockUniversalLink(host: "www.tests.com",
                                           requiresRedirect: false,
                                           shouldOpenExternally: false,
                                           replacementString: "tests://")]
```
```swift
container.register(DeepLinker.self) {
     let router = DeepLinkRouter()
     router.tabBarControllerProvider = { tabBarController }
     router.urlSchemes = [
         MockURLScheme(name: "walmart", description: "walmart scheme"),
         MockURLScheme(name: "walmart", description: "walmart scheme")
     ]
     router.universalLinks = [MockUniversalLink(host: "beta.walmart.com",
                                                requiresRedirect: false,
                                                shouldOpenExternally: false)]
     return router
}
```

### MockUserAgentProvider

Protocol defining the requirements for an object the provides an apps user agent details. The `MockUserAgentProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockUserAgentProvider.swift).

#### Usage 
```swift
// ..
let url = URL(string: "https:/mock.com")!
let networking = AniviaNetworking(
    url: URL(string: "https:/mock.com")!,
    userAgent: MockUserAgentProvider(key: "UserAgent", value: "TestUserAgentValue"),
    networking: Networking(
        session: .aniviaMock,
        configuration: .init(
            isAPQClientEnabled: false,
            isAPQInterceptorEnabled: false
        ))
    )
```
```swift
container.register(UserAgentProviding.self) {
    MockUserAgentProvider(key: "UserAgent", value: "Test/1.0 iOSTests/1.0")
}
```
### MockVideoPermissionProvider

`MockVideoPermissionProvider` Provides an interface to handle Video permission. The `MockVideoPermissionProvider.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockVideoPermissionProvider.swift).

#### Usage 
```swift
container.register(VideoPermissionProvider.self, { MockVideoPermissionProvider() })
```
### MockWindow

`MockWindow` Creates a mock window and sets the provided view controller as the `UIWindow.rootViewController`. The `MockWindow.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/MockWindow.swift).

#### Usage 
```swift
let window = MockWindow()
window.rootViewController = viewController

// When
window.makeKeyAndVisible()
try waitForViewControllerToAppear(viewController: viewController)
```
```swift
let mockViewController = UIViewController()
let mockWindow = MockWindow(rootViewController: mockViewController)
mockWindow.makeKeyAndVisible()

try waitForViewControllerToAppear(viewController: mockViewController)
```
### Mocking

`Mocking` provides an easy way to mock your service requests for the purposes of testing. The `Mocking.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/Mocking.swift).

#### Usage 
```swift
private var cancellables = Set<AnyCancellable>()

func setUp() {
    super.setUp()
    let config = URLSessionConfiguration.default
    config.protocolClasses = [Mocking.self]
    let session = URLSession(configuration: config)
}

func testMyThing() {
    let url = URL(mockingString: "https://myurl.com") // this will have a file and function parameter attached
    Mocking.mockResponse(for: url, .success((Data(), URLResponse())))
    session.dataTaskPublisher(for: url).store(in: &cancellables)
}
```
### PlatformMocks

To register `registerPlatformMocks` and access the different mocks. The `Container+PlatformMocks.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/Container+PlatformMocks.swift).

#### Usage 
```swift
// ..
container.registerPlatformMocks()
```
```swift
// ..
var startupPreference = container.startupPreferenceAPIMock
startupPreference.rampMode = .store
startupPreference.storeRampEnabled = true
```
```swift
// ..
var startupPreference = container.startupPreferenceAPIMock
startupPreference.rampMode = .store
startupPreference.storeRampEnabled = true
```
### StorageAPIMock

`StorageAPIMock` an interface for mock the cache and storage options for your data. The `StorageAPIMock.swift` file can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/TestUtilities/MockSupport/Mocks/StorageAPIMock.swift).

#### Usage 
```swift
// ..
container.register(StorageAPI.self) { StorageAPIMock() }
```
