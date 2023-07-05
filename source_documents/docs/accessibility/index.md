# Accessibility Overview

Walmart is committed to making our mobile apps accessible to all users. Every feature in the app must, at a minimum:

- [x] Provide a clear [title for the screen](https://developer.apple.com/documentation/uikit/uinavigationitem/1624965-title)
- [x] Ensure valid [traits](https://developer.apple.com/documentation/objectivec/nsobject/uiaccessibility/accessibility_traits) are provided
- [x] Provide informative [accessibility labels](https://developer.apple.com/documentation/uikit/uicontrol#1658529) for meaningful images
- [x] Make section headers explicit and not just visual
- [x] Announce dynamic content updates
- [x] Ensure content order and grouping is coherent and not duplicated
- [x] Adhere to [color and contrast guidelines](https://developer.apple.com/design/human-interface-guidelines/accessibility/overview/color-and-contrast/)

For more detail on this checklist, see [Accessibility Testing Checklist - iOS Native App](https://confluence.walmart.com/display/CEACCESS/Accessibility+Testing+Checklist+-+iOS+Native+App)

# VoiceOver

[VoiceOver](https://developer.apple.com/documentation/uikit/accessibility_for_ios_and_tvos/supporting_voiceover_in_your_app) is Apple's built-in screen reader for iOS devices. This tool is the primary means by which we empower users with disabilities to use our apps, and it is essential to ensure our user experience is just as good for them as it is for all other users. Testing VoiceOver requires [running the app on your device](https://gecgithub01.walmart.com/walmart-ios/glass-app#running-on-device).

- [VoiceOver best practices](voiceover.md)
- [Color and contrast](contrast.md)

# Testing

Although UI testing is not strictly related to ADA compliance, the topics are interrelated enough that it warrants mention here. One beneficial side effect of writing an accessible app is that it eases automated testing. UI testing tools like XCUITest "see" the app in much the same way VoiceOver users do.

Conversely, the interrelatedness of these two topics also means a developer must be careful not to negatively impact accessibility when writing tests. Notably:

**Accessibility _labels_ should not be customized for the purposes of UI testing. Instead, use _accessibility identifiers_**

The Glass iOS platform provides

- [Accessibility testing](testing.md)

## See also

- [Accessibility best practices](https://developer.apple.com/design/human-interface-guidelines/accessibility/overview/best-practices/)
- [Accessibility API documentation](https://developer.apple.com/documentation/objectivec/nsobject/uiaccessibility)
- [UI Testing in iOS](https://eon.codes/blog/2019/05/02/UI-testing-in-iOS/)
- [Xcode UI Testing Cheat Sheet](https://www.hackingwithswift.com/articles/148/xcode-ui-testing-cheat-sheet)
