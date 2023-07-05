# Weak Delegate

Delegates should be weak to avoid reference cycles.

Delegate is a common pattern on iOS used in our code as well as in Apple's frameworks. Quite often you assign `self` as a delegate for some other object you create and probably retain a strong reference to as well. This creates a retain cycle between two objects which could lead to keeping them in momery forever causing memory leak and potentially some other side effects depending at what else they do. It's possible to avoid memory leak, by breaking the retain cycle at the right time. E.g. by setting `delegate = nil` when it's no longer needed. But this is more intricacy and often is forgotten about. So to avoid this types of issues, it's better to keep delegate weak. A couple articles on the topic if you need more info:
* https://useyourloaf.com/blog/quick-guide-to-swift-delegates/
* https://medium.com/macoclock/delegate-retain-cycle-in-swift-4d9c813d0544

## Fixing the warning

To fix non-[weak_delegate](https://realm.github.io/SwiftLint/weak_delegate.html), you typically need to:

1. analyze lifecycle of the objects you are working with. Have clear understanding who retains who and for how long. A pencil and a piece of paper may be handy here.
2. change delagete to be `weak`:
```swift
weak var delegate: Delegate
```
3. you may need to do some additional changes depending on your situation
4. test your changes in the app to make sure everything works

Additionally, you may choose to add a unit test to verify that your object doesn't leak.

```swift
func testDoesNotLeak() throws {
    weak var weakController: MyViewController?

    try autoreleasepool {
        let controller = MyViewController()
        _ = MockWindow(rootViewController: controller)
        try waitForViewControllerToAppear(viewController: controller)

        weakController = controller
    }
    XCTAssertNil(weakController)
}
```
