---
title: Tempo Response Mapper
navTitle: Tempo Response Mapper
---

# Response Mapper

The "response mapper" approach described below. This approach allows high customization but 
requires much more work to integrate. This may eventually be replaced with [TempoAPI](tempo-api.md) approach.

## Request

### Custom `params` 
In order to add custom parameters to the `tempo.params` clients can TODO

### Custom `targeting` 
In order to add custom parameters to the `tempo.targeting` JSON clients can 

## Response handling

### Using `Tempo` response mapping

#### Update your query

First, we need to add fields to your query to ensure that we have all the information necessary to stitch together modules and layouts from a `contentLayout` response. You'll need to include the following additional fields in addition to any fields you're fetching for your own use:

- `contentLayout.modules.matchedTrigger.zone`
- `contentLayout.layouts.id`
- `contentLayout.layouts.layout`

Which would roughly look like this in your query:

```GraphQL
query {
  contentLayout(
    // ...
  ) {
    modules(
      // ... 
    ) {
      // ... 
      matchedTrigger {
        zone
      }
    }
    layouts {
      id
      layout
    }
  }
}
```

#### Update your scalar type

> Note: This should only be necessary once per `*GQL` target.

If this is the first time you’ve included `contentLayout.layouts.layout` in a query, it’s likely that your `*GQL` target no longer compiles. This is because the schema describes raw json using a `JSON` scalar type, and so we need to provide a concrete type that `Apollo` can use for those values.

To do this, create a `Scalars.swift` file in your GraphQL target’s `Sources/GraphQL` directory with the following contents:

```Swift
import Tempo

typealias JSON = Tempo.JSONScalar
```
See the `GraphQL/Scalars.swift` in the `Tempo` project for an example.

This most likely will also require adding `Tempo` as a dependency to your `*GQL` target.

#### Add `Convertible` conformances to your `Apollo` generated types

> Note: This step is currently a little more complicated because the schema is still fluctuating. Ideally.once things settle down, we can remove some of this complexity. 

`ResponseMapper` uses a handful of protocols to understand how to interpret every features different `Apollo` generated types, so we'll need to add those conformances to your types:

Currently (while the schemas are in flux), this entails conforming the following four types from your generated `Apollo` response to related protocols in `Tempo`. The "Sloppy"ness of these types is due to the current instability in the GraphQL schemas, and the ongoing changes in which fields have been annotated as `required`.

```swift
typealias ApolloResponse = HomePageModulesQuery.Data

extension ApolloResponse.ContentLayout: SloppyResponseConvertible {
    public var sloppyLayouts: [Layout?]? { layouts }
    public var sloppyModules: [Module?]? { modules }
}

extension ApolloResponse.ContentLayout.Layout: SloppyLayoutConvertible {
    public var sloppyId: String? { id }
    public var sloppyLayout: JSONScalar? { layout }
}

extension ApolloResponse.ContentLayout.Module: SloppyModuleConvertible {
    public var sloppyMatchedTrigger: MatchedTrigger? { matchedTrigger }
}

extension ApolloResponse.ContentLayout.Module.MatchedTrigger: SloppyMatchedTriggerConvertible {
    public var sloppyZone: String? { zone }
}
```

Optionally, for any types that have their properties correctly annotated as `required`, you can omit the properties by instead adding a conformance to the non `Sloppy` version of the above protocols. For features hitting discovery orchestration, this (at the time of writing) includes `Module` and `Module.MatchedTrigger`. Using these, the implementation in `Home` currently looks like this:

```swift
typealias ApolloResponse = HomePageModulesQuery.Data
extension ApolloResponse.ContentLayout: SloppyResponseConvertible {
    public var sloppyLayouts: [Layout?]? { layouts }
    public var sloppyModules: [Module?]? { modules }
}
extension ApolloResponse.ContentLayout.Layout: SloppyLayoutConvertible {
    public var sloppyId: String? { id }
    public var sloppyLayout: JSONScalar? { layout }
}

extension ApolloResponse.ContentLayout.Module: ModuleConvertible, SloppyModuleConvertible {}
extension ApolloResponse.ContentLayout.Module.MatchedTrigger:
    MatchedTriggerConvertible, SloppyMatchedTriggerConvertible {}
```

However! These may need to be adjusted as you download different versions of the schema. Hopefully this can settle down once the schema does.

#### Declare the `Module` and `External` types your feature expects in the response

Now that we've taught `ResponseMapper` about the data it'll be parsing _from_, let's build the types we want it to parse to.

First, declare a `Module` type for the modules you expect to receive from this query. It's likely you already have these data models defined if you're following the [GraphQL client best practices](../networking/graphql-client.md#best-practices), but this should effectively describe how your feature would like to describe the values in the `contentLayout.modules` field of the GraphQL response.

For example in Home, we use an enum where each case relates to a specific type from the GraphQL's `TempoModuleConfigs` union:

```swift
enum Module {
    case carousel(CarouselModel)
    case povCarousel(PovCarouselModel)
    // ...
}
```

We then can implement the transformation from our `ApolloResponse.Module` to our `Module` by adding a `Module: ExpressibleByModule` conformance, and implementing the init. While not required, it's encouraged that you use this transformation to tighten up the models: unwrapping any possibly `nil` required values, transforming `String`s into more appropriate `enum`s, etc.

```swift
extension Module {
    init(rawModule: ApolloResponse.ContentLayout.Module) throws {
        /// Conversion logic goes here
    }
}
```

Additionally, declare an `External` type for modeling any client ("external" to tempo) modules, and adding a conformance to `Tempo`s `ExpressibleByLayoutReference`. This conformance is satisfied by defaults if your `External` type already conforms to `RawRepresentable where RawValue == String`.

```Swift
enum External: String {
    case buttonBar
    case amendBanner
}

extension External: ExpressibleByLayoutReference {}
```

#### Convert your response with `TempoContext.responseMapper`

Insert a call to the `responseMapper.makeFlattenedLayout` in your combine pipeline.


```swift
func getModules(request: ModuleRequest) -> AnyPublisher<ModuleResponse, Error> {
    // ...
        .flatMap { [session] in session.dataTaskPublisher(for: $0) }
        .tryMap { response in 
            try context.responseMapper.makeFlattenedLayout(
                // These are the `Module` and `External` types we defined above
                of: Module.self, External.self,
                from: response.contentLayout,
                withLayoutAtId: "simple-1column")
        }
        // ...
        .eraseToAnyPublisher()
}
```
