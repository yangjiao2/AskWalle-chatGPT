---
title: TempoUI
navTitle: TempoUI
---

# TempoUI

## Overview

`TempoUI` is a plugin framework that is part of the `Walmart` project. It offers a turnkey solution 
to more quickly develop an entire Tempo driven page and a way to share and reuse existing Tempo views and models.   

 ## Process Notes

 - Engage with Platform team early on to discuss best approach to your given feature. Ideally before Tempo seeds are defined.
 - Do a PR that just integrates TempoUI into your plugin. Technically, you probably won’t even need an approval from Platform team as the changes will be to your plugin only. But it might be a good idea to ask in `#glass-ios-tempo` Slack channel if somebody can take a look. As a result of this work, you should be able to fetch your page from Tempo and show something on screen. For example, add **HeadingBanner** to your page in Tempo and you should see it on screen.
 - If your feature requires creation of a new Tempo module(s) - submit a separate PR for each of the modules you add. This way the scope is smaller and it’s easier to review. Consider smaller individual PRs for the GraphQL, data model and the UI portions of the work. Adding tests along the way.
 - When considering developing a new type of module where possible, try to reuse existing components instead of re-inventing the wheel. By re-using existing components we are not only saving development/design/testing/maintenance time and company's money, but we also ensure that users have consistent user experience through the app.
If you are adding a new Tempo module, please try to make it reusable by others too. E.g. choose names that are not specific to your feature or team and try to avoid hard coded strings on the client. Be ready to answer questions about why you are not using one of the existing Tempo modules or not modifying an existing Tempo module to support new use case.
 - Ensure minimum of 75% code coverage for code being added.
 - Since this is a shared module, expect a rigorous code review. Account for several days for PR review and feedback incorporation. 
 - Include clear details in your PR with links to relevant Tempo seeds, other PRs, UX design/specifications etc.
 - This framework is evolving rapidly. Expect conflicts and keep your branch in sync with development. 
Use rebase (not merge) and incremental commits. 
Don't commit then squash after a review. It makes it more difficult for the reviewer to easily see how the feedback was addressed.

## Shared GraphQL

`TempoUIGQL` contains query fragments for many types of Tempo modules, located in `Plugins/TempoUI/TempoUIGQL/Sources/GraphQL/Queries/Fragments/`.
Clients can and should reuse these shared fragments to define their query. Clients can also use shared `Modules.graphql` query associated with  `ModulesRequest<ModulesQuery>`.

If updating GraphQL schema, then, as with any other feature, only the *production* schema should be merged into `development`.

## TempoAPI Integration

See [TempoAPI.swift](/Platform/PlatformAPIs/PlatformAPIs/PlatformAPIs/APIs/TempoAPI/TempoAPI.swift) for how you can fetch a page layout from Tempo backend.
This is relevant in the context of making a `SectionsDataSource` for your page.
`ModulesDataSource` is available as a convenient method of fetching using `TempoAPI`.

## TempoUIAPI

You can use `TempoUIAPI.makeStateCoordinator` and other plugin APIs to show the content that your page has.

Under the covers, `TempoUI` uses a `UICollectionView` to show the content. Each Tempo module has a corresponding `SectionController` that is responsible for providing all the details needed to show it, handle actions that user took, and analytics. Each returned Tempo module will be a separate collection view section. Some Tempo modules may have a single collection view item, while others may have multiple items.

## Integration steps for adding a new module type to `TempoUI`
There are two ways to add a module to `TempoUI`.

1. Adding directly into `TempoUI`.
2. Adding to feature plugin and conforming plugin to `TempoUIModuleProvider`.

If a team is developing a module specific to their feature, it should follow the instructions for using `TempoUIModuleProvider`.
The two approaches have quite a lot in common since `TempoUIAPI` is itself conforming to `TempoUIModuleProvider`.

The basic pieces are:
- Data Layer: A GraphQL fragment(s) and query, view model and `Section` make up the data layer for a given module.
- View Layer: regular views and a `SectionController` make up the view layer.

### Adding a module directly in `TempoUI`.
This can be done for approved shared modules. Discuss with Platform team if you think your module should be part of TempoUI versus residing in your plugin.

#### Data Layer
A Fragment, View Model and Section make up the data layer for a given module.

