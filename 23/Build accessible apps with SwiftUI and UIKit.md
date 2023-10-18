#uikit #accessibility #swiftui 
Discover how advancements in UI frameworks make it easier to build rich, accessible experiences. Find out how technologies like VoiceOver can better interact with your app's interface through accessibility traits and actions. We'll share the latest updates to SwiftUI that help you refine your accessibility experience and show you how to keep accessibility information up-to-date in your UIKit apps.

Everyone deserves access to technology.  Over the past year, we've been working on a number of enhancements to ensure everyone has the best experience.

# Accessibility enhancements
Custom button with on/off state.  Lets us toggle on/off the image filter.  System does not know the create accessibility hint and title for this custom UI.  Watn to ensure we provide a good AX experience.

## Build a new trait collection instance from scratch - 1:54
```swift
import SwiftUI

struct FilterButton: View {
    @State var filter: Bool = false

    var body: some View {
        Button(action: { filter.toggle() }) {
            Text("Filter")
        }
        .background(filter ? darkGreen : lightGreen)
        .accessibilityAddTraits(.isToggle)
    }
}
```

new `isToggle` trait.  

## Add the accessibility toggle trait with UIKit - 2:31
```swift
import UIKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let filterButton = UIButton(type: .custom)

        setupButtonView()

        filterButton.accessibilityTraits = [.toggleButton]

        view.addSubview(filterButton)
    }
}
```

AX notifications are a new API that provide a unified multiplatform way to create announcements to convey information.

SwiftUI, UIKit, AppKit.

* announcement
* layoutChanged
* screenChanged
* pageScrolled
## Post an accessibility notification - 3:43
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        NavigationView {
            PhotoFilterView
                .toolbar {
                    Button(action: {
                        AccessibilityNotification.Announcement("Loading Photos View")
                            .post()
                    }) {
                        Text("Photos")
                    }
                }
        }
    }
}
```

Now we can set an announcement priority, which lets you set the importance of an announcement when multiple are possible.  

## Assign announcement priority - 5:13
```swift
import SwiftUI

struct ZoomingImageView: View {
  
    var defaultPriorityAnnouncement = AttributedString("Opening Camera")

    var lowPriorityAnnouncement: AttributedString {
        var lowPriorityString = AttributedString("Camera Loading")
        lowPriorityString.accessibilitySpeechAnnouncementPriority = .low
        return lowPriorityString
    }
    
    var highPriorityAnnouncement: AttributedString {
        var highPriorityString = AttributedString("Camera Active")
        highPriorityString.accessibilitySpeechAnnouncementPriority = .high
        return highPriorityString
    }
  
    // ...
}
```

|         | can interrupt | can be interrupted |
| ------- | ------------- | ------------------ |
| high    | yes           | no                 |
| default | yes           | yes                |
| low     | no            | yes                   |


## Post announcements with priority set - 5:46
```swift
import SwiftUI

struct CameraButton: View {
    
    // ...
  
    var body: some View {
        Button(action: {
            // Open Camera Code
            AccessibilityNotification.Announcement(defaultPriorityAnnouncement).post()
            // Camera Loading Code
            AccessibilityNotification.Announcement(lowPriorityAnnouncement).post()
            // Camera Loaded Code
            AccessibilityNotification.Announcement(highPriorityAnnouncement).post()
        }) {
            Image("Camera")
           }
        }
    }
}
```

## Assign announcement priority with UIKit - 6:15
```swift
class ViewController: UIViewController {
    let defaultAnnouncement = NSAttributedString(string: "Opening Camera", attributes: 
        [NSAttributedString.Key.UIAccessibilitySpeechAttributeAnnouncementPriority: 
        UIAccessibilityPriority.default]
    )

    let lowPriorityAnnouncement = NSAttributedString(string: "Camera Loading", attributes:   
        [NSAttributedString.Key.UIAccessibilitySpeechAttributeAnnouncementPriority:
        UIAccessibilityPriority.low]
    )

    let highPriorityAnnouncement = NSAttributedString(string: "Camera Active", attributes: 
        [NSAttributedString.Key.UIAccessibilitySpeechAttributeAnnouncementPriority:  
        UIAccessibilityPriority.high]
    )

    // ...
}
```

Can be hard to do this pinch with VO enabled.  Let's add a zoom action.

## Add the accessibility zoom action - 6:56
```swift
struct ZoomingImageView: View {
    @State private var zoomValue = 1.0
    @State var imageName: String?

