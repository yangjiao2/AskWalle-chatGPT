---
title: PetRxFulfillmentDialogViewController
navTitle: PetRxFulfillmentDialogViewController
---

# PetRxFulfillmentDialogViewController

## Description

The *PetRxFulfillmentDialogViewController* is used as an information disclaimer.
This presents the label of the pet medication, a notice with a brief explaination,
an estimated time of arrival and information on order updates, with a close and
confirmation button.

### Model Properties

```swift
    struct PetRxFulfillmentDialogViewController.Model {
        /// title of the Modal
        let title: String
        /// text to be displayed at the top of the spacer
        let description: String
    }
```

### Callback Properties

```swift
    /// Used when the customer swipes the dialog to dismiss and is used to track anylitics
    public var onCloseBottomSheet: ((Bool) -> Void)?
    /// Used when onAppear is called from the view life cycle
    public var onAppear: ((String) -> Void)?
    /// Used when the customer taps the got it button to track anylitics
    public var didTapGotIt: ((String) -> Void)?
    /// Used when the customer taps the close button to track anylitics
    public var didTapClose: ((String) -> Void)?
```

### Example Usage

```swift
    func didTapPetRxViewLearnMore() {
        let petRxFulfillmentController = PetRxFulfillmentDialogViewController(
          model: .init(
            title: "Prescription Label",
            description: "petRx Notice Explanation Label"
          )
        )
        petRxFulfillmentController.onAppear = { [weak self] _ in
            // Do stuff
        }
        petRxFulfillmentController.didTapGotIt = { [weak self] _ in
            // Do stuff
        }
        petRxFulfillmentController.didTapClose = { [weak self] _ in
            // Do stuff
        }
        petRxFulfillmentController.onCloseBottomSheet = { [weak self] _ in
            // Do stuff
        }
        let petRxNavigationController = BottomSheetNavigationController(rootBottomSheet: petRxFulfillmentController)
        navigationController?.present(bottomSheet: petRxNavigationController, as: .modal)
    }
```

<img src="images/petRXDialog.png" width="375"/>
