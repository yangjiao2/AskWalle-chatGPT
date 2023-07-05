---
title: Testing - Test Hooks
navTitle: Test Hooks
---

# Test Hooks

Test Hooks is a common pattern used by many teams to expose private functionality to tests for validation.

Example:

Consider the following simple example

```
class MyClass {
    private var isValid: Bool = false
    
    func validate() {
        isValid = ...
    }
}
```

Now to test that something is considered valid, you can do the following:

```
#if DEBUG
extension MyClass {
    var testHooks: TestHooks {
        .init(target: self)
    }

    struct TestHooks {
        let target: MyClass
        
        var isValid: Bool {
            target.isValid
        }
    }
}
#endif
```

Then in the tests

```
    ...
    
    func testIsValid() {
        let obj = MyClass()
        obj.validate()
        
        XCTAssertTrue(obj.testHooks.isValid)
    }
```

Note that the `TestHooks` is a struct in this example, but if you need mutation functionality from this construct you should use class.
