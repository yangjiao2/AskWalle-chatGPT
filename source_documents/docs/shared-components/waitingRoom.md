---
title: WaitingRoom
navTitle: WaitingRoom
---

#  WaitingRoom

## Description:

WaitingRoom also known as "Queueing" allows the customers to join the waiting room and buy the high demand items they want.

## Overview

The purpose of this Waiting Room is to limit the number of users that can access a specific item page at the same time. In special events, such as holidays
and when high priority limited items become available in stock, a lot of users may want to purchase a specific product at the same time, which can lead to
server overload. 
To avoid this, a queue is created for that specific product, and users requesting the item page are placed in a queue. Users in the queue are not allowed to view 
the item page 
or purchase that product until it's their turn.
While the user is waiting for their turn, the Waiting Room page is displayed instead of the usual item page. When it's their turn in the queue, 
they can access the item page and purchase the product.

The app will display a persistent/sticky banner that the user can interact with:
- to go back to see a list of their currently queued items, 
- leave the line on an individual item basis, 
- or go back to the product page for each of the queued items.

The feature supports several different statuses that a customer can be in throughout the queueing journey:
- Waiting in line to purchase an item (Messaging based on the likelihood of being able to purchase) -> pending type
- Time left to buy (User is now eligible to purchase after waiting) -> valid type

### Example:

<img src="images/WaitingRoom1.png" width="320" />

**The above image shows:**
- 5 items for which the customer is in the WaitingRoom for:
    - For 2 items (above divider line), the customer is eligible to buy (valid state). They can also see the time left to buy that item
    - For 3 items (below item line), The customer is waiting for their turn (pending state). They will get notified once it's their turn to buy
- Customer has the option to:
    - View the queued item
    - Leave the line
    - Close waitingRoom


## Topics:

### WaitingRoomPageView

These are the variables that are used to set waitingRoom view:

```swift
protocol WaitingRoomViewModel {
    /// Title of the waiting room page.
    var headerTitle: String { get }

    /// Description displayed below the title of the waiting room page.
    var headerBody: String { get }

    /// Title of the waiting room Call-To-Action
    var primaryButtonTitle: String { get }

    /// Name of the product.
    var productName: String { get }

    /// URL of a single image representing the product. Not displayed if nil.
    var productImageURL: URL? { get }

    /// Formatted price of the product.
    var productCurrentPrice: String { get }

    /// Original price of the product, already formatted. Not displayed if nil.
    var productWasPrice: String? { get }

    /// Estimated wait time until the ticket becomes valid
    var waitTimeInMinutes: Int { get }

    /// Controls whether waiting room should display estimated time information
    var showEstimatedWait: Bool { get }
}
```

### WaitingRoomErrors

Error subtype specific to Waiting Room errors:

- waitingRoomDisabled
    -  Waiting room feature is disabled. Waiting Room API will not be providing any functionality.

- dealNotFound
    - The Waiting Room API is not aware of any queues associated with the provided item.
    - This indicates a programming error and should not happen regularly.

- ticketError
    - Waiting Room API tried to issue a ticket, but received a ticket with status type error.

- waitTimeIsOver
    - This error may be thrown when a ticket has just been issued and it's already valid.

- networkingError
    - Some internal network request failed.

- unexpected442ResponseError
    - We got a 442 response for a vaild ticket, which is probably a backend error.


### Analytics:

#### Tracking buttonTaps in WaitingRoom:

```swift
    private func trackButtonTap(
        forButton button: Button,
        page: AnalyticsSchema.PageEnum,
        payload: AnalyticsPayload = [:]
    ) {
        analytics.trackButtonTap(forButton: button.rawValue,
                                 forPage: .init(nm: page),
                                 inContext: context,
                                 withPayload: payload)
    }
```

#### Tracking pageViews in WaitingRoom:

```swift
    func trackWaitingRoomPageView(itemID: ID.USItem, waitTimeInMillis: Int, admissionLikelihood: AdmissionLikelihood) {
        let payload: AnalyticsPayload = [
            WaitingRoomKey.waitTime.rawValue: waitTimeInMillis,
            WaitingRoomKey.queueType.rawValue: WaitingRoomAnalyticsQueueType(admissionLikelihood)
        ]
        trackPageView(page: .waitingRoom, itemID: itemID, payload: payload)
    }
```


