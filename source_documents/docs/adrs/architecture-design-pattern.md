# Architecture Design Pattern

- Authors: Angelo Di Paolo, Tim Sears, Jordan Perry, Alex Johnson 
- Decision: Option 1: MVVM+Coordinator

# Context

We need to evaluate options for an architectural design pattern (MVC, MVVM, VIPER) and agree on an approach that we can use as a recommendation for teams to use.

We need to achieve very high test coverage (75%) in every feature from the start with Project Glass. Some teams are better at architecting their feature for testability than other teams so we want to recommend a pattern along with documentation and guidelines (and maybe tooling) that makes every team well-equipped to achieve high test coverage from the start.

# Goal

Provide teams with a recommended architecture pattern that enables every team to write correct code.

There are several factors that contribute to correct code:

- Choice of language / use of language features that make incorrect code impossible. Examples include strong typing, and using "enumerations" instead of raw numbers/strings to represent limited sets of options.
- Factoring of code to make responsibilities clear and avoid "spaghetti" code.
- Unit, functional, integration, etc. tests. While test coverage alone does not prove that code is correct, lack of test coverage does imply that its correctness is unproven.

# Adoption

The group has aligned on an adoption philosophy that will provide consistent patterns, documentation and example code to make onboarding to the application architecture as easy as possible.

We have also aligned that we will work with each feature team individually as they onboard with Glass to ensure that each engineer has a comprehensive understanding of the overall architecture.

Following that, if a feature team wishes to pursue a different architectural decision, they may do so provided the present their proposed architecture to the core team along with their plan to achieve at least 75% test coverage.

# Deliverable

An architecture pattern recommendation due 4/17/2020.

# Design Principles

- No business logic in view layers
- Clear separation of concerns
- High testability 
- Extensibility

# Evaluation Use Cases

- List
- Form
- Detail

# Options

## Option 1: MVVM+Coordinator

The team's current recommendation is to move forward with MVVM+Coordinator+Combine.

The Coordinator pattern and View Controller Composition patterns can be used to supplement MVVM + Combine as they are not mutually exclusive.

[Example Project with 90%+ Test Coverage](https://gecgithub01.walmart.com/j0p015t/mvvm_coordinator_combine)


### Pros

- Consistent Data Flow
- Combine works well for state that changes that should propagate to the view layer.
- The advantage to this is it's a single, consistent mechanism for updating data flow and state.
- It's also a guard rail that abstracts this problem away from the typical engineer.
- Aligns with Apple's roadmap, helps Engineers with career growth, sets the stage for us to adopt Swift UI in the future.

### Cons

- This approach does require iOS 13+. When is the soonest we think Glass could replace the Blue app could happen?
  - Mitigation strategy: We would need to explore another option to solve for bindings. Either roll our own or evaluate a third party? RXSwift? OpenCombine? Bond?
  - Mitigation costs:
    - 1) Maintenance; If we roll our own, this is the highest and we're stuck with it for a very long time (see `SD` classes that were deprecated 2+ years ago in GM). If we go with a 3rd party solution, we will be stuck with that for the long-term (see `MBProgressHUD`) 
    - 2) Onboarding; This will become more prevalent in the future, but using a first party solution like combine isn't going to box us in to situations where we are looking for people with experience using the 3rd party solution. We could have no expectation of previous knowledge if we build our own solution. Combine is what Apple is banking on (see SwiftUI).
    - 3) Bugs; While it is inconceivable to think that Combine will be free from bugs, the bugs that are encountered will also impact a growing number of apps in the store. So those bugs will likely be low cost to customer or within that customer's expectations (OS bugs the user grows accustomed to). Using a 3rd party solution or homegrown doesn't provide as strong of foundation.
  - Ideally, having a replacement implementation with identical call sites could make migration over to Combine at a later point easier, if necessary.
- Developer onboarding learning curve - Some mixed opinions. Potentially we could get there with some good examples. The call sites are pretty simple.
  - We may need to develop some helper methods / extensions to help with the abstraction.
  - This learning curve could be mitigated in the long run because it's going to help naturally onboard engineers to help with their growth and development in the future to help prepare them to adopt Swift UI, etc.


