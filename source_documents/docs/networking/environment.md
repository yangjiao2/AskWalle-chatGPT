---
title: Networking - Environment
navTitle: Environment
---

# Environment

The application supports three primary environments:

- **Production:** The environment used by customers. 
- **Teflon:** A unified testing environment across all of Walmart that mirrors production.
- **Staging:** A somewhat unified environment that has the latest version of a given service.

A global enumeration, [`Environment`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/Environment/Environment.swift#L21), defines the global environment in the app. The current environment can be retrieved in code from the active `Container` instance and changed in the [Debug Panel](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/john/docs/networking-env/docs/faqs.md#how-do-i-access-the-debug-panel). Feature Plugins utilize this environment in their API clients to determine which base URL to use in requests. 

## Example

Typically, features will have their own `Environment` enumeration that supports additional environments such as "local" (mocks) and development. These additional environments are used in the feature's sample app.

```swift
enum CXOEnvironment {
    case production
    case teflon
    case staging
    case mocks
}
```
When running the Walmart App, the global `Environment` is mapped to the appropriate feature environment:

```swift
let environment = CXOEnvironment(environment: container.get())

private extension CXOEnvironment {
    init(environment: Environment) {
        switch environment {
        case .production:
            self = .production
        case .teflon:
            self = .teflon
        case .staging:
            self = .staging
        }
    }
}
```
Next, the feature environment is injected into the API client and the base URL is retrieved from an extension on the the environment:

```swift
final class OrchestrationClient {
    ...
    convenience init(environment: CXOEnvironment, networking: Networking) {
        self.init(
            baseURL: environment.orchestrationBaseURL,
            networking: networking
        )
    }
    ...    
}

private extension CXOEnvironment {
    var orchestrationBaseURL: URL {
        switch self {
        case .production:
            return URL(string: "https://www.walmart.com/orchestra/cartxo/graphql")!

        case .teflon:
            return URL(string: "https://www-teflon.walmart.com/orchestra/cartxo/graphql")!

        case .staging:
            return URL(string: "https://www-e16.walmart.com/orchestra/cartxo/graphql")!

        case .mocks:
            if CommandLine.arguments.contains("UITests") {
                return URL(string: "http://localhost:8800/graphql")!
            }
            return URL(string: "http://localhost:4000/graphql")!
        }
    }
}
```
