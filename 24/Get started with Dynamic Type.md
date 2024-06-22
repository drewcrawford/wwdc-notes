

Dynamic Type lets people choose their preferred text size across the system and all of their apps. To help you get started supporting Dynamic Type, we'll cover the fundamentals: How it works, how to find issues with scaling text in your app, and how to take practical steps using SwiftUI and UIKit to create a great Dynamic Type experience. We'll also show how you can best use the Large Content Viewer to make navigation controls accessible to everyone.

Important quality of visual accessibility: how peopel read and navigate text in your app and how dynamic type can make this an important experience for everyone.

Many people will customize this setting, essential to support this feature, etc.

If you've never considered how your app might work with larger text sizes, you're in the right place.  build interfaces that work across any screen size, orientation and platform.  Different people prefer or need different text sizes.  Improves readability at all text sizes.  Text it out in AX settings!

5 more text sizes become available.  Can also change from control center.

# Scaled text

Your app presents one of the system provided text styles.  ex body is comfortable for multiple liens of texts.
Headline -> distinguishes heading from surrounding content.  Your app's text can automatically adjust to different sizes, while retaining the visual hierarchy of your content.

To use a builtin text style, use the `.font` modifier.
### Built-in text styles with SwiftUI - 3:53
```swift
// Use built-in text styles with SwiftUI
import SwiftUI

struct ContentView: View {

    var body: some View {
        Text("Hello, World!")
            .font(.title)
    }

}
```

Possible issues
* truncations
* clipping
* fixed frames




### Built-in text styles in UIKit - 4:06
```swift
// Built-in text styles in UIKit
import UIKit

class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let label = UILabel(frame: .zero)
        setupConstraints()
        label.text = "Hello, World!"
        label.adjustsFontForContentSizeCategory = true
        label.font = .preferredFont(forTextStyle: .title1)
        label.numberOfLines = 0
        
        self.view.addSubview(label)
    }
}
```
Xcode will generate a preview for every variant of text sizes available, so you can quickly locate issues for a particular view.  Or click on settings button in xcode canvas to select a different text size. 

Use the xcode debugger.  Click settings icon to override dynamic type and other AX settings.  Can also perform an audit of accessibility


# Dynamic layouts
Consider adapting layouts for best experience.  Ex, when creating a new contact poster in the contacts app, the poster options appear in a horizontal stack.  When text size increases, layout switches to vertical stack.


### Dynamic layout in SwiftUI - 7:20
```swift
// Dynamic layout in SwiftUI
import SwiftUI

struct FigureCell: View {
    @Environment(\.dynamicTypeSize) 
    private var dynamicTypeSize: DynamicTypeSize
    
    var dynamicLayout: AnyLayout { 
        dynamicTypeSize.isAccessibilitySize ?
        AnyLayout(HStackLayout()) : AnyLayout(VStackLayout())
    }
    
    let systemImageName: String
    let imageTitle: String
    
    var body: some View {
        dynamicLayout {
            FigureImage(systemImageName: systemImageName)
            FigureTitle(imageTitle: imageTitle)
        }
    }
}
```


### Dynamic layout in SwiftUI - 7:52
```swift
// Dynamic layout in SwiftUI
import SwiftUI

struct FigureContentView: View {
    @Environment(\.dynamicTypeSize) 
    private var dynamicTypeSize: DynamicTypeSize
    
    var dynamicLayout: AnyLayout {
        dynamicTypeSize.isAccessibilitySize ?
        AnyLayout(VStackLayout(alignment: .leading)) : AnyLayout(HStackLayout(alignment: .top))
    }
    
    var body: some View {
        dynamicLayout {
            FigureCell(systemImageName: "figure.stand", imageTitle: "Standing Figure")
            FigureCell(systemImageName: "figure.wave", imageTitle: "Waving Figure")
            FigureCell(systemImageName: "figure.walk", imageTitle: "Walking Figure")
            FigureCell(systemImageName: "figure.roll", imageTitle: "Rolling Figure")
        }
    }
}
```

In UIKit, consider using UIStackView.

### Dynamic layout in UIKit - 8:20
```swift
// Dynamic layout in UIKit
import UIKit

class ViewController: UIViewController {
    private var mainStackView: UIStackView = UIStackView()
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        NotificationCenter.default.addObserver(self, selector: #selector(textSizeDidChange(_:)), name: UIContentSizeCategory.didChangeNotification, object: nil)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupStackView()
    }

    @objc private func textSizeDidChange(_ notification: Notification?) {
        let isAccessibilityCategory = self.traitCollection.preferredContentSizeCategory.isAccessibilityCategory
        mainStackView.axis = isAccessibilityCategory ? .vertical : .horizontal
        setupConstraints()
    }
}
```



# Images and symbols
balance between
* scaling icon with text
* but you have less space anyway, so...


Prioritize scaling essential content, over decorative views.  Wrap text to avoid unused whitespace.
Remove decorative views sparingly

In SwiftUI, we wrap text with icons without any additional work.

