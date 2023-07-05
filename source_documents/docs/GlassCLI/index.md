---
title: Glass CLI
navTitle: Glass CLI
---

# **Glass CLI**

## **Overview**
GlassCLI is the a command line executable, built in Swift, where all build and developer productivity tooling lives in the Glass ecosystem (or will live in the near future :wink: ).

## **Installation**
Installing GlassCLI is as simple as running `./scripts/install-glass-cli.swift` from the root of the repo. That said there are more details that everyone should be aware. First and most importantly, GlassCLI lives in the mono repo and therefore the required version of GlassCLI is tied to a specific commit. It is recommended to run the install script anytime you pull or rebase from development or check out to an old branch. In fact, if you don't, you will likely run into errors throughout the glass ecosystem.

Later versions of GlassCLI also have an automatic update interceptor that will attempt to install the correct version if a version mismatch is detected while running any other command. 

### How is it installed? 
Because GlassCLI exists as a Swift package, all we do is run `swift build --configuration release` from its root directory. Once it's done compiling, we copy it into your systems brew bin, `/usr/local` for Intel machines and `/opt/homebrew` for M1 machines.

## **What is the GlassConf system?**
Version 1.0.0 of GlassCLI introduced the concept of Glass configuration files. In short, a `.glassconf` file can be used to describe metadata associated with a discrete glass capability or plugin. In it's simplest form this metadata contains information about the plugin's code owners (including slack handle, github team, and team name) and the plugins location in the repo. A .glassconf file can be further expanded with optional capabilities to unlock powerful build tooling like localization support. 

Creating a `.glassconf` file for your plugin is as simple as running `glass configuration init --destination Relative/Path/To/MyPlugin --plugin MyPlugin`. This will create a `MyPlugin.glassconf` at `Relative/Path/To/MyPlugin/MyPlugin.glassconf`. After that all you need to do is fill out all the empty sections in the file. 

For a more detailed description of GlassConf and what functionality it can unlock for your team, see [GlassConfiguraton](./Commands/GlassConfiguration.md)

## **Top Level Commands**
The following commands represent the top level commands in GlassCli. Some have various subcommands that are detailed in each commands associated document below.
- [`glass configuration`](./Commands/GlassConfiguration.md): Provides commands for creating and interacting with .glassconf files.
- [`glass environment`](./Commands/Environment.md): Provides commands for interacting with the development environment in the glass repo. 
- [`glass generate`](./Commands/Generate.md): Provides commands for generating plugins in the glass repo.
- [`glass apollo`](./Commands/Apollo.md): Provides commands for registering persisted queries.
- [`glass strings`](./Commands/Strings.md): Provides commands and tooling for localized strings in the glass ecosystem.
- [`glass edit`](./Commands/Edit.md): Convenience command to easily open the SPM package that houses GlassCLI for editing. 
- [`glass gen-zsh-completions`](./Commands/GenZshCompletions.md): Convenience command to generate and write a zsh completions file for GlassCLI
- [`glass verify-version`](./Commands/VerifyVersion.md): Command that validates the correct branch

## **Command Interceptors**
GlassCLI has several command interceptors that greatly increase the power of the tool. See the docs below for more information:
- [Overview](./Interceptors/index.md)
- [Automatic Upadte](./Interceptors/automaticUpdate.md)
- [Glass Configuration Loader](./Interceptors/configurationLoader.md)

## **FAQ**

#### *Why do I see `zsh: command not found: glass` when committing or trying to run a glass cli command?*
GlassCLI is used in several pre commit and pre push hooks and is not installed. You need to run the [installation script](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/scripts/install-glass-cli.swift): `./scripts/install-glass-cli.swift`.

#### *Why did my command file after GlassCLI said an update was required?*
GlassCLI's automatic update detected you were on an older version than what was required by you currently checked out a branch. Thus, the automatic update command interceptor kicked in and updated the tool for you. Just rerun the failed command.