## Option 2: VIPER

Some thoughts on VIPER from the perspective of an iOS Engineer.

We adopted a custom variant of VIPER on Nike Training Club when I was there 2 1/2 years ago. I was the tech lead on that project for about 8 months. I did not instigate the adoption of the pattern, but rather instead took over the role from another engineer and helped continue our best practices of pattern adoption.

### Pros

- View Controllers became lovably small in their code footprint.
- Standardized option meant consistent implement features. This was less a byproduct of VIPER itself and was more that we standardized on something.
- It worked well with Nike's version of BaseView. (smile)

### Cons

- We still did not unit test view controllers or views on NTC. (It doesn't mean you can't, though)
- All of the View Controller bloat pretty much got deferred to the Interactor
- Presentation flow became convoluted and complex and hacks were necessary to get it to work, but that may have been because the most important flow of the app involved replacing existing view controllers via custom UIViewController transitions on top of a modal stack, which is a pretty unusual use case.
- If the core app architecture and navigation is implemented using RIB / VIPER / something similar, it might actually be difficult for feature teams to deviate from it and design their own architecture. I personally like having a standard architecture pattern throughout the app, but I recognize not everyone agrees with me on that.

## Option 3: RIBS

🍖😋 https://github.com/uber/RIBs


### Pros

- Highly decoupled pieces. All communication goes through protocols.
- 1st-class dependency injection support.
- Designed to work on both Android and iOS.
- Support for "viewless" RIBs.
- Clear places for business logic, navigation logic, layout logic, etc.
- Highly testable.

### Cons

- Code is splattered all over the place. In the tutorial, a simple RIB representing "logged out" state comprises 11 classes and protocols. More if you want to inject any dependencies.
- Unintuitive naming. RIBs uses suffixes like "controllable" and "listener" for protocols that are basically delegates.
- Opinion: Sacrifices clarity for testability. The heavy use of protocols allows for both dependencies (e.g. game state) and navigation (e.g. start game) to be mocked. But I worry that this leads to poor quality tests that are so heavily mocked as to be basically just testing the specific implementation, not the actual observable behavior. This is mostly a gut feeling at this point.
- Soft requirement of additional tooling (e.g. code gen). You can forego this requirement, but then you need to stub out all of those classes and protocols by hand.
- Heavily driven by framework-level code, which could make debugging difficult.
- Because it's driven by a framework you don't get to model state using structural types (structs and enums), and instead have to rely on your code (or the framework code) to avoid entering "invalid" states.
- Recommended pattern relies on "RxSwift" / "RxCocoa", which are not bad in and of themselves, but it is a pretty big additional dependency to use and understand.
- Tutorial doesn't work OOB (need to change RxCocoa version in the Podfile)
- Tutorial is a hot mess of modally-presented VCs

### Related, but somewhat off-topic, ideas that would need to explored more fully before being useful

- I like this idea of viewless-RIBs. I typically like to structure apps with nested view controllers. The downside to that is that every VC creates a rendering layer, even if it doesn't actually draw anything. But... UIStackView is kind of magical; it doesn't create a layer. Could we simulate a "viewless" View Controller by setting its view to a stack view (but then not adding any arranged subviews)?

### Summary

Our evaluation consisted of a phone call with some engineers from Uber, reading through the documentation, and going through the official tutorial.

While there are some promising aspects to RIBs, it has many of the same drawbacks and risks that steered us away from React Native:

- It's a framework, not (just) an architectural pattern, so you sort of have to "buy in" to it completely, or not at all.
- It is an intermediary between our code and the underlying platform toolkit (UIKit), which could affect our ability to build the experiences we want.
- We may need to wait for updates to the framework in order to take advantage of new platform features.
- It's primarily maintained by a third party. If Uber were to stop supporting RIBs, we would need to take up the maintenance ourselves, rewrite with a new technology, or live with existing bugs/limitations.

Additionally, there were some other more technical aspects that we felt were contrary to the goals set above. 


