Explore everything new in UIKit, including tab and document launch experiences, transitions, and text and input changes. We'll also discuss better-than-ever interoperability between UIKit and SwiftUI animations and gestures, as well as general improvements throughout UIKit.

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

# Resources
* https://developer.apple.com/documentation/uikit/app_and_environment/automatic_trait_tracking
* https://developer.apple.com/documentation/Updates/UIKit
* https://developer.apple.com/documentation/uikit/uiupdatelink
