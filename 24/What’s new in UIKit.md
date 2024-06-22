Explore everything new in UIKit, including tab and document launch experiences, transitions, and text and input changes. We'll also discuss better-than-ever interoperability between UIKit and SwiftUI animations and gestures, as well as general improvements throughout UIKit.

# Key features

## document launch experience

[[Evolve your document launch experience]]

## updated tab and sidebar

Describes app structure
Multiplatform experience

[[Elevate your tab and sidebar experience in iPadOS]]

## fluid transitions

All-new zoom transition
Works with navigation and presentations
Continuously interactive and interruptible


# UIKit and SwiftUI interoperability

## animations
### Using SwiftUI to animate UIViews with gestures - 0:01
```swift
switch gesture.state {
    case .changed:
        UIView.animate(.interactiveSpring) {
            bead.center = gesture.translation
        }
    case .ended:
        UIView.animate(.spring) {
            bead.center = endOfBracelet
        }
}
```

[[Enhance your UI animations and transitions]]

## gesture recognizers

UIKit and SwiftUI gestures are unified.
Can specify dependencies between them
use `name` to refer 
### Setting failure requirements between gestures - 0:02
```swift
// Inner SwiftUI double tap gesture
Circle()
    .gesture(doubleTap, name: "SwiftUIDoubleTap")

// Outer UIKit single tap gesture
func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRequireFailureOf other: UIGestureRecognizer) -> Bool {
    other.name == "SwiftUIDoubleTap"
}
```

Add your UIKit gesture recognizers to SwiftUI seamlessly
UIGestureRecognizerRepresentable

[[What’s new in SwiftUI]]

# General enhancements

## automatic trait tracking
supported in common view and view controller update methods, such as layoutSubviews and drawRect.

This tracks with traits are used in the method.
UIKit automatically performs associated invalidation for that method, such as setNeedsLayout or setNeedsDisplay.

### Responding to horizontalSizeClass trait - 0:03
```swift
class MyView: UIView {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        if traitCollection.horizontalSizeClass == .compact {
            // apply compact layout
        } else {
            // apply regular layout
        }
    }
}
```

We no longer need `registerForTraitChanges`.

maximum performance with minimal dependencies
Always active within supported methods

see docs
## list improvements

Access via UITraitCollection.listEnvironment
Describes style of the list
Used by 
* UIListcontentConfiguration
* UIBackgroundConfiguration

updatedConfiguration(for:) uses listEnvironment

### Using the new automatic content and background configurations - 0:04
```swift
func configurations(for location: FileLocation) -> (UIListContentConfiguration, UIBackgroundConfiguration) {
    var contentConfiguration = UIListContentConfiguration.cell()
    let backgroundConfiguration = UIBackgroundConfiguration.listCell()
    
    contentConfiguration.text = location.title
    contentConfiguration.image = location.thumbnailImage
    
    return (contentConfiguration, backgroundConfiguration)
}
```

cell/listCell now uatomatically update

| updated     | new    |
| ----------- | ------ |
| cell        | header |
| subttleCell | footer |
| valueCell   |        |
|             |        |

## update link
Similar to CADisplayLink
More features
* view tracking
* Low latency applications
* Better  performance

### Using UIUpdateLink - 0:05
```swift
let updateLink = UIUpdateLink(
    view: view,
    actionTarget: self,
    selector: #selector(update)
)

updateLink.requiresContinuousUpdates = true
updateLink.isEnabled = true

@objc func update(updateLink: UIUpdateLink, updateInfo: UIUpdateInfo) {
    view.center.y = sin(updateInfo.modelTime) * 100 + view.bounds.midY
}
```

see UIUpdateLink docs.

## symbol animations

`.wiggle`
`breathe`
`.rotate` -> only part of symbol rotates
`periodic` -> repeat count, delay.  Or `continuous`.

"magic replace".  Can specify an explicit fallback style.

* updated sf symbols app
* variable oclor looping annotations
* continuous bounce effects
[[What's new in SF Symbols 6]]

[[Animate symbols in your app]]

## sensory feedback
* attach feedback generators to view
* Pass location of each feedback
* `UICanvasFeedbackGenerator`.  

### An example of providing UICanvasFeedbackGenerator with additional context - 0:06
```swift
@ViewLoading var feedbackGenerator: UICanvasFeedbackGenerator

override func viewDidLoad() {
    super.viewDidLoad()
    feedbackGenerator = UICanvasFeedbackGenerator(view: view)
}

func dragAligned(_ sender: UIPanGestureRecognizer) {
    feedbackGenerator.alignmentOccurred(at: sender.location(in: view))
}
```

May play haptics, audio, both, or neither.  Use the same generators on all platforms regardless of expected effect.
## text improvements

New edit menu actions, that brings up new formatting panel with default set of options.
Changing fonts and sizes, adding lists, and modifyign other attributes is clear and easy.


### Using new attributes for highlight - 0:07
```swift
var attributes = [NSAttributedString.Key: Any]()

// Highlight style
attributes[.textHighlightStyle] = NSAttributedString.TextHighlightStyle.default

// Highlight color scheme
attributes[.textHighlightColorScheme] = NSAttributedString.TextHighlightColorScheme.default
```


### Customizing formatting panel - 0:08
```swift
textView.textFormattingConfiguration = .init(groups: [
    .group([
        .component(.fontAttributes, .mini),
        .component(.fontPicker, .regular),
        .component(.textColor, .mini)
    ]),
    .group([
        .component(.fontPointSize, .mini),
        .component(.listStyles, .regular),
        .component(.highlight, .mini)
    ])
])
```

### writing tools support
* UITextView built-in support
* New APIs to customize behavior


## menu actions

Invokable by system
UICommand
UIKeyCommand
UIActions
Performance through iPhone mirroring on the mac keyboard

[[Take your iPad apps to the next level]]

## apple pencil
UIKit has support for all great new features of apple pencil pro
Squeeze gesture
Haptics
Barrel roll angle
Undo slider

Configure PKToolPicker
Add your custom tools
Support Apple Pencil Pro

[[Squeeze the most out of Apple Pencil]]

# Next steps
* compile your app for iOS 18
* Adopt new UIKit APIs
* Take advantage of new features
	* transitions/aniamtions
	* tab bars
	* document launch experience

# Resources
* https://developer.apple.com/documentation/uikit/app_and_environment/automatic_trait_tracking
* https://developer.apple.com/documentation/Updates/UIKit
* https://developer.apple.com/documentation/uikit/uiupdatelink