##### Fragment
 - Add the GraphQL fragment for the module to the `Plugins/TempoUI/TempoUIGQL/Sources/GraphQL/Queries/Fragments` folder group.
 - Update `GraphQL/Queries/Modules.graphql` with the new fragment type.
 - `glass apollo codegen -p TempoUI` and add generated Swift file(s) to API folder.
 - In its response parsing phase, `TempoAPI` needs some metadata to know what fragment to use when decoding the response. 
 Update `TempoUIGQL.SharedModuleMetadata.allModuleMetadata` to provide this information. e.g.
 `GenericModuleMetadata<CoolFeatureFragment>(moduleType: TempoModuleType.coolFeature)`

##### View Model
 - Make a folder group for the module within the existing `Models` folder group.
 - Make a view model for the module in this new folder group.
 - Conform the view model to `FragmentConvertible` in a `<ViewModelType>+FragmentConvertible.swift` file where it initializes from the incoming GraphQL fragment object.

##### Section
 - Create a `Section` for the module. e.g. `struct CoolFeatureSection: PlatformAPIs.Section`. 
 It is recommended to maintain the view model as a property of the `Section` model.
 - Conform the `Section` model to `TempoModuleSection` and setup your initializer. This will use the above created view model. e.g.
```swift
 extension CoolFeatureSection: TempoModuleSection {
    public init(viewModel: HeadingBanner,
                moduleMeta: TempoModuleMetadata,
                analyticsPayload: AnalyticsPayload) throws
    {
       // This is the very important place where you actually setup the `Section` data 
       // that will be used in the collection views diffable data source.
    }
```
 - `TempoUI` receives `AnyTempoModule` instances from `TempoAPI` and needs these converted to `Section` models for its diffable data source. 
 This is done using a `SectionMapper`. 
 `TempoUI` contains a `GenericSectionMapper` class and a `SectionManager` implementation that uses it for this purpose.
 e.g. 
 ```swift
    sectionManager.registerSectionMapperFor(CoolFeatureSectionController.Section.self)
 ```
  `GenericSectionMapper` can be used only if your `Section` conforms to `TempoModuleSection`, otherwise
  create a new specific `SectionMapper` class for the module that maps the `AnyTempoModule` to your new `Section` type. 
 - In order to support mapping of `AnyTempoModule` to the new `Section` type, 
 update `TempoUI`'s `TempoUIModuleProvider` implementation in `TempoUIPluginAPI` by
 registering the new `SectionMapper` in  `TempoUIPluginAPI`'s `func registerMappers(sectionManager: SectionManager, sectionDataSources: [DataSourceType])` 
 e.g. 
 ```swift
        sectionManager.registerMapper(CoolFeatureSectionMapper(),
                       forModuleType: TempoModuleType.coolFeature)
```

#### View Layer
Views and a `SectionController` make up the view layer.

##### Views
Just your regular UIKit views for your design. Nothing special really required for `TempoUI`.
 - Make a folder group for the module in the `Views` folder group.
 - Add the views in the new views folder group.
 - View will use the view model that is part of the data layer.

##### SectionController
Among other things, section controllers provide the actual layout and cells for the UI.
In order to integrate a `SectionController` 
 - Make a class conforming to `SectionController`in the SectionController folder group. e.g. `CoolFeatureSectionController.swift`.
 - Your `SectionController` needs to define a nested `SectionType`. This is the earlier created `Section` data model for your module.
 - Implement the code to provide the cell etc.
 - So `TempoUI` can make the correct `SectionController` for a given type of `Section` when the module is coming into view, register the
  `SectionController` for the given Tempo module type in
 	`TempoUIPluginAPI`'s `func registerSectionControllers(repository: SectionControllerRepository)`.
 	 e.g. 
 ```swift
        repository.register(CoolFeatureSectionController())
 ```
    
### Add a module within feature plugin using `TempoUIModuleProvider`
The overall steps are very similar to the above for modules added directly in `TempoUI`.
The main difference is that only the GraphQL portion goes into `TempoUIGQL`.
The rest of the implementation goes into the feature plugin, which conforms to `TempoUIModuleProvider`.
A similar approach to steps could also be used if a module already exists in a feature plugin, rather than being completely new.
This allows existing modules to be used in the `TempoUI` ecosystem.

#### Data Layer

##### Fragment
See the "Fragment" discussion above. The steps are the same for modules added directly in `TempoUI`.

##### View Model
 - Conform the view model to `FragmentConvertible` in a `<ViewModelType>+FragmentConvertible.swift` file where it initializes from the incoming GraphQL fragment object. This should be using the shared fragment generated in `TempoUIGQL`, not one within the feature.

