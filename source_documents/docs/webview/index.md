---
title: Web Content
navTitle: Web Content
---

# Web Content

- [About](#about)
- [WKWebView Limitations](#wkwebview-limitations)
- [WKWebView vs SFSafariViewController](#wkwebview-vs-sfsafariviewcontroller)
- [When should we use SFSafariViewController?](#when-should-we-use-sfsafariviewcontroller)
- [Infrastructure to load webpages](#infrastructure-to-load-webpages)
- [More Information](#more-information)

## About

This section outlines the limitations of [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) and the recommended approach to load any web content.

## WKWebView Limitations

Webviews are one of the heaviest objects that take significant size during its allocation. It is similar to launching another application within our mobile app and adding two additional processes, a content process and a networking process.
A single webview in the application will actually run three operating system processes: the application process, the web content process, and the web networking process. Creation of more webviews makes the app sluggish. 

Usage of [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview)  to load web content is not recommended due to the limitations outlined below.

-  **Native + Web Bridge** 
Inorder to have a successful back and forth communication between native and webview modules, a robust bridging infrastructure is required. At present, there is no messaging bridge built between app and web views. The JS messaging bridge can be quite complex, as we will have to support backward compatibility between the messaging APIs. At any point we would have multiple versions of apps while we have only one version for website. 

- **Cart Syncing**

  The sync of cart count between native app and webviews can impose severe challenges. 

- **Lesser Navigation control**

  While we can load a page inside webview, we would have lesser navigational control. If the user transition from walmart.com pages to any third party sites, say affiliate.walmart.com, we won't have good control over transition. 

- **Session Stitching**

  App and websites use different ways to manage user sessions. We don't have a uniform way to stitch app and webview sessions which can lead to skewed analytics metrics. 

- **Cookie Synchronization**

  App and website may manage cookies differently. This could lead to synchronization issues. 

- **User state management**

  Since native app and webviews can have different state management systems, syncing users state between webview and app can impose multiple challenges.

- **Breaking changes **
  
  Web app changes can break mobile apps as mobile apps are locked to specific versions. It is easy to miss backward compatibility as tests are run against current app versions.



## WKWebView vs SFSafariViewController

[WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) is part of the WebKit framework: It allows you to embed web content into your app as a seamless part of your app’s UI. You can present a full or partial view of web content directly in your app by loading a view that leverages existing HTML, CSS, and JavaScript content or create your own if your layout and styling requirements are better satisfied by using web technologies.

[SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) is part of the SafariServices framework, and lets you browse a web page, or a website right inside the app. With it, users can enjoy the same web browsing experience they get in Safari — including features like Password Autofill, Reader, and Secure Browsing — without ever having to leave the app.

## When should we use SFSafariViewController?

When you want to display websites inside your app without sending users to Safari, the best tool is [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller). By using this API, you can effectively embed the Safari interface — and many of its key features and privacy protections — into your app.

SFSafariViewController is best used when you need to display interactive web experiences on websites you don’t own, or showcase parts of your web content that are generally outside the scope of your app. It is recommended for features to make use of the infrastructure provided by WalmartPlatform instead of creating their own instances of `SFSafariViewController`

## Infrastructure to load webpages

WalmartPlatform provides utilities that helps features to load web content. Following is a code snippet that can used.

If any feature wants to load a basic html page (for example: https://corporate.walmart.com/privacy-security/california-privacy-rights) which doesn't require user's authentication details or other cookies which are required for transactions, APIs of `ExternalURLHandler` can be used.

Ensure `ExternalURLHandlerProtocol` is registered with your container 
### Registering ExternalURLHandler
```swift
import UIKit
import WalmartPlatform

container.register(ExternalURLHandlerProtocol.self) {
	ExternalURLHandler(UIApplication.shared)
}
```

### Loading web content

```swift
import WalmartPlatform

let url = URL(string: "https://www.walmart.com/all-departments") 
let urlHandler: ExternalURLHandlerProtocol = container.getIfRegistered()
urlHandler.openInSafari(
		url: url,
	  destination: URLHandlerDestination.internal(self),
  	completion: nil
)
```
Note that `URLHandlerDestination` can be configured based on your use case. Following options are available

- `internal` - Setting the configuration as internal presents SFSafariViewController from the provided view controller and loads the web content. This is preferred if any feature wants to load web content without taking user out of Walmart app. Optionally `SFSafariViewControllerDelegate` can be set to handle call backs.
- `external` - Loads the URL in Safari app by moving out of Walmart app.
- `internalWebView` - Presents a WKWebView instance within Walmart app and loads the content of the URL.

### Link-out web content

There are often cases where a particular feature is not supported in app but it might be supported on web. The linkout allows the URL to proceed to a browser for the functionalities that are not available in app. If a feature, say, category pages (with URL https://www.walmart.com/cp/electronics/3944) which lists products based on categories for a user, is not supported in the app, the feature modules can redirect users to view the webpages using link-out strategy. The API `authenticateAndLoadWebContent` attempts to authenticate and load the URL https://www.walmart.com/cp/electronics/3944 based on the [configuration](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/release/22.40/Plugins/PluginAPIs/PluginAPIs/APIs/AppRoot/AppRootAPI.swift#L70-L118) set by the caller of the API.  

Ensure AppRoot is registered with the container
```swift
import AppRoot
import WalmartPlatform

container.registerProvidingChild(AppRootAPI.self) {
    AppRootPluginAPI(container: $0)
}
```

The features can use linkout strategy to load a given URL within the app, ie, internally or move the user out of the app to load the contents on Safari. By default, the URL passed to `authenticateAndLoadWebContent` will take the user out of Walmart app and load the content in Safari app. However, if any feature wishes to load it internally, setting the [configuration](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/PluginAPIs/PluginAPIs/APIs/AppRoot/AppRootAPI.swift#L70-L118) as `internal` presents a `SFSafariViewController` on the current view controller and loads the contents of the URL.

```swift
import PluginAPIs
import WalmartPlatform

let appRoot = container.get(AppRootAPI.self)
let url = URL(string: "<replace-with-your-url>")

appRoot.authenticateAndLoadWebContent(
    for: url,
    currentViewController: <your-view-controller>,
    linkOutConfiguration: .init(
    		loadInternally: true
    )
)
```
`LinkOutConfiguration` can be used to set custom configuration related to title, message, CTA, image, preference to load, etc.

## More Information


- [Limitations with WebView](https://confluence.walmart.com/pages/viewpage.action?spaceKey=PG&title=Limitations+with+WebView)
- [Native / Web Hybrid Evaluation](https://confluence.walmart.com/pages/viewpage.action?pageId=328956861)
- [Why Is WKWebView So Heavy and Why Is Leaking It So Bad?](https://blog.embrace.io/wkwebview-memory-leaks/)
- [Should I use WKWebView or SFSafariViewController for web views in my app?](https://developer.apple.com/news/?id=trjs0tcd)
- [SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller)
