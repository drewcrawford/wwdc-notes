#catalyst 

"optimizing"
*  Offers improved visuals and controls
* Runs your app in the Mac idiom
* Enhances existing Mac Catalyst apps



[[taking iPad apps for mac to the next level - 19]]
[[Designing iPad Apps for Mac - 19]]

# What gets optimized?
100pt vs 77pt

Not scaling your content paves the way for many more visual improvements.
macOS system controls.  System iOS button will render as a system mac button when optimized.
Has sizes and metrics matching on the mac.  Layouts might be altered.
macOS font metrics.
However, if you have hardcoded font sizes, they won't adjust.

macOS standard spacing.  Typically larger on mac than on iOS.

## comparison
This is an alternative to catalyst scaled.
Scaling might be better.  This isn't an obsolete thing.
Scaled is a fast-path to a mac app.  
Optimizing for mac is a possible next step after scaling.
Scaling favors compatibility with iPad
Optimized favors authentic mac look and feel.
Scaled preserves layout.
Optimized requires some layout work.

## what makes a good candidate?
* Lots of text.  They want the true text sizes.  Swift playgrounds.
* Graphic emphasis.  SceneKit or Metal.  Higher framerates, lower graphic consumption on integrated and discrete.
* Custom artwork.  Scaling is important and the graphics may look better.  e.g., mapkit.
* Dependencies might limit you.  UIKit assumptions, I guess.

## screenshot comparison
* button is different
* Space on the leading edge is bigger.  System spacing values.
* Image is larger because it went from 77% of its natural size to 100% of its size.
	* consider using asset catalog to create a mac native asset
* slider space is less because it got ate with system padding and increased image size.
* slider adopted new macOS style.  It's very slight.
* Fonts match visually in size, however, since the view optimized for mac is using a true 13-pt font, it's sharper.  

# Control customization considerations

Tinted buttons aren't standard on the mac, and users aren't accustomed to them.  So we drop them.

* gesture recognizers on `UIControl`s.  However, GRs will not get called if you use a system button that has adopted the mac appearance.
	* Make sure you aren't bringing an iPad interaction over to the mac, you might want a menu item for example.
	* Auditing controls cutomized at the event handling level.  GRs, UIButtons, etc.
	* If the GR is non-negotiable, remember that the custom button type is not rendered with a mac appearance and will maintain the same event tracking as on iPad.
* Visual customizations on `UIControl`s.  e.g. custom slider tint color, thumb, etc.
	* When using system controls, you are limited to customizations where standard mac appearances overlap UIKit API

# Demo
Metrics changes.
Scale change.

`traitCollection.userInterfaceIdiom`.  Unclear to me if this detects "optimized" or merely any mac.

Asset catalog - "mac scaled" is underneath iPad, whereas "optimized for mac" uses native mac assets.
Nav bars don't really make sense.  There's example involving a modal with ios-style "save" and "cancel".  We want to either push these into the toolbar or down into the view itself, but not in this navbar that is neither thing.  Maybe more maclike to move them to the bottom of this view.

We recommend using dynamic text.

Customize installed per-idiom.  

## Recap
* optimized uses the mac idiom
* Mac look and feel, spacing changes
* Mac metrics
* Check for unsatisfiable constraint warnings
* asset catalog
	* Note that system will fall back to iPad assets first, before universal assets.  These may not be optimal
* Paradigm differences.  Navigation bar, button placement.
* Read the HIG.

# Supporting catalina
It's possible to back-deploy optimized apps, however
* on Catalina, they have standard iPadOS look and feel
* Means supporting both iPad and Mac idiom
* Audit layout
* Guard unavailable frameworks

```swift
// Idiom vs conditional compilation block

if traitCollection.userInterfaceIdiom == .mac {
    // "Optimize Interface for Mac" specific code
}

//keep in mind that this will be true regardless of idiom
#if targetEnvironment(macCatalyst)
    // Mac Catalyst specific code
#endif

//might be appropriate to use both techniques
if traitCollection.userInterfaceIdiom == .mac {
    // "Optimize Interface for Mac" specific code
} else if traitCollection.userInterfaceIdiom == .pad {
    #if targetEnvironment(macCatalyst)
        // Mac Catalyst specific code
    #else
        // iPad specific code
    #endif
}
```

