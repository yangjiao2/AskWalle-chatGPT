---
title: Page Performance
navTitle: Page Performance
---

# Page Performance

For the most important pages, we should be able to measure Page Content-Ready Time. Its time taken to initate a page, allocate memory, make a network call and show the content to the customer. While we are measuring the content load time we should also be able to measure various phases for page load time.

In OneApp we used to [Performance Traces](https://gecgithub01.walmart.com/Electrode-iOS/ELAnalytics/blob/b86dbba7ff4c5d833961c9d3415edebcb8b5284e/ELAnalytics/StandardAnalytics/StandardAnalytics%2BPerformance.swift#L21) for glass this performance traces has been updated to [AnalyticsPhasedEvent](https://gecgithub01.walmart.com/pages/walmart-ios/glass-platform/Classes/AnalyticsPhasedEvent.html) Developers will have to create a analytic phased event, add the phases at appropriate start points, complete the event and send the events to anivia. 

## Phases
Following are the possible phases for page load. Please note these are initial guidelines, based on your feature you might have some phases or might not.

|   Phase	| Definition  	|
|---	|---	|
|   initialize	|  View controller init is called | 
|   viewLoaded	|  View controller *loadView()* is called | 
|   viewCreated	|  View controller *viewWillAppear()* is called |
|   renderedPerceivedPage / showSpinner |  Constructed Perceived page / spinner |
|   createRequest  |  Network request created |
|   networkCall  |  Network call made |
|   parseResponse  |  Response parsed and converted to Data Model |
|   renderPage  |  Page loaded with content from network call |


## Dashboard
We can see all the page performance dashboards at [here](https://ce-logsearch01.prod.walmart.com/en-US/app/devices/launch_phases?form.perf_event=launch&form.tpid=iPhone)
Once we have analytics phases events added, please go to above dashboard and update the page name in page input panel. 

## Sample Implementation

In Glass we don't need to stop the previous phase. Everytime we start a new phase previous one is stopped. Below is the same code to demonstrate the same.


```swift

let phasedEvent = AnalyticsPhasedEvent(name: "performanceMetric", payload: ["name": "homepage"])
phasedEvent.startPhase(name: "initialize", payload: [:])
...

phasedEvent.startPhase(name: "viewloaded", payload: [:])

....

phasedEvent.startPhase(name: "viewCreated", payload: [:])

...

phasedEvent.startPhase(name: "createRequest", payload: [:])

...

phasedEvent.startPhase(name: "networkCall", payload: [:])

...

phasedEvent.startPhase(name: "renderPage", payload: [:])

...
phasedEvent.complete()

container.analytics.trackPerformanceEvent(phasedEvent)
```

