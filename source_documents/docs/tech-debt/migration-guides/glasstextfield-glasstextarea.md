### GlassTextField and GlassTextView Migration Guide

GlassTextField and GlassTextView are both very similar so only one migration is given. Please take special care to read about the validation strategy as that is really the only non-trivial migration element.



### Text Field Properties

| **Glass**            | **Living Design**                     | **Example**                                                  |
| -------------------- | ------------------------------------- | ------------------------------------------------------------ |
| text                 | text                                  | textField.text = "username"                                  |
| titleText            | headerLabelText model.headerLabelText | textField.headerLabelText = "Username" textField.model = LDTextField.Model(headerLabelText: "Username") |
| placeholderText      | model.placeholderText                 | textField.model = LDTextField.Model(placeholderText: "Enter your username") |
| leftIcon: GlassIcon? | leadingIcon: LDImage?                 | textField.leadingIcon = LDImage(named: "arrow") textField.model = LDTextField.Model(leadingIcon: LDImage(named: "arrow")) |
| rightIconButton      | trailingContentSlotView               | textField.trailingContentSlotView = UIButton(type: .system)  |
| no clear button      | clearButtonMode                       | textField.clearButtonMode = .whileEditing                    |
| hintText             | helperLabelText                       | textField.headerLabelText = "Enter username or email address." textField.model = LDTextField.Model(headerLabelText: "Enter username or email address.") |
| hintText = nil       | hideHelperView                        | textField.hideHelperView = true                              |
| isSecureEntry        | isSecureTextEntry                     | textField.isSecureTextEntry = true                           |

### Text Field Model

| **Glass.Model**                         | **LivingDesign.Model**                               | **Example**                                                  |
| --------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| leftIcon: GlassIcon?                    | leadingIcon: LDImage?                                | textField.leadingIcon = LDImage(named: "arrow") textField.model = LDTextField.Model(leadingIcon: LDImage(named: "arrow")) |
| rightIcon: GlassIcon?                   | trailingContentSlotView: UIView?                     | textField.trailingContentSlotView = UIButton(type: .system) textField.model = LDTextField.Model(trailingContentSlotView: UIButton(type: .system)) |
| rightIconAction: (() -\> Void)?         | build your own action using trailingContentSlotView  | let button = UIButton(type: .system) button.addTarget(self, action: \#selector(buttonPressed:), for: .primaryActionTriggered) textField.trailingContentSlowView = button |
| placeholderText: String?                | placeholderText: String?                             | textField.model = LDTextField.Model(placeholderText: "Enter your username") |
| text: String?                           | text: String?                                        | textField.text = "h.g.wells@walmart.com" textField.model = LDTextField.Model(text: "h.g.wells@walmart.com") |
| titleText: String?                      | headerLabelText: String                              | textField.headerLabelText = "Email or Username" textField.model = LDTextField.Model(headerLabelText: "Email or Username") |
| hintText: String?                       | helperLabelText: String                              | textField.helperLabelText = "Please enter email or username" textField.model = LDTextField.Model(helperLabelText: "Please enter email or username") |
| validationBlock                         | validationStrategy                                   | Please see below for examples...                             |
| shouldAnimateTitleLabel                 | not available                                        |                                                              |
| shouldValidateEachTextChange            | validationStrategy. shouldValidateAfterEachCharacter | Please see below for examples...                             |
| shouldShowSuccessState: Bool            | not available                                        | the viewState can be adjusted according to the requirements of the caller. They are error, disabled, readOnly, regular, and focused. |
| isSecureEntry: Bool?                    | isSecureTextEntry: Bool?                             | textField.isSecureTextEntry = true                           |
| secureEntryStyle: SecureTextEntryStyle? | not available                                        | *this field was only necessary for the GlassTextField's multiple ways to override secure text entry.* |
| autoClearValidation: Bool?              | validationStrategy. shouldValidateAfterEachCharacter | Please see below for examples...                             |
| next: UIResponder?                      | not available on model, use properties               | textField.nextTextField = passwordField                      |
| previous: UIResponder?                  | not available on model, use properties               | textField.previousTextField = nil                            |
| no size option                          | small vs large                                       | textField.model = LDTextField.Model(size: .large)            |
| no view state                           | viewState                                            | textField.viewState = .error(errorText: "Please input a valid username.") textField.model = LDTextField.Model(viewState: .error(errorText: "Please input a valid username.")) |
| no custom accessibility                 | customAccessibilityText: String?                     | textField.customAccessibilityText = "Please enter your username." |

### Validation Strategy

In order to make LDTextField validate text, show error messages, and give the user feedback you may need to implement a TextValidationStrategy.

The public protocol:

```swift
public protocol TextValidationStrategy {
  func validate(text: String?) -\> Bool
  var errorText: String { get }
  var shouldValidateAfterEachCharacter: Bool { get }
}
```

In GlassTextField and GlassTextArea, this was done using a validationBlock. With TextValidationStrategy you are still able to perform the same basic functions, but with a little more explicit api.

### Validation Strategy Examples

In the following example you can see a single validation strategy applied using a regex validator with an injected errorMessage on the constructor. This is useful when the contents have a single error state.

```swift
public struct EmailValidationStrategy: TextValidationStrategy {
  static let emailRegex = "\^([a-zA-Z0-9_\\-\\.\\+]+)@([a-zA-Z0-9_\\-\\.]+)\\.([a-zA-Z]{2,63})\$"

  public func validate(text: String?) -\> Bool {
  	guard let email = text, !email.isEmpty else {
  	return false
  }

  guard let regex = try? NSRegularExpression(pattern: Self.emailRegex) else {
	  return true
  }

  let results = regex.matches(in: email, range: NSRange(email.startIndex..., in: email))
    return !results.isEmpty
  }

  public var shouldValidateAfterEachCharacter: Bool
  public var errorText: String

  public init(errorMessage: String, shouldValidateAfterEachCharacter: Bool = false) {
    errorText = errorMessage
    self.shouldValidateAfterEachCharacter = shouldValidateAfterEachCharacter
  }
}
```

The following example shows how a "required" style error message can be displayed when no text is entered.

```swift
struct CustomNotEmptyValidation: TextValidationStrategy {
  func validate(text: String?) -\> Bool {
	  !(text ?? "").isEmpty
  }

  var errorText = "This field cannot be empty"
  var shouldValidateAfterEachCharacter = true
}
```

The following example shows how you can support multiple different error messages for the different error conditions and express a hierarchy of which error text should be shown first if your error conditions are not disjoint sets.

```swift
class CvvValidation: TextValidationStrategy {
  func validate(text: String?) -\> Bool {
    if text?.isEmpty == true {
      // update error text
      errorText = "This field cannot be empty"
      return false
    }

    if let text = text, text.contains(where: { !\$0.isNumber }) {
      errorText = "This field only accepts digits"
      return false
    }

    if (text?.count ?? 00) \> 3 {
      // update error text
      errorText = "This field cannot have more than 3 characters"
      return false
    }

    return true
  }

  var errorText = ""
  var shouldValidateAfterEachCharacter = true

  init() {}
}
```