    var body: some View {
        Image(imageName ?? "")
            .scaleEffect(zoomValue)
            .accessibilityZoomAction { action in
                let zoomQuantity = "\(Int(zoomValue)) x zoom"
                switch action.direction {
                case .zoomIn:
                    zoomValue += 1.0
                    AccessibilityNotification.Announcement(zoomQuantity).post()
                case .zoomOut:
                    zoomValue -= 1.0
                    AccessibilityNotification.Announcement(zoomQuantity).post()
                }
            }
    }
}
```



## Add the accessibility zoom action with UIKit - 7:18
```swift
import UIKit

class ViewController: UIViewController {
    let zoomView = ZoomingImageView(frame: .zero)
    let imageView = UIImageView(image: UIImage(named: "tree"))

    override func viewDidLoad() {
        super.viewDidLoad()
        zoomView.isAccessibilityElement = true
        zoomView.accessibilityLabel = "Zooming Image View"
        zoomView.accessibilityTraits = [.image, .supportsZoom]

        zoomView.addSubview(imageView)
        view.addSubview(zoomView)
    }
}
```

kinda lets people scroll through 2x, 3x, 4x, etc. zooms.



## Respond to accessibility zoom actions with UIKit - 7:43
```swift
import UIKit 

class ZoomingImageView: UIScrollView {
    override func accessibilityZoomIn(at point: CGPoint) -> Bool {
        zoomScale += 1.0

        let zoomQuantity = "\(Int(zoomValue)) x zoom"  
        UIAccessibility.post(notification: .announcement, argument: zoomQuantity)
        return true
    }

    override func accessibilityZoomOut(at point: CGPoint) -> Bool {
        zoomScale -= 1.0

        let zoomQuantity = "\(Int(zoomValue)) x zoom" 
        UIAccessibility.post(notification: .announcement, argument: zoomQuantity)             
        return true
    }
}
```

Difficult to quickly press keys in succession.  Sometimes people need to directly interact with your app to use it properly, ex piano keys.  We want people to directly touch the piano keys.  Adopt direct touch trait.

Two new direct touch options
* silent on touch
	* VO will be silent in this area
* requires activation
	* direct touch area require voiceover to activate the elements before touch happens.

## Use accessibility direct touch options - 10:10
```swift
import SwiftUI

struct KeyboardKeyView: View {
    var soundFile: String
    var body: some View {
        Rectangle()
            .fill(.white)
            .frame(width: 35, height: 80)
            .onTapGesture(count: 1) {
                playSound(sound: soundFile, type: "mp3")
            }            
            .accessibilityDirectTouch(options: .silentOnTouch)
    }
}
```

## Use accessibility direct touch options with UIKit - 10:46
```swift
import UIKit

class ViewController: UIViewController {
    let waveformButton = UIButton(type: .custom)

    override func viewDidLoad() {
        super.viewDidLoad()
        
        waveformButton.accessibilityTraits = .allowsDirectInteraction
        waveformButton.accessibilityDirectTouchOptions = .silentOnTouch
        waveformButton.addTarget(self, action: #selector(playTone), for: .touchUpInside)
        
        view.addSubview(waveformButton)
    }
}
```

# Improve accessibility visual

Content shape - set accessibility path.  Controls their appearance on the screen.  Previously, we change the accessibility shape and hit content shape.  Now we have a shape only for AX.

Quickly update an element's path with an existing swiftUI shape.

## Set the accessibility content shape - 12:21
```swift
import SwiftUI

struct ImageView: View {
    var body: some View {
        Image("circle-red")
            .resizable()
            .frame(width: 200, height: 200)
            .accessibilityLabel("Red")
            .contentShape(.accessibility, Circle())
    }
}
```



# Keep state up-to-date

I want the accessibility value for the image to represent whether the photo is filtered or not-filtered.  Now there is an easy way to keep AX attributes always in line with presented UI.

I can run this block.

* provide a closure for attributes
* evaluates closure every time a view is referenced
* note weak reference to self to avoid retain cycle.

## Update accessibility values using block-based setters with UIKit - 13:35
```swift
import UIKit 

class ViewController: UIViewController {
    var isFiltered = false

    override func viewDidLoad() {
        super.viewDidLoad()
        // Set up views
        zoomView.accessibilityValueBlock = { [weak self] in
            guard let self else { return nil }
            return isFiltered ? "Filtered" : "Not Filtered"
        }
    }
}
```

# Next steps
* integrate new accessibility traits
* Create custom accessibility shapes
* Evaluate attribute setting

