# UI Construction Guidelines

- Authors: Angelo Di Paolo, Jordan Perry, Spencer Miles
- Decision: Option 2: Enforce all views to be constructed programatically

# Context

We need to establish tooling guidelines for constructing UIs in Project Glass.

# Goals

- Support common colors, spacing constants, and font references in all views (GlassUI theming)
- Enable Glass team members to easily jump between features
- Ability to get high test coverage with strong support for dependency injection

# Options

## Option 1: Allow a Hybrid approach, supporting constructing view programmatically and with Interface Builder (Storyboards and Xibs)

### Pros

- Enables engineers to use the tools that they are most comfortable with using

### Cons

- Lack of consistent approach adds friction when engineers jump between glass features
- IB-constructed views are difficult to code review in pull requests
- IB-constructed views are difficult to unit test. A view or view controller that needs to be instantiated from a xib will have more difficulty dependency injecting into the `required init(coder: )` call
- Unable to leverage common view construction patterns with IB-constructed views
- Difficult to globally refactor class names and other code references
- Merge conflicts are difficult to resolve
- No compile time safety (Recent example: https://gecgithub01.walmart.com/walmartlabs-wmusiphone/ios-walmart/pull/15418/files)

### Open Questions

- Can we support theming common colors, spacing constants, and font references in IB-based views?

## Option 2: Enforce all views to be constructed programatically.

## Pros

- Strong support for theming common colors, spacing constants, and font references in all views
- Enables us to have a consistent approach that reduces friction when engineers jump between glass features
- Ease of PR review
- Ease of unit testing

## Cons

- Engineers who prefer to use IB are forced to build views programmatically

# Decision Outcome

The team decided to go with `Option 2: Enforce all views to be constructed programmatically`.
