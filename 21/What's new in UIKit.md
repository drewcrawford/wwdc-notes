#uikit 
# Productivity
## iPad multitasking demo
"Center scene multitasking feature"

```swift
// Building an "Open in New Window" action

let newSceneAction = UIWindowScene.ActivationAction({ _ in

    // Create the user activity that represents the new scene content
    let userActivity = NSUserActivity(activityType: "com.myapp.detailscene")

    // Return the activation configuration
    return UIWindowScene.ActivationConfiguration(userActivity: userActivity)

})
```
## Pointer band selection
Enabled by default for UICollectionViews that support multiselection

## Pointer accessories
Secondary shapes combined with pointer styles.  Multiple accessories can be displayed.

## Keyboard shortcuts
Adopt UIMenuBuilder

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    override func buildMenu(with builder: UIMenuBuilder) {
      
        // Use the builder to modify the main menu...
    }
}
```

* Assign categories
* Audit uses of `UIResponder.keyCommands`

More info: [[Take your iPad apps to the next level]]

## Focus system is now available in iPadOS
Arrow keys are used to move between focused items
Tab keys used to move between focused groups
[[Focus on keyboard navigation]]

## Multitouch drag/drop
Enabled on iPhone now.
[[Introducing drag and drop - 17]]
[[Drag and Drop with Collection and Table View - 17]]
[[Data Delivery with Drag and Drop - 17]]
[[Mastering Drag and Drop - 17]]
# UI refinements
## toolbar / tabbar
Updated visual appearance of `UIToolbar` and `UITabBar`
enhanced uspport for SFSymbols

Audit your code for clases where you
* set `isTranslucent` to false
* VCs that have nonstandard `edgesForExtendedLayout`

Both will cause visual issues with new appearance

Custom appearance to opt out

```swift
let appearance = UITabBarAppearance()
appearance.backgroundEffect = nil
appearance.backgroundColor = .blue

tabBar.scrollEdgeAppearance = appearance


let scrollView = ... // Content scroll view in your app
viewController.setContentScrollView(scrollView, for: .bottom)
```

It's possible UIKit won't be able to infer the scrollview for the scroll edge appearance transitions.
To specify the scrollview, see example above

## List headers
New padding, etc.
Use `.plain` style in conjunction with index bars for fast scrubbing.
`.grouped` - great choice for configuration UI or registration flows, similar to Settings app.
`.prominent` - used for sidebar lists on iPad.  Adapting a sidebar list to an inset group list in a compact size class.
e.g. alarm tab in clock app.
`.extraPominentInsetGroup` -> headers maintain hierarchy and avoid becoming lost.  Watch app's face gallery.

Use `UIListContentConfiguration` API in iOS 14.
In 14.5, we introduced `UIListSeparatorConfiguration` to provide advanced customization of separator appearance
Override system appearance on a per-row basis.

## Sheet presentations
* Hal fheight sheets
* Optionally non-modal

[[Customize and resize sheets in UIKit]]

Allowing interaction within and behind the sheet.

## UIDatePicker
Now you can simply tap the time to use the keyboard for input.  And edit time inline with keyboard.


# API enhancements
## Enhanced UIButton API
* Gray - gray background
* Tinted
* Filled - entirely opaque

Better support for dynamic type
Multiline text

`UIButton.Configuration` => More customizable and easy to update.

Popup/popdown buttons
Combo boxes on mac

```swift
// Creating a button with UIButton.Configuration

var config = UIButton.Configuration.tinted()

config.title = "Add to Cart"
config.image = UIImage(systemName: "cart.badge.plus")
config.imagePlacement = .trailing
config.buttonSize = .large
config.cornerStyle = .capsule

self.addToCartButton = UIButton(configuration: config)
```
[[Meet the UIKit button system]]

## Submenus
* New UI for UIMenu submenus
* Preserves menu hierarchy
* No API adoption required
Prior to iOS 15, we replaced the current menu entirely.

[[Meet the UIKit button system]]

## SFSymbol enhancements

Can now use colors
* hierarchical - single tint color to a hierarchy of layers
* palette - multiple colors to be explicitly specified
* multicolor - "fixed multicolor representation"

```swift
// Using a hierarchical color symbol

let configuration = UIImage.SymbolConfiguration(
    hierarchicalColor: UIColor.systemOrange
)

