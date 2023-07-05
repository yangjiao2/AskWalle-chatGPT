---
title: Persisted Queries
navTitle: Persisted Queries
---

# Persisted Queries
- [Persisted Queries](#persisted-queries)
  - [**Introduction**](#introduction)
    - [**What are Persisted Queries and how are they used at Walmart?**](#what-are-persisted-queries-and-how-are-they-used-at-walmart)
    - [**Shadow Gateways**](#shadow-gateways)
    - [**Ephemeral vs Non-ephemeral registration**](#ephemeral-vs-non-ephemeral-registration)
    - [**The operations.json file**](#the-operationsjson-file)
  - [**Persisted Query Registration**](#persisted-query-registration)
    - [**Developer Process**](#developer-process)
    - [**GlassCLI's Registration Process**](#glassclis-registration-process)
      - [**Local Ephemeral Registration**](#local-ephemeral-registration)
      - [**CI Only Non-ephemeral Registration**](#ci-only-non-ephemeral-registration)
  - [**Persisted Query Invocation**](#persisted-query-invocation)
  - [**Market App Integration**](#market-app-integration)
    - [**Networking Plugin Configuration**](#networking-plugin-configuration)
    - [**CCM Opt Out Configuration**](#ccm-opt-out-configuration)
  - [**Further Reading**](#further-reading)

---
## **Introduction**
### **What are Persisted Queries and how are they used at Walmart?**
Glass utilizes GraphQL for our networking layer, which has many benefits, namely it allows developers to quickly iterate on network requests as the do not have to wait for contract changes and can request only the needed information from the BE. This becomes a problem with vanilla GraphQL as malicious actors also have access to the same schema introspection and can gain insights into the inner working of Walmart's back end systems. 

This is where persisted queries come into play. In a nut shell, they allow us to register the specific queries used by our Platforms (Apps and Web), turn off introspection and reject requests using a unregistered queries. 

### **Shadow Gateways**
Shadow gateways are duplicate gateway environments of production, teflon, etc that are not exposed outside the Walmart WAN (VPN). The only difference between these gateways and the ones used by live traffic of our applications is the shadow gateways have introspection enabled. While they could in theory be used to route app traffic, PLEASE DO NOT DO THIS, they were only designed to fulfill our introspection and code generation needs. DO NOT MAKE GQL REQUESTS TO THESE GATEWAYS!

### **Ephemeral vs Non-ephemeral registration**
[mGQL](https://dx.walmart.com/documents/product/Managed%20GraphQL%20(mGQL)/overview), the backend that provides Walmart's persisted query registration capability, has the concept of ephemeral and non-ephemeral queries. In short, ephemeral queries are temporary and intended to support developer testing and iteration while non-ephemeral queries are more permanent and intended for production and prolonged usage.

### **The operations.json file**
The operations.json file is the source of truth for the queries in the repo and their associated hashes. They consist of the details needed to make persisted queries work, including the hash, the operation name, and the query. They are the medium we use to register our queries. Below is an example of what they look like:
```json
{
    "1b3d332933a143d0f20319fd4c17e9bad1e658072c98746fe880aed5aba6bcde": {
    "name": "GetAccountApparels",
    "source": "query GetAccountApparels {\n  account {\n    __typename\n    apparel {\n      __typename\n      ...FashionFragments\n    }\n  }\n}\nfragment FashionFragments on AccountApparel {\n  __typename\n  fashionPreferenceId\n  modelPreference\n  byomImageId\n  zeekitModel {\n    __typename\n    personaId\n    name\n    feet\n    inches\n    size\n    imageURL\n  }\n}"
  }
}
```
---
## **Persisted Query Registration**
### **Developer Process**
1. Backend team(s) makes changes to their schema
2. iOS developers utilize a [Shadow Gateway](#shadow-gateways) to preform introspection and code generation for their platform and feature.
    - The shadow gateways are used to get the schemas for their respective environment. However, CI checks will only allow codegen from the production schema to be merged to development. 
3. During code generation (which updates the operations.json file), platform provided build tooling registers the new queries ephemerally to support local testing and development iteration.
4. iOS developers, once development is complete, open a PR to merge their new query code to the development branch
5. CI checks ensure that the schema used to run the code generation has been merged into production. This is done to prevent non production queries from going out into the wild and breaking the app.
6. The iOS developers merges their PR. 
7. CI registers all queries in the application as persisted (non-ephemeral). For the development branch and release branches this happens once a day at Midnight. However, for release branches this step also happens as part of the SC and RC builds to make it impossible to ship a version of the app to the AppStore with unregistered queries.

### **GlassCLI's Registration Process**
GlassCLI houses the build tooling that makes persisted query registration on the iOS Platform possible. It occurs in two flavors, local ephemeral registration and CI only non-ephemeral registration.

#### **Local Ephemeral Registration**
During the code generation process, the changed or new queries are registered against mGQL's registry ephemerally following the below process:
1. Apollo codegen is kicked off manually or via runscripts in the build system.
2. Code generation creates or updates the feature specific operations.json file with the new queries and their hashes (THIS MUST BE COMMITTED).
3. GlassCLI, assuming there were no issues in code generation, then kicks off the registration task for only the changed operations.json file.

#### **CI Only Non-ephemeral Registration**
1. CI kicks of registration either via a cron or because an SC or RC is being built. This is done using the `glass apollo register-persisted-queries --force-persistence` GlassCLI command.
2. GlassCLI searches the repo for all operations.json files
3. GlassCLI parses all operations, then regardless of the file they came from piles them all up before splitting them into chunks of 50.
4. GlassCLI registers each chunk of 50 operations 1 at time until all are successfully registered
5. If a registration call fails it is retried up to 5 times before proceeding onto the next batch.
6. Any failures will result in RC and SC builds failing and not being uploaded to the AppStore or TestFlight.

---
## **Persisted Query Invocation**
Persisted query invocation is a separate technology from registration and is used to to prevent malicious actors from seeing the content of our queries by proxying network traffic. It also has the nice side effect of reducing round trip query time as we are sending much less data. It works by converting queries (not mutations) from POST requests with a body containing the query document, to a GET request that has the operationName, hash and variables inserted into the url following the the below format:
```
https://www.walmart.com/orchestra/myGQPAPIPath/graphql/myGQLOperation/1b3d332933a143d0f20319fd4c17e9bad1e658072c98746fe880aed5aba6bcde/testInfo1/testInfo2?id=1b3d332933a143d0f20319fd4c17e9bad1e658072c98746fe880aed5aba6bcde&name=JohnyTest
```
NOTE: If the url (post conversion) is bigger than 6MB we throw it away and use the post request. This is due to url restrictions at the Akamai CDN layer.

---
## **Market App Integration**
Platform has provided a few ways to configure Persisted Query Invocation (not registration as that is market agnostic). The first being a [configuration struct](#networking-plugin-configuration) that allows invocation to be turned on or off wholesale for that applicate. The second being an [opt-out CCM](#ccm-opt-out-configuration) list based on api paths

### **Networking Plugin Configuration**
The networking configuration is just a simple struct that controls a few things related to Persisted Queries in the platform Networking API. See the below example for a guide on how to do this:
```swift
extension Networking.Configuration {
    static var usa: Self {
        .init(isAPQClientEnabled: true, isAPQInterceptorEnabled: true, staleCookieJar: HTTPCookieStorage.shared)
    }
}
```
The above configuration is then injected into the Networking API when it is registered as part of the app's startup sequence.

### **CCM Opt Out Configuration**
CCM opt-out provides a way to opt out of invocation for specific API paths. An API path can be considered the feature specific api path for a back end service and follows the below pattern:
```
https://BASE_URL/orchestra/API_PATH/graphql

EX: https://www.walmart.com/orchestra/home/graphql
```
If you wanted to opt out of persisted query invocation for the above path you would set up the following ccm key in your market scope:
```json
"platform.networking.persistedquery.optout.apipath.list": {
    "value": [
        "home"
    ],
    "minVersion": "22.36.1"
}
```

---
## **Further Reading**
- [Apollo Persisted Queries Documentation](https://www.apollographql.com/docs/apollo-server/performance/apq/)
- [mGQL High Level Documentation](https://dx.walmart.com/documents/product/Managed%20GraphQL%20(mGQL)/overview)