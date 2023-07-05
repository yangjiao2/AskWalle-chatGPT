# Shared UI Components

- Author: Angelo Di Paolo
- Decision: Option 3: FeatureUI

## Introduction

Determine a solution for sharing feature-specific UI components across features. Feature-specific UI is defined as UI that is not part of the Living Design 3 spec (LD3) and is not maintained by the platform team.

## Motivation

Features would greatly benefit from the ability to share UI components across plugin modules. This can be done today with Plugin APIs but this approach comes with limitations (see Option 1 below for more info on the limitations).

Several use cases have prompted this ADR:

- Homescreen and Search want to share Department List UI
- Payments Transaction and Payments Account modules want to share UI
- Post-tx team wants to share UI from Item Details and CXO

The ability to share and reuse UI components has several benefits:

- Reduces time to market by enabling teams to leverage components that have already been built by other teams
- Avoids code duplication and allows bugs to be fixed in one place

However, there are drawbacks to sharing UI components that must be considered:

- **Ownership can be unclear.** Is the owner the team that originally built the component? What if the original author is no longer using the component, who is responsible for support and maintenance?
- **Maintenance complexity.** If a team wants to modify a component for reuse, how do we prevent the team from breaking other use cases? How do we keep a component from being a maintenance nightmare as it's modified over time to fit every new feature's use case?

## Proposed Solutions

### Option 1: Plugin APIs

UI component implementations continue to live in plugin modules and are shared with other plugins by exposing plugin APIs that vend component instances. We're already doing this today but there are several limitations.

#### Pros

- Follows our guiding principle of "everything is a plugin API" which keeps plugins loosely coupled using protocols
- Code ownership is clear because every plugin has a code owner
- Teams can share components without any oversight from platform team

#### Cons

- Vending instances of components is limiting (can't subclass types)
- Significant overhead in maintaining protocols to abstract UI components

### Option 2: GlassUI

Feature components live in GlassUI, Glass's LD3 component and UI utility library. It's worth noting that we already have several non-LD3 components living in GlassUI today.

#### Pros

- We're already using GlassUI to share UI components
- Designated home for UI components, makes it easy to decide where code belongs

#### Cons

- Maintained and owned solely by platform team, platform can be a bottleneck for code reviews, releases, support
- Determining what is an LD3 component vs what is a shared feature component is less clear
- Implies that platform will provide support for any component added to the library

### Option 3: FeatureUI

Create a new shared library specifically for UI components that are shared across features but are not part of LD3 and are not owned by platform. This library would depend on GlassUI.

#### Pros

- Aligns with Android's approach to sharing feature UI components
- Enables teams to have ownership over components without platform team being a bottleneck

#### Cons

- Risk of becoming another shared dumping ground like the various shared libraries we used in GM/OneApp/Blue+
  - Mitigation: make platform team default code owners to gate keep changes
  - Mitigation: create separate frameworks for each feature
- Unclear how to decide if something belongs in GlassUI or FeatureUI (Mitigation: define clear guiding principles for where code belongs, make it clear what each repo is designated to contain)
- Ownership may not always be clear

### Option 4: Copy/paste

Teams copy and paste UI component implementations over to other plugins for reuse. See _[Redundancy vs dependencies: which is worse?](https://yosefk.com/blog/redundancy-vs-dependencies-which-is-worse.html)_ for an interesting point of view on why copying and pasting may be a good solution.

#### Pros

- Ownership is clearly defined, you copied it, you own it
- Feature's product and design teams can tailor the UI specifically for their use case avoiding the complexity of maintaining a component for every feature's use case

#### Cons

- Code duplication requires that bugs are fixed in multiple places


## Decision Outcome

Platform team decided to go with `Option 3: FeatureUI` along with creating separate shared UI frameworks for each feature team.

## Reference

- [Pull request documenting the discussion of this ADR](https://gecgithub01.walmart.com/walmart-ios/adrs/pull/1)
