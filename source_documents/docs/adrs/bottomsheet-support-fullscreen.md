# Add support for full screen bottomsheet

- Authors: Hemanth Kumar
- Decision: Option 2 with UIPresentationController


# Context

For stand alone store mode, the customer is not able to properly expand into full-screen view from the resting state version of the bottom sheet on all devices (ex. below experience noted in iPhone 13 Pro)

Currently, height of bottom sheet (i.e. full screen) is hard coded, therefore request is to have dynamic height for bottom sheet enabled, in order for full screen view of stand alone store mode (and future use cases) to properly adjust to the device screen size.


# Goal

Provide teams with a recommended way that enables every team to utilize the bottomsheet full screen mode.
There are several factors that include as technical tasks:

- The sheet should extend to the top of the screen, behind the Status Bar / Safe Area.
- The dragger should fade away.
- All other contents (including the close button) should align below the Status Bar / Safe Area
- Users need not be able to drag to dismiss once the sheet is full screen.


# Options

## Option 1: UISheetPresentationController

Rewrite the BottomSheet using apple default sheet presentation controller and customizing the transitions(https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller)

### Pros
- Using apple's default presentation logics

### Cons
- This does require iOS 15+. 
- By default, `UISheetPresentationController` doesn't provide the full screen, we need to design the scrolling logic to support.
- It's only valid for using its default presentation styles like pageSheet, formSheet, overFullScreen, popover etc., for customization it doesn't bring any values.

## Option 2: UIPresentationController

Rewrite the BottomSheet using apple presentation controller and customising the transitions (https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller)

#### Example:

BottomsheetPresentationController that holds the logic of bottomsheet presentation
```swift
final class BottomSheetPresentationController: UIPresentationController {

     private let presentedYOffset: CGFloat = 150
     private lazy var dimmingView: UIView! = {
         guard let container = containerView else { return nil }
         let view = UIView(frame: container.bounds)
         view.backgroundColor = UIColor.black.withAlphaComponent(0.75)
         view.addGestureRecognizer(
             UITapGestureRecognizer(target: self, action: #selector(didTap(tap:)))
         )
         return view
     }()

     override init(presentedViewController: UIViewController, presenting presentingViewController: UIViewController?) {
         super.init(presentedViewController: presentedViewController, presenting: presentingViewController)

         presentedViewController.view.addGestureRecognizer(
             UIPanGestureRecognizer(target: self, action: #selector(didPan(pan:)))
         )
     }

     @objc func didPan(pan: UIPanGestureRecognizer) {
         guard let view = pan.view, let superView = view.superview,
               let presented = presentedView, let container = containerView else { return }

         let location = pan.translation(in: superView)
         switch pan.state {
         case .began:
             presented.frame.size.height = container.frame.height
         case .changed:
             presented.frame.origin.y = location.y + presentedYOffset
         case .ended:
             switch presented.frame.origin.y {
             case 80...container.frame.height:
                 changeScale(to: self.frameOfPresentedViewInContainerView)
             default:
                 changeScale(to: UIScreen.main.bounds)
             }
         default:
             break
         }
     }

     @objc func didTap(tap: UITapGestureRecognizer) {
         presentedViewController.dismiss(animated: true, completion: nil)
     }

     private func changeScale(to frame: CGRect) {
         guard let presented = presentedView else { return }

         UIView.animate(withDuration: 0.8, delay: 0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0.5, options: .curveEaseInOut, animations: {
             presented.frame = frame

         }, completion: nil)
     }

     override var frameOfPresentedViewInContainerView: CGRect {
         guard let container = containerView else { return .zero }

         return CGRect(x: 0, y: self.presentedYOffset, width: container.bounds.width, height: container.bounds.height - self.presentedYOffset)
     }

     override func presentationTransitionWillBegin() {
         guard let container = containerView,
               let coordinator = presentingViewController.transitionCoordinator else { return }

         dimmingView.alpha = 0
         container.addSubview(dimmingView)
         dimmingView.addSubview(presentedViewController.view)
         coordinator.animate(alongsideTransition: { [weak self] context in
             guard let `self` = self else { return }

             self.dimmingView.alpha = 1
         }, completion: nil)
     }

     override func dismissalTransitionWillBegin() {
         guard let coordinator = presentingViewController.transitionCoordinator else { return }

         coordinator.animate(alongsideTransition: { [weak self] (context) -> Void in
             guard let `self` = self else { return }

             self.dimmingView.alpha = 0
         }, completion: nil)
     }

     override func dismissalTransitionDidEnd(_ completed: Bool) {
         if completed {
             dimmingView.removeFromSuperview()
         }
     }
 }
```

Presentation Transition delegate
```swift
final class BottomSheetTransitioningDelegate: NSObject, UIViewControllerTransitioningDelegate {

     // MARK: - UIViewControllerTransitioningDelegate
     func presentationController(forPresented presented: UIViewController, presenting: UIViewController?, source: UIViewController) -> UIPresentationController? {
         return BottomSheetPresentationController(presentedViewController: presented, presenting: presenting)
     }
 }
```

Presenting a view controller
```swift
@objc private func presentFullBottomSheet() {
    let viewController = UIViewController()
    viewController.view.backgroundColor = .cyan
    let transitioningDelegate = BottomSheetTransitioningDelegate()
    viewController.modalPresentationStyle = .custom
    viewController.transitioningDelegate = transitioningDelegate
    self.present(viewController, animated: true)
}
```

### Pros
- Satisfy with current supported iOS version of the app
- We can reuse the existing logic for maintaining the current requirements(3 tier)

### Cons
- This is almost similar to the existing component apart from the transition delegates

## Option 3: Use current Bottomsheet

The existing Bottomsheet already supports the full screen, just we need to pass the proper height with `tiersType` and the `maxBottomSheetHeight` when starting the bottomsheet, and add a missing topAnchor of the bottomsheet container

```
self.topAnchor.constraint(greaterThanOrEqualTo: containerView.topAnchor).isActive = true
```

### Pros
- Existing logic, just need minor fixes, Code already in place

### Cons
- We might need to re-write the component as this was not maintainable 
- Lot of hardcoding in bottomsheet

## Decision Outcome
Option 2 with UIPresentationController

# Reference

https://developer.apple.com/documentation/uikit/uisheetpresentationcontroller
https://developer.apple.com/documentation/uikit/uipresentationcontroller
