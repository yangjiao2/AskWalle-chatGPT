---
title: Variant View
navTitle: Variant View
---


#  VariantView

## Description:

This is a subview shown in the GlassProductTile. It displays the variant options. It typically as an array of colors. It can also be used as a label for brevities sake.

### Model Setup

```swift
    struct GlassProductTile.VariantModel {
        public enum Style {
            case swatch(GlassProductTile.GlassSwatch.Model)
            case label(String?)
        }

        public enum Alignment {
            case center, leading

            var containerAlignment: UIStackView.Alignment {
                switch self {
                case .center: return .center
                case .leading: return .leading
                }
            }
        }

        public var alignment: Alignment
        public var style: Style
        public var variantText: String?

        var isSwatchHidden: Bool {
            if case .swatch = style { return false }
            return true
        }

        var isLabelHidden: Bool {
            if case .label = style { return false }
            return true
        }
    }
```

### Composition

    - First Example

```swift
import DiscoveryUIShared

func addVariantViewExampleOne() {
    let colors: [LDColor] = .greens
    let swatchModel: GlassSwatch.Model = .init(swatchConfigurations: colors.map { view in { $0.backgroundColor = view.uiColor }},
                                               size: .small)
    let variantView = GlassProductTile.VariantView(model: .init(style: .swatch(swatchModel),
                                                   alignment: .center))
    addAutoLayoutSubview(variantView)
}
```

<img src="images/variant/variantOne.png"/>

    - Second Example

```swift
func addVariantViewExampleTwo() {
    let text = "7 Options"
    let variantView = GlassProductTile.VariantView(model: .init(style: .label(text),
                                                   alignment: .center))
    addAutoLayoutSubview(variantView)
}
```

<img src="images/variant/variantTwo.png"/>

    - Third Example

```swift
func addVariantViewExampleThree() {
    let colors: [LDColor] = [
        .orange05,
        .orange10,
        .orange20,
        .orange30,
        .orange40,
        .orange50,
        .orange60,
        .orange70,
        .orange80]
    let swatchModel: GlassSwatch.Model = .init(swatchConfigurations: colors.map { view in { $0.backgroundColor = view.uiColor } },
                                               size: .small)
    let variantView = GlassProductTile.VariantView(model: .init(style: .swatch(swatchModel),
                                                   alignment: .center))
    addAutoLayoutSubview(variantView)
}
```

<img src="images/variant/variantThree.png"/>