##### Section
 - Create a `Section` and `SectionMapper` as described above. 
 - Register the `SectionMapper` in your plugins implementation of
  `func registerMappers(sectionManager: SectionManager, sectionDataSources: [DataSourceType])` 
 e.g. 
 ```swift
        sectionManager.registerMapper(PluginFeatureSectionMapper(),
                       forModuleType: TempoModuleType.coolFeature)
```

#### View Layer
The view code is implemented in the feature. `TempoUI` is blissfully unconcerned with the details.

##### SectionController
 - As described above, make a class conforming to `SectionController` in your project. e.g. `PluginFeatureSectionController.swift`.
 -  register the `SectionController` for the plugin's `TempoUIModuleProvider` conformance
 	`TempoUIPluginAPI`'s `func registerSectionControllers(repository: SectionControllerRepository)`.
 	 e.g. 
 ```swift
        repository.register(PluginFeatureSectionController())
 ```
 
#### Integration with TempoUI
The final step that is required is to inform `TempoUI` about your plugin so that it knows to call your `TempoUIModuleProvider` functions.
This requires a change to the `TempoUI` code. It is done in `SharedModuleProviderStorage.swift`.
You will need to add your plugin API into the `var moduleProviders` in a similar manner to the existing plugins.
e.g.
```swift
    container.getIfRegistered(CoolAPI.self).flatMap { moduleProviders.append($0) }
```

### Mocks & Testing
 - Try to have a single location for all the mocks developed in the main `TempoUI` target inside `#if DEBUG`. This ensures they can be used in tests and used in sample apps. 
 These will likely be ported to a `TempoUIMocks` framework later.
 - Make a Mock fragment and `AnyTempoModule` for the module type in its own file `Models/<ModuleName>/<ModuleName>+Mocks.swift`.
    All the mocking code should be inside `#if DEBUG` so it is not part of the release build.
 - Add the mock to the `mockAPI.mockModules` in `ContainerBootstrap.swift`.
 - If the module has text labels, it is helpful to use the type of the module. This way people unfamiliar with the various Tempo modules can see what is available already.

### Sample app integration (TODO)
 - For shared modules that are part of `TempoUI` anyone should be able to see an example of the module in the `TempoUIApp`.
 - Integrate the module into the `TempoUIApp` app `Tempo` view.
 
### Plugin integration

Once all of the above is done, it is time to use the overall `TempoUI` to show the Tempo driven page.
 `TempoUIAPI` provides most of the functionality needed including showing "loading" and "error" states.

The basic steps to integrate are:
- Add `PluginAPIs` and `PlatformAPIs` as dependencies in your project.
- Use `TempoUIAPI.makeStateCoordinator` providing all the details
```swift
let tempoUIAPI: TempoUIAPI = container.get(TempoUIAPI.self)
let coordinator = tempoUIAPI.makeStateCoordinator(
   presentationContext: .push(navigationController),
   analyticsData: TempoUIAnalyticsData(context: .healthAndWellness, page: .init(nm: .insurance), payload: [:]),
   diagnosticMetadata: ConcreteDiagnosticMetadata(owner: .wellness, tags: ["Insurance"], metadata: [:]),
   requestData: TempoUIRequestData(
       pathComponents: ["orchestra", "wellness", "graphql"],
       pageType: "MobileWellnessInsurancePage",
       layoutId: "mobile-simple"
   ),
   sectionDataSources: [.feedback(self)]
)
coordinator.start()
```
- Depending of what Tempo modules are used on your Tempo page, you may need to specify additional `sectionDataSource` or leave it empty.
- You may be in a situation where you want to refresh Tempo data from the server whenever user signs in or signs out, or in response to some other events. To do that, you will need to retain a reference to coordinator that you get back from `makeStateCoordinator` and call `invalidateCacheAndRefresh()` on it when you need it to refresh Tempo data.

#### With more control over "loading" and "error" states

- **Alternatively**, if you need more control, you may choose to use `makeCoordinator` instead of `makeStateCoordinator`. In this case you are responbile for showing "loading" and "error" states to the user. You will be providing `SectionsDataSource` (can use `ModulesDataSource`).
- Within your MVVM+C structure you will have a view controller that likely shows some kind of a loading view then, when the `Section`s are loaded in your data source it will start the coordinator.
```swift
let tempoUIAPI: TempoUIAPI = container.get(TempoUIAPI.self)
let coordinator = tempoUIAPI.makeCoordinator(
    in: .parent(viewController, viewController.view),
    dataSource: sectionsDataSource,
    delegate: self
)
tempoUICoordinator.start()
```

## How to get support

Use the `#glass-ios-tempo` channel in Slack for support.
