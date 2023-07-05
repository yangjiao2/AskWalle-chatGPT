---
title: Testing - Mocking Networking Calls
navTitle: Mocking Networking Calls
---

# Mocking Networking Calls

### Tools Needed

For mocking networking calls for UITests, we are going to use TestUtilities tool `MockServer` (which is `Codable`).

### Overview of `MockServer`

- Server must be set up first and it needs to be passed at app launch. (See section [Setting up mock server](#Setting-up-mock-server))

- You must configure the mock server to know what requests need to be respond to before requests are made, otherwise
    the connection will be closed with no response.
    
- Server responds to requests that match the provided rules.
   Following are a few example of available rules: Request matching `method`, `path`, `headers`, `queryParameters`, `body`,
   `fixture` etc.
    
- MockServer can remember the response for a request based on response configuration.

- If the MockServer is running, it will consume ALL requests whether or not a mock was set up or not. Basically if no response provided for 
    a request, request will fail with error "noResponseProvided".
    


### Setting up mock server

- Server needs to be set up first and needs to be passed at app launch.
- Ideally this should be the first step of your UITests if networking calls are required.
```
    var app: XCUIApplication! = XCUIApplication()
    let server = MockServer()
    app.launchEnvironment.mockServer = server
```
### Configuring response for a request

- Example 1 (Set up response in Data object form)
```
    server.respond(
    with: MockResponse(status: .ok,
                       headers: [.init(name: "Content-Type", value: "application/json")],
                       body: DataObjectCreatedFromaJsonFile),
    when: .path(matches: "/orchestra/featureModule/graphql"),
    removeAfterResponding: false
    )
```

- Example 2 (Set up response from a regular json file which will eventually converted to Data object by MockServer)
```
    server.respond(
    with: MockResponse(status: .ok,
                        headers: [],
                        fixturePath: "Fixtures/YourMoudleUItestsFolder/testFeature/graphql"),
                        when: .path(matches: "/graphql"
                        )
    )
```
- Once response is set up for a request, go ahead with making requests and UI element validation.
