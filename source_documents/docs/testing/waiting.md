---
title: Testing - Waiting in Tests
navTitle: Waiting in Tests
---

# Waiting in Tests

- [Introduction](#introduction)
- [Unit and Integration Tests](#unit-and-integration-tests)
	- [Waiting for Publishers](#waiting-for-publishers)
	- [Waiting for UIKit](#waiting-for-uikit)
	- [Waiting for Conditions](#waiting-for-conditions)
	- [Waiting for Other Actions](#waiting-for-other-actions)
- [Functional UI Tests](#functional-ui-tests)
	- [General](#general)
- [Other Considerations](#other-considerations)

## Introduction

[Glass Platform](https://gecgithub01.walmart.com/walmart-ios/glass-platform) provides helpers for dealing with waiting for certain actions to complete.

These can be found [here](https://gecgithub01.walmart.com/walmart-ios/glass-platform/blob/development/TestUtilities/TestUtilities/Helpers/Waits.swift).

## Unit and Integration Tests

### Waiting for Publishers

[Combine](https://developer.apple.com/documentation/combine) introduces the concept of a [Publisher](https://developer.apple.com/documentation/combine/publisher), which is a type that can transmit a sequence of values over time.

To test a publisher, a test needs to be prepared for handling a transmitted value immediately or later. If a test were to incorrectly wait (or not wait at all), it could result in the test failing unexpectedly.

Example usage:

Given a publisher of type `PassthroughSubject<Int, Error>`, you would want to write tests that both wait on value(s) or an error.

```
func testValueReceived() throws {
    ...
    
    let value = try waitForValue(publisher)
    XCTAssertEqual(value, ...)
}

func testValuesReceived() throws {
    ...

    let values = try waitForValues(publisher, expectedCount: 2)
    XCTAssertEqual(values, ...)
}

func testFailureOccurred() throws {
    ...

    let error = try waitForFailure(publisher)
    XCTAssertEqual(error, ...)
}

func testFinishes() throws {
    ...

    try waitForFinish(publisher)
}
```

### Waiting for UIKit

Given a view controller:

```
func testViewControllerAppeared() throws {
    ...

    try waitForViewControllerToAppear(viewController: viewController)
}

func testViewControllerPresented() throws {
    ...

    try waitForViewControllerToPresent(presenter: presentingViewController)
}

func testViewControllerDismissed() throws {
    ...

    try waitForPresentedViewControllerToDismiss(presenter: presentingViewController)
}

func testAlertPresented() throws {
    ...

    try waitForAlertToBePresented(on: presentingViewController)
}

func testAlertPresentedByTitle() throws {
    ...

    try waitForAlertToBePresented(on: presentingViewController, withTitle: "Error")
}

func testNavigationPush() throws {
    ...

    try waitForNavigationControllerToPush(navigationController: navigationController, UIViewController.self)
}
```

Given a view:

```
func testViewUnhidden() throws {
    ...

    try waitForViewToBeUnhidden(view: view)
}

func testImageSet() throws {
    ...

    try waitForImageToBeSet(on: imageView)
}
```

### Waiting for Conditions

There are a few conveniences for dealing with conditions. All conditions have a default timeout (either `.defaultWait` or `.defaultNonWait`).

1. Closure based
2. Autoclosure based
3. Non-condition; This is for determining that a condition did not occur. This has a default timeout of `.defaultNonWait`.

### Waiting for Other Actions

If an existing `Wait` does not satisfy your need, there are stil options for dealing with waiting

1. Make a new wait and contribute to `TestUtilities`
2. Use `XCTestExpectation`. Extensions exist for this to where you don't need to specify your timeout and will utilize `.defaultWait`.

## Functional UI Tests

### General

We rely on the native waiting functionality built in to `XCUIElement`. To help with dealing with the time that one should wait for existence/absence, there are extensions that automatically specify the timeout to be `.defaultWait`.

## Other Considerations

- [Waits.swift](https://gecgithub01.walmart.com/walmart-ios/glass-platform/blob/development/TestUtilities/TestUtilities/Helpers/Waits.swift)
- [XCUIApplication+Extensions.swift](https://gecgithub01.walmart.com/walmart-ios/glass-platform/blob/development/TestUtilities/TestUtilities/Helpers/XCUIApplication%2BExtensions.swift)
