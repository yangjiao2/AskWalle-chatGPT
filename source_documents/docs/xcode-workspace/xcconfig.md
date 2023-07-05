---
title: Xcconfig
navTitle: Xcconfig
---

# Xcconfig

The Glass project (Walmart.xcworkspace) utilizes xcconfig files to hierarchically manage Xcode build settings across all of the market apps, sample apps, plugin frameworks, test bundles, etc. Xcconfig files are preferable over embedding build settings in an Xcode project file because they are easier to maintain with source control and allow inheritance.

## Resources

To learn more about xcconfig files, check out the following articles and WWDC videos.

- [Apple Build Settings Reference](https://developer.apple.com/library/archive/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/1-Build_Setting_Reference/build_setting_ref.html)
- [Explore advanced project configuration in Xcode](https://developer.apple.com/videos/play/wwdc2021/10210/)
- [Xcode Build Configuration Files](https://nshipster.com/xcconfig/)

## Hierarchy

The xcconfig files common to the Glass workspace are located in the `Configuration/` folder at the root of the repository. The xcconfig file at the root of the xcconfig tree hierarchy is `Platform.xcconfig`; all xcconfig files inherit from this file.

TODO: Add diagram and more docs.

## Build setting overrides

There are a number of build settings that can be overridden to tailor a build based on its particular context. For example, a developer creating a local build may prefer generating GraphQL code through the command line rather than a build phase and a CI build may require GraphQL code generation to be disabled entirely. Both of these requirements are accomplishable by specifying build setting overrides in an xcconfg file that is optionally included (`#include?`) in `Platform.xcconfig`. The specific Glass overrides are listed below and can be included in either `Glass-Project-Developer.xcconfig` or `CI.xcconfig` depending on the use case.

### Available build setting overrides

| Build Setting                               | Override                                             |                                                                                                      | Values                 | Default Value                                      | CI Value           |
|---------------------------------------------|------------------------------------------------------|------------------------------------------------------------------------------------------------------|------------------------|----------------------------------------------------|--------------------|
| APOLLO_CODE_GENERATION_ENABLED              | APOLLO_CODE_GENERATION_ENABLED_OVERRIDE              | Determines whether GraphQL code generation is enabled in a GQL target's build phase.                 | NO, YES                | YES                                                | NO                 |
| GLASS_DEBUG_INFORMATION_FORMAT              | GLASS_DEBUG_INFORMATION_FORMAT_OVERRIDE              | Determines the DEBUG_INFORMATION_FORMAT value for a build.                                           | dwarf, dwarf-with-dsym | Debug: dwarf <br /> Profile/Release: dwarf-with-dsym | dwarf-with-dsym    |
| GLASS_TREAT_WARNINGS_AS_ERRORS              | GLASS_TREAT_WARNINGS_AS_ERRORS_OVERRIDE              | Determines whether Swift warnings are treated as errors.                                             | NO, YES                | NO                                                 | Debug: YES         |
| GLASS_WARN_COMPILE_TIME_ENABLED             | GLASS_WARN_COMPILE_TIME_ENABLED_OVERRIDE             | Determines if warnings about long type-checking of expressions and function bodies are enabled.      | NO, YES                | NO                                                 | YES                |
| GLASS_WARN_LONG_EXPRESSION_TYPE_CHECKING_MS | GLASS_WARN_LONG_EXPRESSION_TYPE_CHECKING_MS_OVERRIDE | The threshold in ms to warn about long type checking (-warn-long-expression-type-checking).          | milliseconds           | 125                                                | 5000               |
| GLASS_WARN_LONG_FUNCTION_BODIES_MS          | GLASS_WARN_LONG_FUNCTION_BODIES_MS_OVERRIDE          | The threshold in ms to warn about long type checking of function bodies (-warn-long-function-bodies) | milliseconds           | 125                                                | 5000               |
| SWIFTLINT_ENABLED                           | SWIFTLINT_ENABLED_OVERRIDE                           | Determines whether SwiftLint is run when building.                                                   | 0, 1                   | 1                                                  | 0 (run separately) |
| SWIFTLINT_CHANGES                           | SWIFTLINT_CHANGES_OVERRIDE                           | Determines whether only the modified files (as determined by GIT) are linted.                        | 0, 1                   | 1                                                  |                    |
| SWIFTLINT_MODE                              | SWIFTLINT_MODE_OVERRIDE                              | Determines the linter mode for swiftlint.                                                            | lenient, strict        | lenient                                            |                    |
| SWIFTLINT_INCLUDE_UPCOMING_RULES            | SWIFTLINT_INCLUDE_UPCOMING_RULES_OVERRIDE            | Determines if SwiftLint is also run with rules that aren't enforced yet, but that are planned to be. | 0, 1                   | 0                                                  |                    |

### Glass-Project-Developer.xcconfig

The `Glass-Project-Developer.xcconfig` is intended for local developer overrides. For example, if a developer does not want Swiftlint to run during their local builds, they can create their own Glass-Project-Developer.xcconfig file in `Configuration/`. The file is ignored by GIT to prevent accidental check-ins of the file.

#### Tip

You may have multiple clones or work-trees of `glass-app` and wish to share an xcconfig file across them. To accomplish this, you can create a symbolic link in `Configuration/` to a shared `Glass-Project-Developer.xcconfig` file located somewhere in your file system.

#### Example

Precondition: You are in the `Configuration/` directory when executing the following command and your shared Glass-Project-Developer.xcconfig is located in parent directory of the repo root.

##### Directory structure

```text
development/
├─ Glass-Project-Developer.xcconfig
├─ glass-app/
│  ├─ Configuration/
│  │  ├─ Glass-Project-Developer.xcconfig (symlink)
```

##### symlink command

```sh
ln -s ../../Glass-Project-Developer.xcconfig .
```

### CI.xcconfig

The `CI.xcconfig` file is used for specifying build settings for **all builds** that occur in CI (looper). The file is moved from `BuildSupport/` to `Configuration/` before a build is started.
