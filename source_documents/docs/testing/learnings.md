---
title: Testing - Learnings
navTitle: Learnings
---

# Testing Learnings

## Combating Fragility

### Taking Advantage of Container

Leveraging `Container` for the non-test code has the following advantages:

- Well defined dependency management
- Implicit dependency injection

But requires the following considerations in your tests:

- Knowing what dependencies are required for the particular piece of code under test
- Avoidance of sharing a `Container` between tests

### Mocking in XCUITest

The Glass iOS app initially relied on localhost mocking to mock network in UI Tests. We used a library called `Peasy`. 

Due to all of the `JAMF` products installed on all of our machines (including our looper agents), this proved to be too unreliable and fragile.

This required a technique where we initialize a `MockServer` (which is `Codable`) and we pass it as a launch environment to be decoded in app under test.

This technique allows us to achieve a higher level of determinism than we were able to achieve with `Peasy`.

### CI/CD Pipeline and Keeping things Moving

Our CI/CD pipeline runs all _necessary_ tests when a build is started. This means that for nightly builds, all of the tests will be run. On Pull Requests, only the tests associated with the changes will be run.

To accommodate running the tests while still receiving build feedback quickly, we parallelize the test plans on the available agents. 
