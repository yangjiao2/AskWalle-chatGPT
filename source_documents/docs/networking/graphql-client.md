---
title: Networking - GraphQL client
navTitle: GraphQL client
---

# GraphQL client

Although we use Apollo's out-of-the-box code generation capabilities, we have our own GraphQL networking client wrapper that mirrors the way we make REST network requests. This guide will show you how to use that wrapper to make a request against a GraphQL service.

## Prerequisites

This guide assumes you have Apollo code generation running in your feature plugin, either from an existing setup or by following the [infrastructure setup guide](graphql-infrastructure.md).

## Making a request

Let's make a query. In Xcode, go to your `GraphQL/Queries` folder and create a new file from the "Empty" template named `MyQuery.graphql`. Don't add it to any build targets.

After writing your query, build the app and you should see a new file appear in the `GraphQL/API` folder named `MyQuery.graphql.swift`. Add it to your Xcode project, along with any other files in that folder that aren't already in Xcode. This time make sure that the new files are added to your framework build target.

At this point we can start writing Swift code against the generated GraphQL API. The pattern is very closely aligned with regular old HTTP requests.

```swift
import Combine
import MyPluginGQL
import WalmartPlatform

func myRequest() -> AnyPublisher<MyQuery.Data, Error> {
    let url = URL(string: "https://dev.walmart.com/graphql")!
    let query = MyQuery()
    let request = GraphQLRequest(url: url, operation: query)

    return container.networking.dataTaskPublisher(for: request)
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}
```

Congratulations! You now have all the tools you need to write a great feature against a GraphQL backend.

## Best practices

Although the data models that Apollo generates are awesome and easy to use, we don't recommend referencing them directly in you feature code; instead, you should maintain your own separate data model. This approach delivers several important benefits:

- Your feature code is decoupled from the GraphQL schema. When the schema changes, you only have to update the code that converts the GraphQL response to your data model.
- Your data model will be a lot more readable than the Apollo-generated models.
- Your data model can be more protocol-oriented, making it easier to write well-architected, testable code.

Here's an example that shows what it looks like to implement your own data model.

```swift
// Custom data model definition
struct Product {
    let id: String
    let name: String
}

// Convert Apollo-generated data model to custom data model
extension Product {
    init(_ product: ProductQuery.Data.Product) {
        id = product.id
        name = product.name
    }
}

// Usage example: fetch Apollo-generated model internally, but publish custom model
func fetchProduct(id: String) -> AnyPublisher<Product, Error> {
    let query = ProductQuery(id: id)
    let request = GraphQLRequest(url: url, operation: query)
    let networking = Networking(
        session: .shared,
        configuration: .init(
            isAPQClientEnabled: true,
            isAPQInterceptorEnabled: true
        )
    )

    return networking.dataTaskPublisher(for: request)
        .map { Product($0.product) }
        .eraseToAnyPublisher()
}
```
