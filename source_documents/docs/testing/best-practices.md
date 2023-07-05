---
title: Testing - Best Practices
navTitle: Best Practices
---

# Best Practices

## Use Test Doubles to Mock External Dependencies

You should write a test double whenever a source of data for your functionality can be mutated by something that isn't controlled by your code. These can include:

- Apple frameworks
- Other plugins
- Other modules
- 3rd party dependencies

Example:

If a requirement necessitated the usage of `UserDefaults`, one might think that you should just utilize `UserDefaults.standard` directly for your needs. However, one should consider the implications of this. This data store is maintained by Apple and you'd need to both interact with the Apple apis and the disk for persistence.

The recommendation would be instead to do something like the following:

```swift
protocol Defaults {
    func set(_ value: Any?, forKey: String)
}

extension UserDefaults: Defaults {}
```

This now allows you to interact with `UserDefaults` through the protocol `Defaults`

```swift
class MyClass {
    private let defaults: Defaults
    init(defaults: Defaults = UserDefaults.standard) {
        self.defaults = defaults
    }
    
    ...
}
```

In the tests, one can now:

- Create a mock implementation of `Defaults`
- Pass in the mock implementation to the intializer which will cause `MyClass` to utilize the mock implementation instead of `UserDefaults`

This results in a more deterministic test because you can directly control how defaults are managed and you can test the data storage directly.

## Always Make Assertions

Test should always be asserting that the conditions that are being tested are met. A test without an assertion is a useless test because it can never fail and is a waste of CI resources.

## Code Coverage

Code coverage is important but it's also important to avoid adding tests only for the sake of increasing code coverage. Make sure your tests are testing real use cases and not just executing a chunk of code to gain test coverage. Make sure your tests are making meaningful assertions.

## Control the Environment

It is important to control the environment in which your tests are running.

### Avoid Artificial Delays and use Waits

Use the [waits](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Platform/GlassPlatform/TestUtilities/Sources/Helpers/Waits.swift) helpers in TestUtilities to wait on specific conditions instead of adding arbitrary delays.

### Date/Time

