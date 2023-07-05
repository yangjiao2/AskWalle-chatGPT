# Contributing to Cart & Checkout on iOS/Android

:wave: Welcome! We’re happy to have you contributing to our code base. Please make sure you have read the general Contribution Guidelines ([iOS guidelines](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/CONTRIBUTING.md) / [Android guidelines](https://gecgithub01.walmart.com/Walmart-Android/walmart-glass/blob/development/docs/CONTRIBUTING.md)). Beyond that, there are a few things to keep in mind when working on Cart & Checkout (CXO) specifically.

## Team agreement

Whether you’re part of CXO or not, it’s important to have a look through [our team agreement](https://confluence.walmart.com/display/~t0s00lb/CXO+Mobile+Team+Agreements). This will familiarize you with the way we work.

### Some questions to ask yourself...

> Should this feature be behind a CCM flag?

> Should the PR be broken into smaller PR's?

> Should the feature be contained in it's own plugin?

> Does shared API cause any issues with existing features?

## Error handling

Error handling is particularly tricky and important in CXO. Because we’re dealing with the actual transaction, the number of things that can go wrong is broad, and the impact is high. Being the critical point in the customer journey, small miss in error handling can literally translate to conversion losses in the tens to hundreds of millions of dollars.  A few examples:

- Payment declined
- Items go out of stock after viewing cart but before reviewing the order
- User changes their selected store and the time slot they reserved is no longer available

There are many, many other scenarios. Some questions to consider:

 - [ ] Does your change use any new queries?
 - [ ] Are there failure scenarios in your change?
 - [ ] Are **_all_** new error cases documented?
 - [ ] Has your product partner verified error cases and handling with the CXO product leads?
 - [ ] Have you written unit tests covering every error scenario?
 - [ ] Have you added Graphu fixtures (see below) for all service changes?

Here is a list of all existing known [CXO GraphQL Errors](https://confluence.walmart.com/display/PG/OL+Mock+Variants+ce-cartxo)

## Graphu

CXO makes use of the in-house [Graphu mocking tool](https://gecgithub01.walmart.com/glass/mock-cxo) to ease testing of unusual scenarios. By running the CXO feature application against mock services, you can quickly verify functionality under a wide variety of scenarios.

Should you need a query response, add a fixture to Graphu according to these [instructions](https://gecgithub01.walmart.com/glass/mock-cxo#how-to-add-a-fixture)

Should you need a mutation response, these [instructions](https://gecgithub01.walmart.com/glass/mock-cxo#how-to-add-new-query-resolver-ex-cart) are helpful, just add to the `mutations.js` file instead of `queries.js`.

### When adding fixtures:

1. Validate your fixture against the schema

2. Use a descriptive name.

_Good_

> Unscheduled Pickup with all Items In Stock.json

_Bad_

  > instock.json

## Testing

There should be excellent examples for tests of all types. Unit tests should be first concentrated on the smallest possible unit first. Each layer of logic has different responsibilities. Testing can also reveal unexpected dependencies, it is important for checkout to have these boundaries clearly defined as predictable behavior is the top priority.

### GraphQL

Local use of GraphQL objects defined in the Apollo generated code is discouraged. There is a thin layer of transformation that happens after an API call is made. These methods make heavy use of the Orchestration Client. Please ensure that any additional queries and mutations are adequately tested.

Orchestration Client fixtures are located at {Source Root}/Plugins/CXO/CXOTests/Fixtures/OrchestrationClientTests/**. 

	func testFetchCartIgnoresDuplicateItemIDs() throws {
        // Given
        client = OrchestrationClient(baseURL: URL(fixturesFor: self), networking: .fixtures(),
        							 analytics: MockCartLoadPerformanceTracker())
        
        // When
        let data = try waitForValue(client.fetchCart(for: CartRequest(customerID: "customer-1",
        								storeLocation: .make())))
        // Then
        XCTAssertEqual(
        	data.cart.items.map({ $0.id }),
        	["11111111-1111-1111-1111-111111111111"]
        )
	}

The fixture json must be named `response.json` with the fixture folder name corresponding to the name of the test. The fixture folder for the above test should be named: `testFetchCartIgnoresDuplicateItemIDs`. There are plenty of examples of this setup.

### DataSource

All API calls are delegated inside datasource objects. The methods for requesting and returning data should be tested. If there are logical transformations here, they should also be tested.

`ContractDataSource` is a good example for how to test a datasource. 

```
func testCreateSuccessUpdatesData() throws {
    let mockService = MockCheckoutService()
    let dataSource = ContractDataSource(service: mockService)
    
    dataSource.createPurchaseContract(cartID: "cart-1")
    XCTAssertTrue(try waitForValue(dataSource.isLoadingIsTrue))
    
    mockService.createPurchaseContractMethod.succeed(response: .init(contract: .make(id: "contract-1")))
    
    XCTAssertNotNil(try waitForValue(dataSource.createCurrentContractPublisher()))
    XCTAssertFalse(try waitForValue(dataSource.isLoadingIsFalse))
    
    XCTAssertEqual(dataSource.data.contract?.id, "contract-1")
}
```
    
In this example, we inject the mock checkout service. Then we check that the loading state was entered.  We should setup the mock service with the correct mock response, then assert the final state agrees with the injected response.

### Coordinator

`CheckoutCoordinator` is a very complex data structure managing most of the behavior for the checkout process. There are dozens of tests ranging from testing publishers to the creation and destruction of child coordinators. Any additions to this coordinator or others should be taken seriously and well tested.

Coordinators can include UIKit objects like UIViewControllers. This means there are a few types of tests and test utilities that might come into play.

Testing the functions declared on the Coordinator protocol directly are a good place to start. For this you'll only need injectable mocks and an instance of the coordinator. Then there are some helper methods in TestUtilities (i.e. `waitForBottomSheet`) for validating that the view stack has changed in the intended way.

```
func testStart() throws {
     // Given
     _ = MockWindow(rootViewController: navigationController)
     let coordinator = CheckoutUnavailableItemsCoordinator(
     	errorLineItems: [],
     	requestType: .createPurchaseContract,
     	contractDataSource: ContractDataSource(service: checkoutService),
     	cartDataSource: .init(service: MockCartService(),
                              cart: Cart.make(itemCount: 3),
     						  analytics: MockCartLoadPerformanceTracker(),
                              ccmContainer: .make()
        ),
     	cartSFLDataSourceAggregator: RecordingMockCartSFLDataSourceAggregator.makeDisabled(),
     	navigationController: navigationController
     )
     
     // When
     coordinator.start()
     
     // Then
     try waitForBottomSheet(on: navigationController, type: UnavailableItemsViewController.self)
}
```

### Testing UI (not UI tests)

As you can see from the above example, the wait at the bottom depends on the `MockWindow` at the top. In order to proceed through the UIKit lifecycle methods, the views and view controllers must be added to a window. Since most objects inside of a view controller should be private to promote good encapsulation and information hiding practices, please make heavy use of the "test hooks" convention to ensure that model to view behavior is enforced correctly in your tests.

```
// MARK: - TestHooks

#if DEBUG
extension ReserveSlotContainerViewController {
    struct TestHooks {
        let target: ReserveSlotContainerViewController

        var loadingViewController: GlassLoadingViewController { target.loadingViewController }
        var fulfillmentControlView: FulfillmentControlView { target.fulfillmentControlView }
        var reserveSlotViewController: ReserveSlotViewController { target.reserveSlotViewController }
        var reserveSlotView: ReserveSlotView { target.reserveSlotView }
        var reserveSlotViewTopConstraint: NSLayoutConstraint? { target.reserveSlotViewTopConstraint }
        var reserveSlotViewBottomConstraint: NSLayoutConstraint? { target.reserveSlotViewBottomConstraint }

        func close() { target.close() }
    }

    var testHooks: TestHooks {
        TestHooks(target: self)
    }
}
#endif
```

### Testing Presentation

No business logic should exist inside the view layer (view controller or view classes). If you do not see how it can be avoided, please pair with someone on the checkout team to try and come up with an alternative. The best locations for this type of work depends on the work being done:

* transforming GraphQL to domain objects should be in the service layer (e.g. OrchestrationClient)
* workflow logic should be done in coordinator layer
* transforming domain objects to something usable in the UI should be done in the presentation classes

This allows for custom initializers inside the presentation objects that accept a wide range of parameters. Anything in the coordinator is fair game. The important thing to remember is the UI is built declaratively, so all objects inside the model of the view should be idempotent unless a refresh of the view is intended. The custom initializers are intended to say in plain english, given these widely varying circumstances, the view model should be this exactly every time. There should be sufficient tests to ensure that for all cases, the intended view model is correct. 

### UI/Functional Testing

Hopefully you're familiar with the patterns implemented in Glass, if not, please watch any videos related to glass plugin architecture. 

[Walmart Architecture Conference 2020](https://confluence.walmart.com/display/GPONEW/iOS+Architecture+Pattern+Evaluation)

[Walmart MVP Test Plan](https://confluence.walmart.com/display/PG/Glass+MVP+Test+Plan)
