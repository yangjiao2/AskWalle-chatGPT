# ADR: Platform Theme Application System
###### Authored: 12/27/2022
###### Author(s): Drake Yundt (d0y00qn)
###### Decision: Option 1

## Introduction
As glass moves deeper and deeper into a multi market/ multi app world, we need to be able to support theme application across all UI in the Glass ecosystem from a Platform perspective. 

## Motivation
Market applications, such as MX Bodega and US B2B, require their own look and feel from a Product perspective. In order to accomplish this we need to be able to reliably theme the UI of the Glass ecosystem, while also preparing ourselves to support UI features like dark and light mode.

### Guiding Principles
- Must support gradual adoption of UI components
- Must support a certain amount of overridability
- Must support a future unified design language/ Theme Specification.
- Must support both B2B and MX Bodega's current theming needs.
- Needs to support the future addition of DarkMode

## Considered Options

### Option 1: Incorporate new Themeable protocol into the existing ViewConstructable pattern
###### - [Proof of Concept](https://gecgithub01.walmart.com/walmart-ios/glass-app/pull/36558)

#### Pros:
- Implementation of applyTheme method can be done incrementally, one UI component at a time. 
- Components can override standard theme application by providing their own implementation of applyTheme.
- Once a design language / theme specification is aligned across all orgs, the same application system can be used to apply it.
- Supports both B2B and Bodega's needs
- Darkmode can easily be supported by triggering applyTheme from the BaseView classes view via overriding the existing traitCollectionDidChange method.

#### Cons:
- Some legacy components will need to be refactored to use ViewConstructible. THIS NEEDS TO BE DONE ANY WAY!
- ViewConstructable needs to be changed to not include @objc optional functions. WE SHOULDN'T BE DOING THIS ANYWAY!

#### Example:

```swift

// MARK: Themeable
public protocol Themeable: AnyObject {
    var themeSubscription: AnyCancellable? { get set }

    func applyTheme(_ theme: WalmartTheme)
}

public extension Themeable {
    func subscribeToThemeUpdates() {
        themeSubscription = WalmartTheme
            .themePublisher
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: { [weak self] theme in
                self?.applyTheme(theme)
            })
    }
}

extension WalmartTheme {
    public private(set) static var themePublisher: PassthroughSubject<WalmartTheme, Never> = .init()

    public private(set) static var current: WalmartTheme = .default {
        didSet {
            themePublisher.send(current)
        }
    }

    public static func setTheme(_ theme: WalmartTheme) {
        current = theme
    }

    private static var `default`: WalmartTheme = .init(
        brandColors: .init(
            tabBar: .init(
                background: .gray00,
                badge: .red100,
                iconNormal: .gray100,
                iconSelected: .blue100,
                textNormal: .gray130,
                textSelected: .blue100
            ),
            navigationBar: .init(
                backgroundColor: .blue100
            ),
            buttons: .init(
                primary: .init(
                    normal: .init(
                        foreground: .gray00,
                        background: .blue100
                    ),
                    highlighted: .init(
                        foreground: .gray00,
                        background: .blue160
                    ),
                    disabled: .init(
                        foreground: .gray00,
                        background: .gray50
                    )
                ),
                secondary: .init(
                    normal: .init(
                        foreground: .gray200,
                        background: .gray00
                    ),
                    highlighted: .init(
                        foreground: .gray200,
                        background: .gray00
                    ),
                    disabled: .init(
                        foreground: .gray200,
                        background: .gray00
                    )
                ),
                destructive: .init(
                    normal: .init(
                        foreground: .gray00,
                        background: .red100
                    ),
                    highlighted: .init(
                        foreground: .gray00,
                        background: .red100
                    ),
                    disabled: .init(
                        foreground: .gray00,
                        background: .red100
                    )
                )
            )
        )
    )
}

// MARK: ViewConstructable
public protocol ViewConstructable: Themeable {
    func viewWillConstruct()
    func viewDidConstruct()
    func constructView()
    func constructSubviewHierarchy()
    func constructSubviewLayoutConstraints()
}

// MARK: - Construction
public extension ViewConstructable {
    func construct() {
        viewWillConstruct()
        constructView()
        applyTheme(.current)
        subscribeToThemeUpdates()
        constructSubviewHierarchy()
        constructSubviewLayoutConstraints()
        viewDidConstruct()
    }
}

// MARK: - Default implementations of optional methods
public extension ViewConstructable {
    func viewWillConstruct() { }
    func viewDidConstruct() { }
}
```

#### Open Question(s):
- How will we migrate from the temporary theme to the aligned theme specification coming out of the theming working group? 
  - We could namespace the themes. This would allow components to migrate to the new theme spec gradually once it is ready. 

### Option 2: Inject overrides at the Plugin level.
This option is being considered only to play devils advocate... We really should not consider it as a valid approach. That said the existing Plugin Configuration system that we use for things like base urls, could be adapted to allow market apps to inject colors for things like the navigation bar and the like. However, this all falls apart when low level UI libraries such as LivingDesign come into play as they do not support this type of configuration.

#### Pros:
- Supports gradual implementation plugin by plugin.
- Meets B2B and MX Bodega current needs.

#### Cons:
- Does not scale at all
- Requires tons of boilerplate in Plugin's APIs that should not be coupled to theme to begin with.
- Violates Glass's separation of concerns mantra.
- Darkmode would require each plugin to implement it separately, will no scale.
- Migrating to an aligned theme specification and design language would be extremely cumbersome if not impossible.

### Recommended Solution:
The recommended solution is of course [Option 1](#option-1-incorporate-new-themeable-protocol-into-the-existing-viewconstructible-pattern), it not only meets all the guiding principles set forward by this ADR but does so in a clean and robust way that will support our future theming ambitions wherever they may take us.
