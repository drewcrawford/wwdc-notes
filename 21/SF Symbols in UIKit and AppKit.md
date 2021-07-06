#sfsymbols

# Color rendering modes
Layered symbols, one color per layer
* Hierarchical
* Palette
* Muticolor
Layer hierarchy(primary, secondary, tertiary)
Uses template rendering mode

[[Introducing SF Symbols - 19]]

## Monochrome symbols.
```swift
// Play

let playImage = UIImage(systemName: "play")

playImageView.image = playImage 
playImageView.tintColor = .systemBlue
```
Pre ios-15/macos11: only one color mode
use tint color
that's it

## Hierarchical
```swift
// Device image

var image = NSImage(systemSymbolName: "ipad.landscape",
                    accessibilityDescription: "iPad")

let config = NSImage.SymbolConfiguration(hierarchicalColor: .label)

deviceView.image = image
deviceView.symbolConfiguration = config
```

Color is used for "primary layer".  Secondary/tertiary layers have reduced opacity.

* Color scheme based on 1 color
* Other colors derived by reducing opacity
* Uses hierarchy level to define color

## Palette
Plain images
```swift
// Initialize button configuration

let speakerConfig = UIButtonConfiguration.plain
speakerConfig.image = UIImage(systemName: "speaker.wave.2")

let callConfig = UIButtonConfiguration.plain
callConfig.image = UIImage(systemName: "phone")

let deleteConfig = UIButtonConfiguration.plain
deleteConfig.image = UIImage(systemName: "trash")
```

Image pairings instead.

```swift
// Button container view

actionsView.imageVariant = .circle.fill
```

Specify colors in a palette
```swift
// Speaker button color configuration

let config = UIImage.SymbolConfiguration(paletteColors: [.tintColor, .systemGray2])

speakerConfig.preferredSymbolConfigurationForImage = config
speakerButton.configuration = speakerConfig
```

```swift
// Colors matter!

let config = UIImage.SymbolConfiguration(paletteColors: [.tintColor, .systemGray2])

let config = UIImage.SymbolConfiguration(paletteColors: [.white, .tintColor])

let config = UIImage.SymbolConfiguration(paletteColors: [.white, .systemRed])
```

### Tint color (UIKit only)
* Resolves to tint color of view
* Can be used everywhere you use a color
* Same rules/caveats as other dynamic colors
* AppKit equivalent: `.controlAccentColor`

[[Implementing dark mode on iOS]]

## Palette color symbols
* Primary and tertiary
* 2 hierarchy layers
* One for each layer

Can specify 2 colors and will be applied to "available layers" for related symbols for a mixed number of layers.

Configurable set of colors
Uses hierarchy level to apply color
Convenience for 2 layer symbols

## Multicolor
```swift
// configure table view cell

let image = UIImage(systemName: category.iconName)

let config = UIImage.SymbolConfiguration.preferringMultiColor

let tintColor = category.colorForIcon

cell.imageView.image = image
cell.imageView.preferredSymbolConfiguration = config
cell.imageView.tintColor = tintColor
```

Which symbols support multicolor?  SF Symbols App.  Use inspector to see the color rendering mode.

## Multicolor preference
* Indicates preference
* Fallbakc configuration possible (for certain symbols?)
* No explicit fallback => monochrome

* Fixed set of colors
* Adopts tint color automatically

## Color configurations in Xcode

IB support.  

[[Build interfaces with style]]

# Combining configurations
2 configurations.  Point size, and color config.  Only apply 1, how do we comebine them?

```swift
// Combined configuration

let image = UIImage(systemImage: "ipad.and.iphone")
headerView.image = image

let fontConfig = UIImage.SymbolConfiguration(pointSize: 60, scale: .large)
let colorConfig = UIImage.SymbolConfiguration(hierarchicalColor: .systemBlue)
let config = fontConfig.applying(colorConfig)

headerView.preferredSymbolConfiguration = config
```

Not limited to color config only, can also use this with other configurations.


# Colors in attributed strings

Combine symbols with text.  

```swift
// Hotel amenities

let amenitiesString = NSMutableAttributedString(...)

if (room.amenities.contains(.tv)) {
    let config = UIImage.SymbolConfiguration(
                         hierarchicalColor: .systemGreen)
    let tvImage = UIImage(systemImage: "tv", 
                          withConfiguration: config)

    let attachment = NSTextAttachment(image: tvImage)
    let attachmentString = NSAttributedString(attachment: 
                                               attachment)
    let tvString = attachmentString.mutableCopy()
    tvString.append(NSAttributedString(" TV, ")

    amenitiesString.append(tvString)
}
```
```swift
// hotel amenities

let amenitiesLabel = UILabel()

amenitiesLabel.textColor = .systemGreen
amenitiesLabel.font = UIFont.systemFont(ofSize: 25)

amenitiesLabel.attributedString = amenitiesString
```

# Wrap up
* 3 new color configurations
* New `.tintColor` color
* Combining configurations
* Applying configurations to attributed strings

[[What's new in SF symbols]]
[[SF Symbols in SwiftUI]]

