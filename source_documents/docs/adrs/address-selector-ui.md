Select Address Components
=================================
- Author: [Wellington](https://gecgithub01.walmart.com/w0m01q1)
- Status: In Progress
- Decision: In Progress

## Problem
Although we (Account) do not use it in any of our flows, the Address Selector is a component that is used in many different places throughout the app. 
Often times there are competing requirements. 

Here are some examples:

+ The Global Location header might want to display a custom message for their flow.
+ The Walmart+ team wants to show a bottom sheet after the customer selects and address but before the Location is set.
+ Checkout wants slightly different text for their gifting flows.
+ Subscriptions wants to cross reference addresses with GBAS to determine if they are eligible.


[Confluence Page](https://confluence.walmart.com/display/PG/Account+-+Address+Component+Scope)

All of these competing requirements are putting a lot of business logic requirements on the Address Selector.
We want to support these teams having an address selector without getting involved with their Business Logic.

## Solution
In the long run, we do not want to own any of the business logic required.
We want instead to provide a set of customizable UI Components that any teams can use and customize to their needs.

### Approach
We already have FeatureUI project that is perfectly suited for this use case. 

1. Create an AccountSharedUI project inside FeatureUI.
2. Create the Base UI components for the Address Selector Views — the table, as well as the cells. These components should be designed as open classes in order to be extendable and customizable by users.
3. Create documentation and provide examples for how to use these components
4. Rework the code in Account to use the Shared UI components
5. Create a new API in AddressAPI that allows API users to query for a list of DeliveryAddresses.


### Suggested Implementation

```swift 
open class AddressSelectorCell: BaseTableViewCell {

      public let radioButton = LDRadio()
      public let editButton = LDLinkButton()
      public let nameLabel = LDLabel()
      public let addressLabel = LDLabel()
      public let isDefaultLabel = LDLabel()
      public let actionStackView = UIStackView()
      public let itemStackView = UIStackView()
      public let dividerView = LDDivider()

      public var model: Model

      override func constructSubviewHierarchy() {
            addAutoLayoutSubviews([
                  itemStackView,
                  dividerView
            ])
      }

      override func constructSubviewLayoutConstraints() {
            NSLayoutConstraint.activate(
                  itemStackView.constraints(pinningTo: self,
                                edges: [.horizontal],
                                insets: .init(LDSpacing.space16)),
                  itemStackView.topAnchor.constraint(equalTo: topAnchor,
                                                     constant: LDSpacing.space8),

                  dividerView.constraints(pinningTo: self,
                                          edges: [.leading, .trailing],
                                          insets: .init(LDSpacing.space16)),
                  dividerView.topAnchor.constraint(equalTo: itemStackView.bottomAnchor,
                                                   constant: LDSpacing.space16),
                  dividerView.bottomAnchor.constraint(equalTo: bottomAnchor,
                                                      constant: -LDSpacing.space8)
            )
      }
}
```

…so on so forth.



```swift
protocol AddressAPI {
      //…
      func fetchAddresses() -> AnyPublisher<Address, Error>
      //…
}
```


API users can then fetch addresses and pass them to the views within their own Coordinator/Data Source

```swift

addressAPI.fetchAddresses()
.sink { [weak viewController] viewController?.model = $0 }
.store(in: &tasks)

````

> Note: This is meant as a suggestive example. Implementation may differ slightly.