# SwiftUI optimized for Mac
* similar control, text, and spacing approach as UIKit
* Context aware views may represent themselves differently
* Layout changes depend on container views

## GroupBox
Used for layering structured content and automatically choosing the right semantic colors.

```swift
// Nested GroupBoxes

struct ContentView: View {
    var body: some View {
        GroupBox {
            VStack {
                Text("High level information")
                GroupBox {
                    Text("Some elaborate details")
                }
            }
        }
    }
}
```

## Toggle
```swift
// DefaultToggleStyle

struct ContentView: View {
    @State var completed: Bool = false

    var body: some View {
        Toggle("Complete?", isOn: $completed)
    }
}
```
Sliding switch on iPadOS.  In optimized for mac, this is a checkbox.

## button
```swift
// System Button with SF Symbol

struct ContentView: View {
    var body: some View {
        Button(action: { }, label: {
            HStack {
                Image(systemName: "rays")
                Text("Click Me!")
            }
        })
    }
}
```

## datepicker
```swift
// DefaultDatePickerStyle

struct ContentView: View {
    @State var dueDate = Date()

    var body: some View {
        DatePicker("Due:", selection: $dueDate)
    }
}
```

default is compact in both idioms

## picker
```swift
// DefaultPickerStyle

struct ContentView: View {
    @State var sizeIndex = 2

    var body: some View {
        Picker("Size:", selection: $sizeIndex) {
            Text("Small").tag(1)
            Text("Medium").tag(2)
            Text("Large").tag(3)
        }
    }
}
```

Note that when optimizing for mac, you use a mac picker (combo box),  not a scrolling picker.
segmented control style is a non-default option on both platforms

## custom buttons
They come across exactly as they are.  Note that you need to conform to `ButtonStyle`.

```swift
// Custom gradient button

struct CustomNinetiesButtonStyle: ButtonStyle {
    var angle: Angle = .degrees(54.95)

    func gradient(shifted: Bool) -> AngularGradient {
        let lightTeal = Color(#colorLiteral(red: 0.2785285413, green: 0.9299042821, blue: 0.9448828101, alpha: 1))
        let yellow    = Color(#colorLiteral(red: 0.9300076365, green: 0.8226149678, blue: 0.59575665, alpha: 1))
        let pink      = Color(#colorLiteral(red: 0.9437599778, green: 0.3392140865, blue: 0.8994731307, alpha: 1))
        let purple    = Color(#colorLiteral(red: 0.5234025717, green: 0.3247769475, blue: 0.9921132922, alpha: 1))
        let softBlue  = Color(#colorLiteral(red: 0.137432307, green: 0.5998355746, blue: 0.9898411632, alpha: 1))

        let gradient = Gradient(stops:
                                    [.init(color:lightTeal, location: 0.2),
                                     .init(color: softBlue, location: 0.4),
                                     .init(color: purple, location: 0.6),
                                     .init(color: pink, location: 0.8),
                                     .init(color: yellow, location: 1.0)])

        return AngularGradient(gradient: gradient, center: .init(x: 0.25, y: 0.55), angle: shifted ? angle : .zero)
    }

    func makeBody(configuration: ButtonStyleConfiguration) -> some View {
        let background = NinetiesBackground(isPressed: configuration.isPressed,
                                            pressedGradient: gradient(shifted: false),
                                            unpressedGradient: gradient(shifted: true))
        return configuration.label
            .foregroundColor(configuration.isPressed ? Color.pink : Color.white)
            .modifier(background)
    }

    struct NinetiesBackground: ViewModifier {
        let isPressed: Bool
        let pressedGradient: AngularGradient
        let unpressedGradient: AngularGradient

        func body(content: Content) -> some View {
            let foreground = content
                    .padding(.horizontal, 24)
                    .padding(.vertical, 14)
                    .foregroundColor(.white)
            return foreground
                .background(Capsule().fill(isPressed ? pressedGradient : unpressedGradient))
        }
    }
}
struct ContentView: View {

    var body: some View {
      Button("Awesome", action: {})
          .buttonStyle(CustomNinetiesButtonStyle())
    }
}
```

## demo
## recap
* what gets optimized?
* good candidates for optimized for mac
* optimizing a catalyst app
* swiftUI optimized for mac

