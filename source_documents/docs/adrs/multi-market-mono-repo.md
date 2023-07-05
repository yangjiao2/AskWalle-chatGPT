# Multi Market Mono Repo

- Authors: [Alexandre Santos](https://gecgithub01.walmart.com/a0s028v) / [Mak](https://gecgithub01.walmart.com/m0s01p7)
- Decision: Option 1: New project - new target

## Introduction

A solution for hosting market specific (US, Canada, Mexico, etc.) iOS code in Glass mono repo.

## Motivation

The decision is to go mono repo and we plan to leverage as much as possible from the Glass frameworks such as GlassUI, WalmartPlatform and Plugins.
By going mono repo, code sharing between US and Canada is as simple as integrating code from a local folder.
We aspire for each market to work independently on stable framework code and have a smooth CI/CD pipeline.

## Considered Options

### Option 1: New project - new target

1. Create a new folder "Walmart.ca" inside root folder of the repo
1. Have Canada specific configurations, container specific code and Canada specific test cases inside that folder

#### Pros

- New Xcode Project has only Canada specific target such as Walmart.ca, Walmart.caTests and Walmart.caUITests
- Onboarding of new market is to replicate the new folder and make minimum refactoring in CI/CD

#### Cons

- Build scripts might need to be adjusted to use a different project file when executing tests/builds for the Walmart Canada app

### Option 2: Existing project - new target

1. Create a new Walmart.ca target app in the existing Walmart (US) project
1. As best practice, we can have Canada specific configurations, container specific code and test cases inside a new folder "Walmart.ca"

#### Pros

- Reusing an existing project
- More visibility of other markets to teams working on plugins
- Canada Market is just another target App in the same developing environment


#### Cons

- Requires engineers to pay attention on what files they are adding to each target as some files might be shared and others be unique to each market app
- Requires very high level of collaboration between market specific team and glass team
- Not sure if we are ready for this at this initial stage


## Recommended Solution

We recommend the "Option 1: New project - new target", since this would be safe to start during early phase of collaboration.

## Future Enhancement

As part of supporting other consuming apps, like the Health & Wellness app for example, we would need to expose some of the plugins as artifacts, by using something like Swift Package Manager to allow those apps to install the Glass packages.

By doing the above steps, we would be making it easier for the plugins to be consumed as isolated dependencies, as it would be clear in their public interface, what are their dependencies and required configurations.

### Reference Links

- [Apple SPM](https://developer.apple.com/documentation/swift_packages)
- [SPM Dependency](https://confluence.walmart.com/display/CAMAPP/SPM+Dependency+Management)
