---
title: Tempo Preview
navTitle: Tempo Preview
---

# Tempo Preview

This capability allows for site merchants, developers and testers to get a preview of Tempo content when it is staged or scheduled.
It allows for testing of some configuration before it is live in production.

In summary, preview works by adding certain parameters to the `"tempo"` element in the CLS request body.
e.g.

```
"tempo": {
    "params": [{
        "key": "previewDate",
        "value": "1653841980000"
    }, {
        "key": "wm_preview",
        "value": true
    }],
    "targeting": "{\"deviceType\":\"ios\",\"deviceVersion\":\"22.20.0\",\"userState\":\"loggedOut\",\"storeTransactability\":\"nonTransactable\"}"
}
```

The Platform Tempo code will populate this additional data from a `TempoPreview` model.
NOTE: The above JSON example has a `"targeting"` element that is a String representation of JSON. 
The contents of this string is set at the Platform level.
Specifically, the default contents of the `params` and `targeting`.

But features can add additional Tempo targeting values to the `"params"` and the `"targeting"` element if needed. 

e.g. here we added `membershipTenure`, `storeId` & `zipcode` as a result of parsing a Tempo Preview QR code with the deep link - 
`walmart-debug://tempo?wm_preview=true&pageType=MobileHomescreen&zipcode=70601&previewDate=1653319980000&deviceType=ios&storeTransactability=transactable&storeId=521&userState=loggedOut&membershipTenure=ANNUAL`

```
{\"deviceType\":\"ios\",\"deviceVersion\":\"22.21.0\",\"membershipTenure\":\"ANNUAL\",\"storeId\":\"521\",\"storeTransactability\":\"transactable\",\"userState\":\"loggedOut\",\"zipcode\":\"70601\"}
```

## Integration

Features integrate with the [TempoPreviewAPI](/Plugins/PluginAPIs/PluginAPIs/APIs/TempoPreviewAPI.swift) plugin API. Primarily using these functions:-

```swift
/// Clients can observe this to trigger refresh.
func previewPublisherFor(pageType: TempoPreviewAPIType) -> AnyTempoPreviewPublisher

/// Convenience function to add the watermark view
/// on top of a view when in preview mode.
///  - parameter view: The view to add on top of.
///  - Note: This mode observing can be accomplished via the `previewPublisherFor(pageType:)` function.
func addWatermarkView(to view: UIView)

/// Convenience function to remove the watermark view
/// from on a view when exiting from preview mode.
///  - parameter view: The view to remove from.
///  - Note: This mode observing can be accomplished via the `previewPublisherFor(pageType:)` function.
func removeWatermarkView(from view: UIView)
```

### Model subscription

At some appropriate place, maybe in the plugin or the coordinator, the feature should subscribe to a preview publisher.
e.g.
```swift
func subscribeToTempoPreviewAPI() {
    guard let tempoPreviewAPI = container.getIfRegistered(TempoPreviewAPI.self) else { return }
    // The preview publisher for the specific page type should be used.
    tempoPreviewAPI.previewPublisherFor(pageType: .itemDetails)
        .dropFirst()
        .receive(on: DispatchQueue.main)
        .sink { [weak self] _ in
            guard let self = self else { return }
            switch result {
            case .success(let tempoPreview):
                // Upon receiving a successful value, the page should do something to refetch its content, 
                // using the given preview model in the request.
                self.fetchWithPreview(tempoPreview)
            case .failure:
                break
            }
        }
        .store(in: &store)
}
```

### Watermark

Features should add and remove the watermark view when they enter and exit preview mode as determined by the
model publisher. 

Example code:-

```swift
tempoPreviewPublisher?.dropFirst()
    .receive(on: DispatchQueue.main)
    .sink(receiveValue: { [weak self] previewResult in
    guard let self = self else { return }
    if case .success(let model) = previewResult {
        if model != nil {
            // enter preview mode
            previewAPI.addWatermarkView(to: self.view) // will add on the main view controller view.
        } else {
            // exit preview mode
            previewAPI.removeWatermarkView(from: self.view)
        }
    }
}).store(in: &store)
```

## Preview request

Clients need to make a new request to OL/Tempo when the model changes.

### Response Mapper

With the response mapper approach, features need to call the `TempoContextMetadata` `func with(preview: TempoPreview?) -> Self` 
function to include the preview data in the request. 

e.g.
```swift
// Adds the preview targeting to the default targeting.
let tempo = try context.tempo.with(preview: preview)

ModulePageQuery(
    channel: request.contentLayout.channel,
    pageType: request.contentLayout.pageType,
    tenant: context.tempoTenant,
    p13n: try p13n.asScalar(),
    tempo: tempo.asScalar(),
    layoutId: request.layoutId.rawValue,
    version: request.contentLayout.version
)
```

#### Implementation

The bulk of the implementation is in [TempoClient.swift](/Modules/Tempo/Tempo/Sources/TempoClient.swift) & [TempoContext.swift](/Modules/Tempo/Tempo/Sources/TempoContext.swift)