### Scale inline images with SwiftUI - 10:12
```swift
// Inline images in SwiftUI
import SwiftUI

struct ContentView: View {

    var body: some View {
        List {
            FigureListCell(figureName: "Standing Figure", systemImage: "figure.stand")
            FigureListCell(figureName: "Rolling Figure", systemImage: "figure.roll")
            FigureListCell(figureName: "Waving Figure", systemImage: "figure.wave")
            FigureListCell(figureName: "Walking Figure", systemImage: "figure.walk")
        }
    }

}
```

UIKit - Achieve by using NSAttributedString, NSTextAttachment.
### Scale inline images with UIKit - 10:30
```swift
// Inline images in UIKit
func attributedStringWithImage(systemImageName: String, imageTitle: String) -> NSAttributedString {
    let attachment = NSTextAttachment()
    attachment.image = UIImage(systemName: systemImageName)
    
    let attachmentAttributedString = NSMutableAttributedString(attachment: attachment)
    attachmentAttributedString.append(NSAttributedString(string: imageTitle))
    
    return attachmentAttributedString
}
```

Maybe you want the image to adjust its size as well, if it has text or key iconography that is relevant to the rest of the content.

SFSymbol -> resizes automatically.


### Scale images in SwiftUI - 11:05
```swift
// Scaling images in SwiftUI
import SwiftUI

struct ContentView: View {
    @ScaledMetric var imageWidth = 125.0
    var body: some View {
        VStack {
            Image("Spatula")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: imageWidth)
            Text("Grill Party!")
                .frame(alignment: .center)
        }
    }
}
```

UIKit - use SymbolConfiguration.
### Scale symbols with UIKit - 11:38
```swift
// Symbol configuration in UIKit
import UIKit

func imageWithBodyConfiguration(systemImageName: String) -> UIImage? {
  let imageConfiguration = UIImage.SymbolConfiguration(textStyle: .body)
  let configuredImage = UIImage(systemName: systemImageName, withConfiguration: imageConfiguration)
  return configuredImage
}
```


# Large content viewer

Demo.

Automatically supported in
* tab bars
* status bars
* navigation bars
* toolbars
Add support to custom controls.

### Add large content viewer support with SwiftUI - 13:15
```swift
// Large content viewer support in SwiftUI
import SwiftUI

struct FigureBar: View {
    @Binding var selectedFigure: Figure
    
    var body: some View {
       HStack {
            ForEach(Figure.allCases) { figure in
                FigureButton(figure: figure, isSelected: selectedFigure == figure)
                    .onTapGesture {
                        selectedFigure = figure
                    }
                    .accessibilityShowsLargeContentViewer {
                        Label(figure.imageTitle, systemImage: figure.systemImage)
                    }
            }
        }
    }
}
```

Conform view to UILargeContentViewerItem
provide title and image
add an instance of `UILargeContentViewrInteraction` to the view
Resolve ambiguous gestures using `gestureRecognizerForExclusionRelationship` on the interaction.


### Add large content viewer support with UIKit - 13:45
```swift
// Large content viewer support in UIKit
import UIKit

class FigureCell: UIStackView {
    var systemImageName: String!
    var imageTitle: String!
    var imageLabel: UILabel!
    var titleImageView: UIImageView!
    
    required init(coder: NSCoder) {
        super.init(coder: coder)
        setupFigureCell()
    }
    
    init(systemImageName: String, imageTitle: String) {
        super.init(frame: .zero)
        
        self.systemImageName = systemImageName
        self.imageTitle = imageTitle
        
        setupFigureCell()

        self.addInteraction(UILargeContentViewerInteraction())
        self.showsLargeContentViewer = true
        self.largeContentImage = UIImage(systemName: systemImageName)
        self.scalesLargeContentImage = true
        self.largeContentTitle = imageTitle
    }
}
```

# Next steps
* test your app with larger sizes
* Use system styles and font
* Tailor layout to prioritize readability
* audit your app for accessibility

[[Catch up on accessibility in SwiftUI]]


# Resources
* [accessibilityShowsLargeContentViewer()](https://developer.apple.com/documentation/SwiftUI/View/accessibilityShowsLargeContentViewer())
* [Enhancing the accessibility of your SwiftUI app](https://developer.apple.com/documentation/Accessibility/enhancing-the-accessibility-of-your-swiftui-app)
* [Forum: Accessibility & Inclusion](https://developer.apple.com/forums/topics/accessibility-and-inclusion?cid=vf-a-0010)
* [Human Interface Guidelines: Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility)
* [Human Interface Guidelines: Typography](https://developer.apple.com/design/human-interface-guidelines/typography)
* [UILargeContentViewerInteraction](https://developer.apple.com/documentation/uikit/uilargecontentviewerinteraction)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10074/4/3CB84B8B-3CC6-4EAB-AA46-E9FD7D160048/downloads/wwdc2024-10074_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10074/4/3CB84B8B-3CC6-4EAB-AA46-E9FD7D160048/downloads/wwdc2024-10074_sd.mp4?dl=1)