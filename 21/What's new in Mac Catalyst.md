#catalyst 

Bring existing iOS applications to macOS.

# Overview of new APIs
## Pop-up button support
`showsMenuAsPrimaryAction`.

`changesSelectionAsPrimaryAction`.  Title of button tracks the selection.

With these two properties there are 4 ways to configure the button.

* both false => standard push button
* `changesSelectionAsPrimaryAction` => toggle button
* `showsMenuASPrimaryAction` => pulldown menu
* both => new popup button

For buttons configured with a menu, menu is immediately with left-click.  

For buttons that don't show a menu, behavior depends on app idiom.  In iPad idiom, menus are triggered with secondary interaction.  Mac idiom => long press on the button.

[[Meet the UIKit button system]]

## Tooltips

Floating window over a view.  Can provide context.

* UIToolTipInteraction

```swift
let toolTipInteraction = UIToolTipInteraction(defaultToolTip: string)
view.addInteraction(tooltipInteraction)
```

* UIControl

```swift
control.toolTip = "Enable updates"
```

* Expansion tooltips

UILabel => `showsExpansionTextWhenTruncated`

## Printing APIs
```xml
<key>UIApplicationSupportsPrintCommand</key>
</true>
```

Tells OS to make the menu items.  Still need to implement print support in code.

`printContent` in UIResponder.

```swift
func printContent(_ sender: Any?) {
    let printInteractionController = UIPrintInteractionController.shared
    ...
}
```

## Window subtitle
Provide auxillary info about state.  

`UIScene.subtitle`

Shortcuts from custom intents.  If you previously disabled building intents extension for catalyst, consider e-enabling

[[Meet shortcuts for macOS]]

# Overview of enhanced APIs
* Control opt-out
* UIButton
* UISlider

[[Meet the UIKit button system]]

`UIBehavioralStyle`.  `preferredBehavioralStyle` RW.  `behavioralStyle` RO resolved property.

```swift
let button = UIButton(configuration: config)
button.preferredBehavioralStyle = .pad
```

## Window tab opt-out

```xml
<key>UIApplicationSceneManifest</key>
	<dict>
		<key>UIApplicationSupportsMultipleScenes</key>
		<true/>
		<key>UIApplicationSupportsTabbedSceneCollection</key>
		<false/>
	</dict>
```

## Cursor changes
* UIPointerLockState
* UIPointerShape
* UIPointerStyle.hidden
# Demo

[[Qualities of a great Mac Catalyst app]]
[[Optimize the Interface of Your Mac Catalyst App]]

## Scene subtitles
1.  Display VC titles
2.  Provide navigation context

```swift
let subtitle: String = "..."
let scene: UIScene = ...
scene.subtitle = subtitle
```

Set either at scene connection time or later.

## Tooltips
* Provide detail on hover
	* Educating, without cluttering UI
* Truncated text

```swift
// ToolTip Interaction
let imageView: UIImageView = UIImageView(frame: .zero)
let interaction = UIToolTipInteraction(defaultToolTip: "...")
imageView.addInteraction(interaction)

// ToolTips - Label Expansion Text
let label: UILabel = UILabel()
label.text = "..."
label.showsExpansionTextWhenTruncated = true

// ToolTips — On UIControls
let switchControl = UISwitch()
switchControl.toolTip = "..."
```

## New buttons

[[Meet the UIKit button system]]

Buttons pick up the tint color of their environment.

```swift
let submitButton = UIButton(type: .system)
submitButton.role = .primary
submitButton.setTitle("Submit", for: .normal)
```

```swift
// Toggle button with menu
let toggleButton = UIButton(configuration: .filled(), primaryAction: nil)
toggleButton.configuration?.title = "Points Multiplier"
toggleButton.changesSelectionAsPrimaryAction = true
toggleButton.menu = ...

// Elsewhere...
toggleButton.configuration?.baseBackgroundColor = .systemRed
```

```swift
let resetButton = UIButton(configuration: .plain(), primaryAction: nil)
resetButton.configuration?.title = "Reset"
resetButton.role = .destructive
resetButton.tintColor = .systemRed
```

Similar to iPadOS appearance.

```swift
let popup = UIButton(type: .system)
popup.changesSelectionAsPrimaryAction = true
popup.showsMenuAsPrimaryAction = true
popup.menu = ...
```


`menu` is critical to getting the popup appearance.  

Opt out of macos appearance

```swift
let button = UIButton(configuration: .filled(), primaryAction: nil)
button.configuration?.image = UIImage(systemName: "leaf")
button.preferredBehavioralStyle = .pad
button.configuration?.preferredSymbolConfigurationForImage =   
    UIImage.SymbolConfiguration(pointSize: 60)
button.changesSelectionAsPrimaryAction = true
button.configurationUpdateHandler = colorUpdateHandler
```

Mix and match in many ways.

## Printing
* Built-in key command support
* Built on the responder chain

1.  `UIApplicationSupportsPrintCommand` => YES

```swift
override func printContent(_: Any?) {
    ...
}

override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
    if action == #selector(self.printContent(_:)) {
        ...
    } else {
        return super.canPerformAction(action, withSender: sender)
    }
}

//allows a nonresponder to respond?
override func target(forAction action: Selector, withSender sender: Any?) -> Any? {
    switch action {
    case #selector(UIResponder.printContent(_:)):
        ...
    default:
        return super.target(forAction: action, withSender: sender)
    }
}
```

# Wrap up
* New mac catalyst APIs
* Enhanced mac catalyst APIs
* Adoption in Trip Planner

https://developer.apple.com/documentation/uikit/printing/building_and_improving_your_app_with_mac_catalyst
https://developer.apple.com/tutorials/Mac-Catalyst
https://developer.apple.com/design/human-interface-guidelines/ios/overview/mac-catalyst/
https://developer.apple.com/documentation/uikit/mac_catalyst
