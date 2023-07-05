---
title: Testing - Primer
navTitle: Primer
---

# Testing Primer

[Testing Terms Glossary](https://engineering.walmart.com/docs/testing/glossary).

## Static Tests
These are the tests that check the code, independent of whether or not the code has been built. From the perspective of working on this project, these should generally stay out of the way and should require little maintenance.

Some examples of these are:

- SwiftLint
- SonarQube
- Compiler warnings/errors
- Project integrity checks

## Unit Tests

These should:

- be written with XCTest
- test the specs of a function/class/struct
- be deterministic
- break builds (locally, PR checks, scheduled builds)
- utilize test doubles

These should not:

- access the network
- access the file system / local databases (files written to disk, UserDefaults, Keychain) 

## Integration Tests

These should:

- be written with XCTest
- validate the portion of the code that interfaces to an external network sub-system while using a test double to represent that sub-system.
- be deterministic
- break builds (locally, PR checks, scheduled builds)
- utilize test doubles
- access networking via localhost/mocks

These should not:

- access the file system / local databases (files written to disk, UserDefaults, Keychain)

## Functional Tests

These should:

- be written with XCTest and XCUITest
- verify that all modules of a sub-module work together
- be deterministic
- break builds (locally, PR checks, scheduled builds)
- utilize test doubles
- access networking via localhost/mocks

These should not:

- access the file system / local databases (files written to disk, UserDefaults, Keychain)

## Visual Regression Tests

These should:

- be written primarily with XCTest (primarily XCUITest)
- verify that UI appears correctly to users
- be deterministic
- break builds (locally, PR checks, scheduled builds)
- utilize test doubles

These should not:

- access the network
- access the file system / local databases (files written to disk, UserDefaults, Keychain)

## Contract Tests

These should:

- be written with XCTest
- validate the test doubles used in other tests
- access the network
- access the file system / local databases (files written to disk, UserDefaults, Keychain)

These should not:

- use test doubles for validation

## E2E Tests

These should:

- validate the software system along with its integration with external interfaces
- access the network
- access the file system / local databases (files written to disk, UserDefaults, Keychain)

These should not:

- use test doubles