Fix Calendar to `Gregorian` and fix the `TimeZone.` Use [`Clock`](https://gecgithub01.walmart.com/pages/walmart-ios/glass-app/walmartfoundation/documentation/walmartfoundation/clock-swift.protocol) to mock and inject `Date` values.

### App State

Tests shouldn't depend on a state of the app that can change outside of the test or setup. Tests should be clean slates, since we can't guarantee the order of execution.

### Localization

Ensure the localization cannot change or tests adapt to localization.

## Break up your Monolithic Tests

One useful technique in addressing flaky tests is to see if they can be broken up into more "unit" like tests. Take for example the following unit test. It is well written, logical, and covers all the UI for a given view in the app. 

```swift
func testHome_SignedInUser_MultipleActiveOrders() throws {
        app.launch()
        let tabBar = app.navigationBars["Shop"]
        XCTAssertTrue(tabBar.waitForExistence(), "Failed to find Shop")
        XCTAssertTrue(tabBar.isVisible, "Failed to tap on Shop")
 
        let collectionViews = app.collectionViews
        let ostHeaderLabel = collectionViews.staticTexts[.id(type: .OrderStatusTrackerHeaderLabel)]
        XCTAssertTrue(ostHeaderLabel.waitForExistence(),
                      "Failed to find OrderStatusTrackerHeaderLabel")
        XCTAssertTrue(ostHeaderLabel.isVisible,
                      "Failed to tap OrderStatusTrackerHeaderLabel in current visible view")
 
        let ostSubheaderLabel = collectionViews.staticTexts[.id(type: .OrderStatusTrackerSubheaderLabel)]
        XCTAssertTrue(ostSubheaderLabel.waitForExistence(),
                      "Failed to find OrderStatusTrackerSubheaderLabel")
        XCTAssertTrue(ostSubheaderLabel.isVisible,
                      "Failed to tap OrderStatusTrackerSubheaderLabel in current visible view")
 
        let ostCtaButton = collectionViews.buttons[.id(type: .OrderStatusTrackerCtaButton)]
        XCTAssertTrue(ostCtaButton.waitForExistence(),
                      "Failed to find OrderStatusTrackerCtaButton")
        XCTAssertTrue(ostCtaButton.isVisible,
                      "Failed to tap OrderStatusTrackerCtaButton in current visible view")
 
        let ostProgressView = collectionViews.otherElements[.id(type: .OrderStatusTrackerProgressView)]
        XCTAssertTrue(ostProgressView.waitForExistence(),
                      "Failed to find OrderStatusTrackerProgressView")
        XCTAssertTrue(ostProgressView.isVisible,
                      "Failed to tap OrderStatusTrackerProgressView in current visible view")
 
        let ostLinkButton = collectionViews.buttons[.id(type: .OrderStatusTrackerLinkButton)]
        XCTAssertTrue(ostLinkButton.waitForExistence(),
                      "Failed to find OrderStatusTrackerLinkButton")
        XCTAssertTrue(ostLinkButton.isVisible,
                      "Failed to tap OrderStatusTrackerLinkButton in current visible view")
}
```

While comprehensive, it does have some subtle issues that we can address.

1. If the test fails, it will not report the successful bits in the final results. We are looking at 5 elements here and if just 1 isn't reporting as we expect, the entire test fails.
2. We do not consider the underlying data model and if its properly configured before we start examining UI components.
3. No checks are made to ensure the data model is properly "hooked into" the UI via SwiftUI/Storyboard.

By decomposing the test into separate tests, we now have much finer grained control over identifying the flaky portion of this test in isolation. This also carries the extra bonus of now being able to explicitly express the decoupled nature of the elements in the test. Even if you have a test that does validate the coupled nature between 2+ elements, you still have the opportunity to pare down the test to just validate that set of dependencies.

```swift
func testHome_SignedInUser_MultipleActiveOrders_TabBar() throws {
        app.launch()
        let tabBar = app.navigationBars["Shop"]
        XCTAssertTrue(tabBar.waitForExistence(), "Failed to find Shop")
        XCTAssertTrue(tabBar.isVisible, "Failed to tap on Shop")
}
 
 
func testHome_SignedInUser_MultipleActiveOrders_HeaderLabel() throws {
        app.launch()
        let collectionViews = app.collectionViews
        let ostHeaderLabel = collectionViews.staticTexts[.id(type: .OrderStatusTrackerHeaderLabel)]
        XCTAssertTrue(ostHeaderLabel.waitForExistence(),
                      "Failed to find OrderStatusTrackerHeaderLabel")
        XCTAssertTrue(ostHeaderLabel.isVisible,
                      "Failed to tap OrderStatusTrackerHeaderLabel in current visible view")
}
 
 
func testHome_SignedInUser_MultipleActiveOrders_SubHeaderLabel() throws {
        app.launch()
        let collectionViews = app.collectionViews
        let ostSubheaderLabel = collectionViews.staticTexts[.id(type: .OrderStatusTrackerSubheaderLabel)]
        XCTAssertTrue(ostSubheaderLabel.waitForExistence(),
                      "Failed to find OrderStatusTrackerSubheaderLabel")
        XCTAssertTrue(ostSubheaderLabel.isVisible,
                      "Failed to tap OrderStatusTrackerSubheaderLabel in current visible view")
}
```

## No UI elements in XCTest based test classes

One foundational technique for writing stable unit tests is to stay away from ViewController or Visual object subclasses from driving test state in tests that derive from XCTest. Their behaviour in a non-UI Test context is ill defined and will vary across executional environments. Getting to a stable test state is best accomplished by the judicious use of view models and other subclassable state models that encapsulate external data being manipulated. If you have non-UI code that wants to alert the User in some visual manner under certain circumstances, that is ok. However, your testable code should not be making direct calls. Instead by providing a well defined protocol that can act as the intermediary to make the notification. This way, such a protocol can be easily implemented in the test class and provide its own internal state to track what alerts should be made. Your test can easily make assertions about that test. There is no need to test the behavior of native alert elements at all. The mobile OS provides has done that for you already. 

If you do have the need to test UI behavior, then please strongly consider migrating your test(s) to be based on XCUITest instead. From there, you can define a fully qualified (test) app and ensure that you have a legit context to trigger visual events.

## Do not make calls to external APIs

If you are writing XCTest based tests, make sure that you do not directly make calls to any external or System calls. Some primary examples are:

- Preferences: Believe it or not, while the UserDefaults object is thread safe, it is **NOT** process safe. If you are running parallel tests that are both attempting to modify preferences at the same time, you will trigger a kernel level exception in the Simulator. This is rare case and due to the random execution of tests, reproducibility will be tough.
- Opening URLs: If you have tests that necessarily cause side effects that open URLs via various schemes, they will trigger non-deterministic behavior as the Test itself relies on no expected state of the Simulator clone its run on.

The solution to both these issues is to create Mock interfaces for them. Allow the call to be made and return expected values.

## Defensive Programming

Code to ensure the app is in the right state. 
**Example:** A test launches the app and starts looking for UI elements. While the app.launch() is synchronous, there are no checks to interrogate the logic driving the setup of the view/module nor are we ensuring we are in the correct state in the visual lifecycle of the view. While the check for the elements visibility is a sound step to make here, we are not verifying any underlying data model that drives that label is correct. It could very well be the element there in the layout, buts its 'text' value is an empty string. This would lend some amount of credence to the flakiness in the tests.

On the laptop; more cores, better performance, faster test execution and timing keeps this from cropping up. On the Mini, many simulators running simultaneously, less cores, less performance can lead to flakey test results. Adding more defensive steps into the unit test ensured we walked through the UI in a known manner.

## Test the Data Models even for UI Tests

[Running CI Tasks Locally](https://gecgithub01.walmart.com/walmart-ios/glass-app#running-ci-tasks-locally):

- If devs need to recreate this issue locally, please **do not** just run the test case in question but run the whole test suite. Most of the issues occur when the entire test suite/plan is executed. Similarly for compilation errors, please **do not** just compile your feature module but compile the whole project (including platform). Best option to recreate the CI issues locally is to follow the below instructions:

**Build Compilation & Test:**

Make sure the Xcode command-line tools are set: *sudo xcode-select -s <path_to_xcode_app>*

Build & Test: *./BuildSupport/local-ci-flow.sh*

**Running Code Coverage Locally and Comparing the Results:**

*./BuildSupport/local-code-coverage.sh -lc just code coverage*<br />
*./BuildSupport/local-code-coverage.sh -ln comparing the coverage with remote master*<br />
*./BuildSupport/local-code-coverage.sh -lm comparing the coverage with local master*

## Using `@testable`

`@testable` should only be used to import the module that your test target is directly testing. For example, if your feature is Scanner, your test target should only use `@testable` to import Scanner, not other frameworks like WalmartPlatform.

Using `@testable` to import other external modules is dangerous because if the internals of these modules change, your tests break, and your tests should not depend on the internal implementation details of other modules.
