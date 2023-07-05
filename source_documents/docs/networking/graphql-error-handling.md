---
title: Networking - GraphQL error handling
navTitle: GraphQL error handling
---

# GraphQL error handling

Handling GraphQL errors requires a little more effort on the client side compared to the traditional REST model, but ultimately results in a much richer and more flexible error experience.

## Prerequisites

This document assumes you have [Apollo code generation](graphql-infrastructure.md) running in your feature plugin and a [GraphQL client implementation](graphql-client.md) that works with your service.

## What kinds of errors are possible?

Instead of the traditional networking errors we all [know and love](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/418), GraphQL services always respond with a successful 200 status code. Any errors that occurred are embedded within the response body, which is a JSON object with two top-level keys: `data` and `errors`. `data` is an object containing the result of your query and `errors` is an array containing any errors that occurred. It is even possible to receive a response containing *both* data and errors, as long as the errors are low severity.

To describe the different kinds of errors that can arise when interacting with a GraphQL service, we have a [custom `GraphQLError` type](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/GraphQL/GraphQLError.swift). It may be any one of the following cases:

- `serviceErrors` The response contained one or more errors and no data. This case has one associated value: the array of errors.
- `recoverableError` The response contained both data and one or more errors. This case has two associated values: the array of errors and the decoded response data.
- `decodingError` The data in the response body could not be decoded by the client (rare).
- `invalidData` The response body was malformed and could not be parsed by the client (rare).
- `missingData` The response contained neither data nor errors (rare).

By default the `recoverableError` case will not be thrown. Instead, the data part of the response will be returned as if there were no errors. If you would like to inspect the error when the response contains both data and errors, set the `shouldFailOnRecoverableError` parameter to `true` when you construct your `GraphQLRequest` object.

## How to handle errors

The errors outlined above will be surfaced to callers as failure results in the Combine task started when you make a request to a GraphQL service. Note that the failure type of the task is a plain `Error` because other types of errors may also occur. Therefore it is necessary to manually cast the error to the `GraphQLError` type. Let's add some error handling logic to our [example client implementation](graphql-client.md#making-a-request).

```swift
import Combine
import MyPluginGQL
import WalmartPlatform

struct MyError: Error {
    var message: String
}

func myRequest() -> AnyPublisher<MyQuery.Data, MyError> {
    let url = URL(string: "https://dev.walmart.com/graphql")!
    let query = MyQuery()
    let request = GraphQLRequest(url: url, operation: query, shouldFailOnRecoverableError: true)
    let networking = Networking(
        session: .shared,
        configuration: .init(
            isAPQClientEnabled: true,
            isAPQInterceptorEnabled: true
        )
    )

    return networking.dataTaskPublisher(for: request)
        .receive(on: DispatchQueue.main)
        .mapError { MyError(error: $0) }
        .eraseToAnyPublisher()
}

extension MyError {
    init(error: Error) {
        if let graphQLError = error as? GraphQLError {
            switch graphQLError {
            case .decodingError, .encodingError, .invalidData, .missingData:
                self.init(message: "GraphQL error")
            case .recoverableError(let errors, _), .serviceErrors(let errors):
                let errorCode = errors.first?.extensions["code"] as? String
                self.init(message: errorCode ?? "GraphQL error")
            }
        } else {
            self.init(message: error.localizedDescription)
        }
    }
}
```

Using the `mapError` operator, we're able to inspect any failure that `dataTaskPublisher` publishes and transform it into the more user-friendly `MyError` type which we could then use to populate an error alert. By grabbing the associated value from the `serviceErrors` case we get access to the array of error objects from the service response.

## See also

- [GraphQL specification for errors](https://spec.graphql.org/June2018/#sec-Errors-and-Non-Nullability)