### TempoAPI

Note this implementation is not fully complete. 
Publishers of the `TempoAPIPreviewModel` model used in [TempoAPI](/Platform/PlatformAPIs/PlatformAPIs/PlatformAPIs/APIs/TempoAPI/TempoAPI.swift) need added to `TempoPreview`. 

```swift
// 1. Clients instantiate a `GenericTempoPreviewModel` using their `TempoPreviewPageType` page type to specialize it (make it unique).
let previewModel = GenericTempoPreviewModel(pageTypeGetter: TestPagePreviewModel(),
                                            previewDate: .init(rawValue: paramString))

// 2. Call the `TempoAPIConfiguration` instance to make a `ContentLayoutRequestContext` that has the standard parameters, passing in TempoPreview model.
// This sets up all the "tempo" and "targeting" parameters merging the preview model and the default parameters.
let configuration = tempoAPI.configuration
let context = configuration.makePreviewContentLayoutRequestContext(customerID: customerID,
                                                                   previewModel: previewModel)

// From here the workflow is just the general TempoAPI call pattern.

// 3. Instantiate a `ContentLayoutRequest` object using the above `ContentLayoutRequestContext`
let request = SomeTempoRequest(pathComponents: [],
                               context: contentLayoutRequestContext)
                               
// 4. Fetch the content layout, passing the request
let publisher = tempoAPI.fetchContentLayout(request: request,
                                            diagnosticMetadata: diagnosticMetadata,
                                            hooks: hooks)
                                            
// 5. TempoAPI will call `buildQuery` passing across an Apollo friendly `TempoAPIRequestContext` object that is constructed out of the request.
// 6. TempoAPI will do the fetch and publish back the layout/error.
```

#### Implementation

The bulk of the implementation is in [TempoTargeting.swift](/Platform/PlatformAPIs/PlatformAPIs/PlatformAPIs/APIs/TempoAPI/Request/TempoTargeting.swift)

## Testing 

Most testing is done using either the Tempo UI or directly using deep links.

### Deep link

#### Simulator 

You can use the `xcrun` command to send a deep link to the simulator. e.g.

`xcrun simctl openurl booted "walmart-debug://tempo?wm_preview=true&pageType=MobileHomescreen&zipcode=70601&previewDate=1653319980000&deviceType=ios&storeTransactability=transactable&storeId=521&zipcode=70601&userState=loggedOut&membershipTenure=ANNUAL"`

#### Device

In order to directly test with deep link URLs on a device you need to copy and paste the deep link into an app on your device. 
e.g. Slack, Messages, Notes to yourself etc. You can then tap the link or copy and paste to Safari. 

### QR code

Developers can also use the web Tempo UI as merchants do to generate and scan QR codes.

#### Basic QR code testing 

QR codes with the supported targeting can be setup using the Tempo Preview UI.

[Example](https://preview.cxtools-stg.walmart.com/?wm_preview=true&pageType=MobileHomescreen&targeting=%7B%22deviceType%22%3A%22ios%22%2C%22storeTransactability%22%3A%22transactable%22%2C%22storeId%22%3A521%2C%22zipcode%22%3A70601%2C%22userState%22%3A%22loggedOut%22%2C%22membershipTenure%22%3A%22ANNUAL%22%7D&zipcode=70601&wm_preview_date=1653319980000&tenant=WM_GLASS&channel=Mobile)

has a QR code that can be scanned from the app.
 
You can configure your specific page and targeting from this tool.

1. Configure the QR code in UI [staging](https://preview.cxtools-stg.walmart.com/)
    - Select the page type
    - Configure the "Configs" then press "Share".
2. Scanning the QR code from the native camera app and selecting "Open in Walmart" or scanner with app's scanner.

#### Configured modules and pages

If you are testing targeting for a configured module or page in the Tempo UI

1. Use "Page Preview" tab on a given page in Tempo.
2. For a specific module, go to the "Location" tab and tap the "Preview" image link for the page.

## Best practices and responsibilities for feature integration

It is up to individual pages to opt-in to this capabilities. 

1. Define your page type in the [TempoPreviewAPIType](/Plugins/PluginAPIs/PluginAPIs/APIs/Types/Tempo.swift) enumeration of pages.
2. Add your preview subscription using `TempoPreviewAPI` `previewPublisherFor(pageType:)`.
3. Honor the preview mode requests by refreshing your page when the preview publisher publishes a new value.
4. Show the watermark using `addWatermarkView` when the preview model was non-nil. 
Hide the watermark using `removeWatermarkView` when the value returns to nil.
5. In your handler to the model publisher, make a good attempt to navigate to your page.
e.g. switch to your tab using `AppRootAPI` and do any needed presentation other other logic to make your preview visible.
6. Test your integration with all possible targetings.

## How to get support

Use the `#glass-ios-tempo` channel for support.
