---
title: Analytics - Diagnostic Events
navTitle: Diagnostic Events
---

# Diagnostic Events

**Diagnostic Error Definition:**
> Bad things which affect user experience that are our (walmart's) fault, but aren't crashes.

### Details

[Diagnostic events](https://gecgithub01.walmart.com/walmart-ios/glass-platform/blob/development/WalmartPlatform/Analytics/Intents/CustomerDiagnosticsEvent.swift) enable engineers to track meaningful events that impact the consumer in a unified way. Think of these as log messages but less spammy and more informative. The events fall into two categories: info and error. Both categories of events share the same general structure outlined below.

```
{
  "act" : "log",
  "file" : "myFile.swift",
  "function" : "myCoolFunction()",
  "line" : 1337,
  "logType" : "error",
  "message" : "An informative message",
  "owner" : "@walmart-ios/glass-platform-ios",
  "tags" : [
    "tag-0",
    "tag-1",
    "tag-2"
  ],
  "metaKeyOne": "customValueOne",
  "metaKeyTwo": "customValueTwo"
}
```

- **act.** Diagnostic events are sent to Anivia like product analytics but use `log` as the event type.
- **file.** The file where the event originated.
- **function.** The function where the event originated.
- **line.** The line number where the event originated.
- **logType.** The type of diagnostic event, [`info` | `error`].
- **message.** An informative message describing the event that can aid in debugging. 
- **owner.** The *owner* of the event. Teams should use their corresponding GitHub code owner identifier for this value. Constants have been automatically generated from the `CODEOWNERS` file for convenience.  
- **tags.** A collection of additional identifiers to assist in identifying the event. These are particularly useful for grouping errors. 
- **metadata.** A collection of additional and relevant key/value pairs. These are particularly useful for narrowing a query.

### Example

```
{
  "logType": "error",
  "line": 119,
  "file": "Plugins/MyItems/MyItems/Sources/DataSource/Actions/FetchPageModule.swift",
  "function": "diagnosticEvent",
  "act": "log",
  "owner": "@walmart-ios/glass-recommendations-ios",
  "message": "Decoding failure. error=Error at path \"contentLayout.modules.0.configs.displayMode)\": missingValue",
  "tags": [
    "graphql",
    "decoding"
  ],
}
```

The above log event is an example of an error event caused by a GraphQL contract breach (a missing required field). Note the event is easily traceable to the origin team via the `owner` and the root cause easily determined by the `message`. It's important to include enough context in the event to understand the meaning of a log event.

### Info 

The `info` level is intended to inform teams of salient events that impact the consumer in a non-adverse way. These events are primarily intended to augment existing product analytics and to assist in debugging and monitoring in production. They're similar to your canonical log statement but should be used strategically such as providing more context in a user-flow or tracking a notable event. While they can be used more liberally than the `error` level, judgement should still be exercised. 

#### Example

Imagine you're on the identity team and writing some code to migrate the authentication token from OneApp to Glass using the shared keychain and you want to monitor or debug your code in production. Recording an `info` would be a perfect way to do this!

```swift
let event = CustomerDiagnosticsEvent(
    message: "Walmart one-app auth token migrated successfully.",
    owner: .postTx,
    metadata: ["OneAppAuthMigrationResult": "success"]
)
container.analytics.track(event: event)
```

The message clearly conveys the meaning of the event and the `metadata. OneAppAuthMigrationResult` field makes it easy to identify. A complementary `error` event also be useful in this particular case. 

See [Recording Events](#recordingevents) for more information on the recording API.

### Error

The `error` level is intended to inform teams of events that **adversely** impact the consumer. Teams should exercise judiciousness when recording events at this level as occurrences of them signal failure in systems and a poor or broken consumer experience. For example, if a server response is missing critical data that prevents the user from engaging in a feature, an error event could be recorded to alert teams of the issue. 

Other worthy examples of events that could warrant the `error` level include: 

* **API Contract violations.** An API contract violation occurs when a service violates their agreed upon contract with the client. Some examples include:
   - **Missing Data.** The payload is missing requisite data such a field.
   - **Invalid Data.** The data returned to the client is invalid, e.g. a malformed URL or deep link.
   - **Unexpected Status Code**. A status code the client does not expect.
* **Invariant checks.** Invariant, sanity, and pre- and post-condition checks often take the form of an assertion and are removed when the app is built for release. Therefore, these checks can “fail” unnoticed in production and cause unexpected behavior in applications. See [DiagnosticAssert](#diagnosticassert).

## Recording Events 

To record a diagnostic event, one simply has to create an instance of a `CustomerDiagnosticEvent` and record it using the `AnalyticsTracking.trackDiagnosticsEvent(_:)` API. 

```swift
// create the event (by default the `info` is used)
let event = CustomerDiagnosticsEvent(message: "The API has been violated!", owner: .recommendations)

// record the event
func trackDiagnosticEvent(container: Container, event: CustomerDiagnosticsEvent) {
    let analytics = container.getIfRegistered(Analytics.self)
    analytics.trackDiagnosticsEvent(event)
}
```

### DiagnosticError

When tracking errors, often times you want to track existing instances of a `Swift.Error` type. Instead of manually mapping the error to a diagnostic event, you can have the error type conform to the [`DiagnosticError`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/Analytics/Diagnostics/DiagnosticError.swift) protocol and create an event using the convenience initializer, [`init(error:owner)`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/Analytics/Diagnostics/DiagnosticError.swift#L66).

#### Example

For example, the [`TempoResponseError`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/Tempo/Tempo/Sources/Response%20Mapping/ResponseMapper.swift#L11) that occurs as a result of unexpected or invalid data from Tempo represents a quintessential error that clients should track. Conforming `TempoResponseError` to `DiagnosticError` allows the error to be easily tracked. 

```swift
// conformance
extension TempoResponseError: DiagnosticError {

    public var diagnosticMessage: String {
        switch self {
        case .dataCorrupted(let ctx),
             .layoutNotFound(let ctx),
             .missingValue(let ctx):
            return ctx.debugDescription
        }
    }

    public var diagnosticErrorTag: DiagnosticErrorTag { .tempo }

    public var diagnosticTags: [AnalyticsValue] {
        switch self {
        case .dataCorrupted: return ["dataCorrupted"]
        case .layoutNotFound: return ["layoutNotFound"]
        case .missingValue: return ["missingValue"]
        }
    }
}

// tracking
func trackDiagnosticError(container: Container, error: DiagnosticError) {
    let analytics = container.getIfRegistered(Analytics.self)
    analytics.trackDiagnosticError(error, owner: .discovery)
}
```

### DiagnosticAssert

Swift assertions are a valuable tool to ensure invariants are maintained during development. However, they are removed in Release builds and can unknowingly _fail_ in production! To track these occurrences, it is recommended to use the [diagnostic assertion wrappers](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/Analytics/Diagnostics/DiagnosticAssertions.swift). These wrappers log a diagnostic error event in addition to asserting the invariant condition in Debug builds. Note diagnostic asserts will **not** crash the app in production. 

#### Example

```swift
// replace these
assert(myInvariantCondition, "My invariant was not met...")

// with diagnosticAssert equivalents 
diagnosticAssert(myInvariantCondition, "My invariant was not met...", owner: .ads)
```

### CodeOwner

All diagnostic events must include an owner to map the event to a specific team. In Glass, the `owner` corresponds to the GitHub codeowners. A [`CodeOwner`](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Modules/WalmartPlatformExtensions/WalmartPlatformExtensions/Sources/Analytics/Diagnostics/CodeOwner.swift) enumeration has been generated from the CODEOWNERS file for convenience. 

## Best Practices 

Below are a collection of best practices and considerations when instrumenting your code with diagnostic events. 

- **Event Volume.** While the monitoring tools at Walmart can store _a lot_ of data, do not excessively record events in a place as a loop. The platform automatically samples info events for you.
- **Bugs.** When fixing production bugs, consider if a diagnostic event could have caught the bug sooner or detect the issue in staging. 
- **Customer Anguish.** Any code path that results in friction for the customer should result in a diagnostic event being recorded. Friction can manifest as unexpected application state, mishandled deep links, etc. Remember to only add diagnostic events if product analytics do not adequately track the event.
- **Tags and Metadata.** Include identifying tags and metadata to make querying easier. You shouldn't have to write a regex to track your event. Additionally, consider augmenting the log event with additional key/value pairs to provide more context.

## API Notes

### Sampling

TODO: Provide info on sampling.

### Force Upgrades

The application stops sending diagnostic events when the app does not meet the minimum version supported by Walmart.

## Monitoring and Querying

### Querying

All diagnostic events can be queried using the filter condition `event=log`:

```
index="devices" aid=usoa tpid=iPhone act=log
```

Specific level:

```
index="devices" aid=usoa tpid=iPhone act=log logType=error
```

Specific owner:

```
index="devices" aid=usoa tpid=iPhone act=log owner="@walmart-ios/glass-recommendations-ios"
```

Specific tags:

```
index="devices" aid=usoa tpid=iPhone act=log tags{}=graphql,decoding
```

Error free rate by `owner`:

```
index="devices" aid=usoa tpid=iPhone act="log" 
    | eval sampleRate=coalesce(sampleRate,1)
    | stats dc(eval(if(logType="error", sid, NULL()))) AS errorCount first(sampleRate) as sampleRate by owner
    | appendcols [search index="devices" aid=usoa tpid=iPhone  | stats dc(sid) as sessionCount ] 
    | filldown sessionCount 
    | foreach errorCount [ eval errorFreeRate = round((1- (errorCount/sessionCount)) * 100, 4) ]
```

### Monitoring 

TODO: Provide example of feature dashboard


