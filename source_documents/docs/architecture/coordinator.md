---
title: Architecture - Coordinator
navTitle: Coordinator
---

# Coordinator

- [What Is A "Coordinator"?](#what-is-a-coordinator)
- [Using Coordinators](#using-coordinators)
	- [How to use start and finish](#how-to-use-start-finish-and-onfinish)
	- [Coordinator results](#coordinator-results)
	- [Child coordinators](#child-coordinators)
	- [Managing a shared navigation controller](#managing-a-shared-navigation-controller)
- [Using Coordinator with MVVM (MVVM+C)](#using-coordinator-with-mvvm-mvvm+c)
	- [Example](#example)
    
## What Is A "Coordinator"?

A `Coordinator` is a class that can be used to wrap flows or any other potentially reusable business logic.  Their primary use in the `Walmart App` is to help us conform to the [MVVM+C](mvvmc.md) architectural pattern by managing `UIViewController`s, their navigation, their datasources, their view models, and other external interfaces.

Instances of `Coordinator` provide public lifecycle calls `start()` and `finish(_ result:)` to kick off and spin down their internally managed behaviors.  The public closure `onFinish` is provided to externally configure `finish(_ result:)`, providing flexibility in handling its result.  Note that the result of `finish(_ result:)` is a generic type which is specified in the declaration of a given subclass.

## Using Coordinators

### How to use start, finish, and onFinish

The coordinator provides the abilities to start, finish, and respond to a finished coordinator. You should use `start` when you are ready for the actions of a coordinator to take place (actions can be showing UI, performing business logic, firing a network request, etc). You should use `finish` when you a coordinator has completed its actions (typically from within the `Coordinator` instance itself). You can then use `onFinish` to handle responding to a finished `Coordinator` instance.

### Coordinator results

The result that gets passed in the `onFinish` block on task completion is configurable via generics. This can range from enums that help you determine what state it ended in to a `Never` which would indicate that finish should never actually be called.

### Child coordinators

A coordinator has built-in hooks for child coordinator management. You might choose to break a series of actions into multiple coordinators instead of one larger one. This can be especially helpful when a flow is branched.

You can deal with additions or removals (e.g. in tests for verification) via the `childDelegate` property (of type `CoordinatorChildDelegate`).

```swift
class ParentCoordinator: Coordinator<Result> {
    override func start() {
        ...
    }
    
    // triggered from view controller action when necessary
    // this can help split up a parent flow into multiple coordinators and allow for
    // optionality of parts of the flow
    private func startOptionalChildCoordinator() {
        let coordinator = ChildCoordinator()
        addChild(coordinator: coordinator)
        
        coordinator.onFinish = { (result) in
            // are we finished because this child coordinator is finished?
            // do we need to do something else when this child finished inside this parent still?
        }
        
        coordinator.start()
    }
}

class ChildCoordinator: Coordinator<Result> {

}
```

### Managing a shared navigation controller

When a `GlassNavigationController` is used, you can set your coordinator up to respond to it like so:

```swift
...

func start() {
    ...
    
    navigationController.popPublisher(for: viewController)
                        .sink { [weak self] (_) in self?.finish(.cancelled) }
                        .store(in: &tasks)
    
    ...
}

...
```

This allows your coordinator to appropriately translate a navigation controller pop as a coordinator finish result.

## Using Coordinator with MVVM (MVVM+C)

The Coordinator in MVVM has the role of handling flows (or subsets of flows). It invokes network calls, handles actions from UI interaction (passed up from views), and providing view models to the coordinated views/view controllers.

### Example

Given a view controller

```swift
protocol MyViewControllerDelegate: AnyObject {
    func cancelPressed()
    func finishPressed()
}

class MyViewController: UIViewController {
    ...
    
    
    weak var delegate: MyViewControllerDelegate?
    
    ...
}
```

You can have the following coordinator:

```swift
enum MyCoordinatorResult {
    case cancelled
    case finished
}

class MyCoordinator: Coordinator<MyCoordinatorResult>, MyViewControllerDelegate {
    private let navigationController: GlassNavigationController
    private let viewController: MyViewController
    
    private var tasks = Set<AnyCancellable>()
    
    init(navigationController: GlassNavigationController) {
        self.navigationController = navigationController
        viewController = MyViewController()
        
        super.init()
        
        viewController.delegate = self
    }
    
    override func start() {
        navigationController.pushViewController(viewController, animated: true)
        
        navigationController.popPublisher(for: viewController)
                            .sink { [weak self] (_) in self?.finish(.cancelled) }
                            .store(in: &tasks)
                            
        ...
        // this is where you'd do things like:
        // - make network requests
        // - connect view models to view controller
    }
    
    // MARK: - MyViewControllerDelegate
    
    func cancelPressed() {
        finish(.cancelled)
    }
    
    func finishPressed() {
        finish(.finished)
    }
}
```
