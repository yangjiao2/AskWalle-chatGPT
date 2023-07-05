---
title: Apollo Framework
navTitle: Apollo Framework
---

# Apollo

- [About](#about)
- [How Are They Integrated?](#how-are-they-integrated)
- [Current Version](#current-version)
- [Updating SDK versions](#updating-sdk-versions)
- [Point of contact](#point-of-contact)
- [More Information](#more-information)

## About
Glass depends on [Apollo framework suite(includes Apollo, ApolloAPI, ApolloUtils, ApolloCodegenLib)](https://github.com/apollographql/apollo-ios/tree/0.53.0/Sources) to perform GQL operations. Glass iOS app uses [xcframeworks](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/apollo) for all of these dependencies, while GlassCLI uses [SPM package](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/Platform/GlassCLI/Package.swift#L24) for ApolloCodegenLib.

## How Are They Integrated?

- Apollo, ApolloAPI, ApolloUtils are pre-compiled into xcframeworks and deployed as static-artefacts to be consumed by the apps directly.
- These xcframeworks are fetched, unzipped and placed into `Carthage/Build` folders automatically when running `glass environment setup`.
- They are already embedded and linked to the main app target with the Carthage/Build folder set in the framework search paths.
 
## Current Version
Version [0.53.0](https://github.com/apollographql/apollo-ios/releases/tag/0.53.0) is used in development.

## Updating SDK versions

- Apollo, ApolloAPI, ApolloUtils are pre-compiled into xcframeworks and hosted onto [static-artefacts repo](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer/tree/main/repository/com/apollo). These are then deployed onto pangaea using this [CI Job](https://ci.walmart.com/job/ios-walmart-deploy-static-framework/).
 More info on how static-artefacts repo needs to be updated - [here](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer#readme)
- Once the deployment is successfully completed, update the `BinaryDependencySpecification` for  [Apollo](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/BinaryDependencySpecification/Apollo.json), [ApolloAPI](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/BinaryDependencySpecification/ApolloAPI.json) and [ApolloUtils](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/BinaryDependencySpecification/ApolloUtils.json) to point to the newly deployed Apollo frameworks.
- Update the [Cartfile.resolved](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/Cartfile.resolved#L3...L5) to resolve to the latest version of Apollo frameworks as specified in BinaryDependencySpecification
- Run `glass environment setup` to fetch these new version into `Carthage/Build`
- Compile the app to ensure there are no new warning/compile issues generated due to these new versions of Apollo.
- Create a PR with the changes in BinaryDependencySpecification and Cartfile.resolved


**Pre-compiling Apollo xcframeworks**:
- Platform team maintains a Apollo fork [here](https://gecgithub01.walmart.com/walmart-ios/glass-apollo).
- Clone the repo, switch to the `master` branch and pull the changes, if any.
- Fetch upstream changes and create frameworks using the terminal -
```
        git remote add upstream https://github.com/apollographql/apollo-ios.git
        git fetch upstream
        git checkout master
        git pull master
        git checkout -b "some_branch_name"
        git merge tag_name (ex: `git merge 0.53.0`) This should be the tag you want to create the xcframework for.
        Build the xcframeworks by running `./build.sh`
        Push the branch created and raise a PR against master
```
- Use these frameworks created to upload and deploy onto static-artefacts (detailed info above)


## Point of contact

- Primary contact - [@walmart-ios/glass-platform-ios](https://gecgithub01.walmart.com/orgs/walmart-ios/teams/glass-platform-ios/members)

## More Information

- Apollo, ApolloUtils and ApolloAPI are all dynamic frameworks and need to be embedded("Embed & Sign") within your executables only(like demo apps).
- Do not embed these in your static framework targets. It is already embedded in the main Walmart target.