## Other added benefits of iOS 13

- We will avoid iOS 13 migration efforts early next year.
- DiffableDataSource (https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource)
  - Clean state management of a table view
  - Row insertions/deletions occur automatically based on snapshotting
- Build with ground up supporting new modal presentation style for consistent user experience across their device (let our UX get out of the way)
- Bridge to SwiftUI



### Testimonial of the expected value of `UICollectionViewCompositionalLayout`

Having spent the past year engineering on Homescreen, I'm quite familiar with the challenges that come with trying to put complex UI within table view cells. Most carousels are composed of collection views imbedded it table view cells. Customer Engagement/Customer Connection have to swap between putting all the tiles into a stack view in a single cell for horizontally `.regular` layouts, to providing tiles in individual cells for horizontally `.compact` ones. Not only are these approaches messy and time consuming to write, they have a significant impact on Accessibility. One example of this is that the awkward situation in CEM/CCM is actually a work around required because the simpler implementation of always using a stack view and just swapping the axis broke voice over. Another example is that the standard focus tracking provided by voiceover simply falls over for our current UI, and so we're forced reimplement tracking ourselves.

iOS 13's `UICollectionViewCompositionalLayout` however is custom built to address the challenges of this type of UI. They literally state this in the WWDC video, when they discuss that it was built to solve these exact same challenges in the home page of the App Store.

### Testimonial as to the value of a "Diffing" approach to table/collection view data

On a project at a former employer, we initially used a Diffing strategy similar to the one provided by `UICollectionViewDiffableDataSource` to power an infinite list that could expand in either direction. By combining it with unidirectional data flow (similar in affect to what's shown in the prototype app), we saw significant reduction in complexity

The real benefit however came when we started building out the ability to filter and search within the list. Because the system was already designed to simply calculate the differences in the new and old data sets and then apply them, adding this feature required no changes to how we interacted with the `UICollectionView`. In fact, the only change to the `UIViewController` that housed the collection view was to add the UI for the filter/search bar. All of the filtering logic was handled at the Model/ViewModel, and improvements to the UX we're all handled by the `UICollectionViewLayout`. The code that handled updating the collection view with new data did not change at all.

Further more, `DiffableDataSource` is also a much easier to use implementation of the pattern. While there are existing 3rd party libraries that can calculate diffs, these still left us with a couple of non-trivial responsibilities where mistakes caused runtime crashes:

- updating the data source at the correct time during the `performBatchUpdates` process. The timing is a small detail, but it's easy to get wrong (even some helpers in diffing libraries have issues here).
- avoiding update collisions. This is particularly important if you're animating the changes, because if you receive new data during an existing change animation, you have to ensure the existing animation finishes before you calculate a new diff and start the next update. You also have to ensure that you correctly time updating the data that's powering your `DataSource` implementation.

`DiffableDataSource` however handles these issues by controlling the update process through it's `apply` API. Given the number Walmart features powered by table/collection views, this is a huge reduction in particularly error prone boilerplate over the diffing solutions available before iOS 13.

### Technical concerns with using a different Reactive framework instead of Combine

While there are Reactive "Binding" libraries other than `Combine`, trying to use them interchangeably would be much more difficult than simply trying to map the APIs. Here, for example, are some behaviors that tend to vary from library to library but where those differences could easily cause behavior to change in an attempt to move from one library to another:

- What effects a `Publisher` does on creation before it has a `Subscriber`, vs after it's first `Subscriber` (aka Hot vs Cold)
- What happens when a second `Subscriber` attaches to an existing `Publisher` that has already sent values (aka Sharing, multicast, buffering)
- How the system behaves when the `Publisher` wants to send data faster than the `Subscriber` can handle process it (aka backpressure)
- Memory management

## Decision Outcome

The team decided to go with `Option 1: MVVM+Coordinator`.

# Reference

- [Original architecture pattern evaluation doc](https://confluence.walmart.com/display/GPONEW/iOS+Architecture+Pattern+Evaluation)
- [RIBs](https://github.com/uber/RIBs)