let image = UIImage(
    systemName: "sun.max.circle.fill",
    withConfiguration: configuration
)
```

### Style variants
Previously, these were specified with dotted strings.
New syntax
[[Design and build SF Symbols]]
[[SF Symbols in UIKit and AppKit]]

## Content size category limits
* restrict dynamic type sizes in view hierarchies
* Set minimum and maximum sizes
* `UIView.minimumContentSizeCategory` `maxiumContentSizeCategory`

Easily set a fuller or ? for the size.  Ensures your text/images look great at every text size setting.

Do not use this size to undo `limitText` size.  Its paramount that everything is legible on large text sizes.

?

## UIColor enhancements
Unified colors.
## Dynamic tint color
`UIColor.tintColor` resolved at runtime based on app or trait hierarchy

## Color picker enhancements
Continuous selection

`didSelect:continuously:`

## TextKit2
New text layout API
Used behind the scenes in `UITextField`
No adoption required

Better layout to text in languages with complex scripts.

[[Meet TextKit 2]]

## UIScene state restoration
NSUserActivity represents interface state

State restoration enhancements:
* Text interaction state properties
* Callback for restoring state after storyboard load
* Ability to extend state restoration

> If you're still using the old UIApplication lifecycle, now is the time to switch to UIScene.

[[Take your iPad apps to the next level]]

## Scene level sharing
Represent what's ahreable in an app
Used by
* Siri's "Share this" feature
* `NSSharingServicePickerToolbarItem` on Mac

[[Design great actions for Shortcuts, Siri, and Suggestions]]
[[Qualifies of a great Mac Catalyst app]]

## Cell configuration closures
* UICollectionView and UITableView
* React to changes in cell state
* Subclassing no longer necessary

```swift
// New UICollectionViewCell.configurationUpdateHandler closures

let cell: UICollectionViewCell = ...

cell.configurationUpdateHandler = { cell, state in
    var content = UIListContentConfiguration.cell().updated(for: state)
    content.text = "Hello world!"
    if state.isDisabled {
        content.textProperties.color = .systemGray
    }
    cell.contentConfiguration = content
}
```

Also available for UIButton

## Diffable datasource improvements
Apply snapshots without animation
New API to reconfigure existing items when the identifier doesn't change.

# Performance
## Cell prefetching improvements
* Behind-the-scenes prefetching changes
* no adoption required
* Up to 2x more time to prepare cells!

Can now `await` image

```swift
// Image display preparation

if let image = UIImage(contentsOfFile: pathToImage) {
    // Prepare the image for display asynchronously.
    async {
        let preparedImage = await image.byPreparingForDisplay()

        imageView.image = preparedImage
    }
}
```

```swift
// Image thumbnailing

if let bigImage = UIImage(contentsOfFile: pathToBigImage) {
    // Prepare the thumbnail asynchronously.
    async {
        let smallImage = await bigImage.byPreparingThumbnail(ofSize: smallSize)

        imageView.image = smallImage
    }
}
```

[[Make blazing fast lists and collection views]]

## Swift async/await
[[Meet asyncawait in Swift]]
[[Meet AsyncSequence]]

# Security/privacy

## Location button
One-time access to location
Flexible configuration API
[[Meet the Location Button]]

## Pasteboard
Eliminating the banner disclosing copy/paste every time that the system can confirm it's related to using a standard copy/paste interface.  e.x. cmd-v.

## Standard paste items
UIREsponder selectors and UIAction identifiers for:
* Paste
* Paste and Go
* Paste and Search
* Paste and Match Style

For each of these there are standard UIResponder selectors.

## Basic info
Soemtimes you need only basic info on what's in the pasteboard, is it URL, phone number etc? 

None of these require a notice (if we think access is standard?)

## Private Click Measurement
App-to-web click attribution, integrates with Safari attribution.

* Privacy-preserving measurement of ad clicks and taps
* Cover ads with `UIEventAttributionViews`
* Supply `UIEventAttribution` when opening a URL

[[Meet privacy-preserving ad attribution]]
https://webkit.org/blog/11529/introducing-private-click-measurement-pcm/

# Wrap up
* Compile your app for iOS 15
* Check out new system features
* Adopt the new iOS 15 look
* Use the new APIs



