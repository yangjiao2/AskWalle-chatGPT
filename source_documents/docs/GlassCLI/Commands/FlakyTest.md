# **Glass FlakyTest Command**

`glass flaky-test` is a set of commands for detecting, disabling, and re-enabling test cases that fail.

##  Motivation

Glass developers often spend a great deal of time dealing with failing tests. A test that is unrelated to your changes may fail and prevent you from merging your work. When a test fails, you spend time tracking down the team that owns the test, alerting the team, and keeping up to date with progress on the resolution. This adds an abdundance of overhead to a developer's workflow, impedes progress, and reduces focus time.

## Overview

Tests are run on the development branch every night. If a test fails, it is considered to be flaky and is automatically disabled. After the test is disabled, the team owning the test is alerted via slack in [#glass-ios-test-failures](https://walmart.slack.com/archives/C05BZS3QMSR) and is given a two-week deadline to resolve the failure. The team owning the test will continue to be warned in every PR they open until the test is resolved. The team owning the test will be blocked from merging PRs if the deadline has passed and the test has not been resolved.

## Disabling Tests

`glass flaky-test disable` find all failing tests contained in the xcresult bundles and updates test plans to disable them.

1. Finds all failing tests from all xcresult bundles at a given path.
2. Updates the associated test plans to skip the failing tests.
3. Adds each new failing test to the failing test registry (`FailingTestCases.json` file). A new failing test is a test that has not already been added to the failing test registry.
4. Alerts teams of new failing tests in [#glass-ios-test-failures](https://walmart.slack.com/archives/C05BZS3QMSR) along with a deadline for when PRs will be blocked if the failure has not be resolved.

## Resolving Tests

Run `glass flaky-test resolve` to mark the test as resolved. The command removes the test from the failing test registry and re-enables the test in the test plan. You will need to commit changes to both the failing test registry file (`FailingTestCases.json`) and the test plan.

```
glass flaky-test resolve -n test_my_test_case_name
```