Bring delight to your app with animated symbols. Explore the new Symbols framework, which features a unified API to create and configure symbol effects. Learn how SwiftUI, AppKit, and UIKit make it easy to animate symbols in user interfaces. Discover tips and tricks to seamlessly integrate the new animations alongside other app content. To get the most from this session, we recommend first watching “What's new in SF Symbols 5.”

Because people are familiar with symbols, they make your app easier to use.  Bring more life into your app, etc.

# Symbol effects
[[What's new in SF Symbols 5]]

Symbols framework.  Included for free when you use swiftUI/appkit/uikit.  Each effect has a simple dot-separated name.  

* discrete -> one-off animation
* indefinite -> keeps end state indefinitely, only end when removed
* transition -> in and out of view, appear/disappear
* content transition -> one symbol to another

Each behavior corresponds to a protocol.  Effects declare their behavior support by conforming to protocols.

|  | discrete | indefinite | transition | content transition |
| ---- | ---- | ---- | ---- | ---- |
| bounce | yes | no | no | no |
| pulse | yes | yes | no | no |
| variable color | yes | yes | no | no |
| scale | no | yes | no | no |
| appear | no | yes | yes | no |
| disappear | no | yes | yes | no |
| replace | no | no | no | yes |
# UI framework APIs
### Symbol effects in SwiftUI - 6:02
```swift
Image(systemName: "wifi.router")
    .symbolEffect(.variableColor.iterative.reversing)
    .symbolEffect(.scale.up)
```

### Symbol effects in AppKit and UIKit - 6:02
```swift
let imageView: NSImageView = ...

imageView.addSymbolEffect(.variableColor.iterative.reversing)
imageView.addSymbolEffect(.scale.up)
```


I also need to control when an effect is active.  I don't want to keep playing after app connects to network.

This can be done by adding boolean parameter.

### Indefinite symbol effects in SwiftUI - 6:49
```swift
struct ContentView: View {
    @State var isConnectingToInternet: Bool = true
    
    var body: some View {
        Image(systemName: "wifi.router")
            .symbolEffect(
                .variableColor.iterative.reversing,
                isActive: isConnectingToInternet
            )
    }
}
```

in appkit/ui kit we have an imperative style

### Indefinite symbol effects in AppKit and UIKit - 7:09
```swift
let imageView: NSImageView = ...

imageView.addSymbolEffect(.variableColor.iterative.reversing)

// Later, remove the effect
imageView.removeSymbolEffect(ofType: .variableColor)
```

what about discrete effects?  

In swiftUI I can use same symbole ffect modifier to add discrete effects.  I must also provide swiftUI a value.  When the value changes, swiftUI triggers the discrete effect.

ex

### Discrete symbol effects in SwiftUI - 8:26
```swift
struct ContentView: View {
    @State var bounceValue: Int = 0
    
    var body: some View {
        VStack {
            Image(systemName: "antenna.radiowaves.left.and.right")
                .symbolEffect(
                    .bounce,
                    options: .repeat(2),
                    value: bounceValue
                )
            
            Button("Animate") {
                bounceValue += 1
            }
        }
    }
}
```

imperative in appkit/uikit

### Discrete symbol effects in AppKit and UIKit - 8:26
```swift
let imageView: NSImageView = ...

// Bounce
imageView.addSymbolEffect(.bounce, options: .repeat(2))
```


### Content transition symbol effects in SwiftUI - 9:40
```swift
struct ContentView: View {
    @State var isPaused: Bool = false
    
    var body: some View {
        Button {
            isPaused.toggle()
        } label: {
            Image(systemName: isPaused ? "pause.fill" : "play.fill")
                .contentTransition(.symbolEffect(.replace.offUp))
        }
    }
}
```

### Content transition symbol effects in AppKit and UIKit - 9:57
```swift
let imageView: UIImageView = ...
imageView.image = UIImage(systemName: "play.fill")

// Change the image with a Replace effect
let pauseImage = UIImage(systemName: "pause.fill")!
imageView.setSymbolImage(pauseImage, contentTransition: .replace.offUp)
```

Finally, we have appear/disappear which can show/hide symbols.  These effects are uniquely classified as transition effects.  Before we get into that, let's talk about parallel universes.

In one universe, the image disappears.  But the image view is still in hierarchy.  No change to layout.

In parallel universe, imageview is truly added/removed from the hierarchy.  As a result, layout of surrounding views may change.

Appear/disappear support both behaviors.  First behavior is possible because appear/disappear are indefinite effects.  



### Indefinite Appear and Disappear symbol effects in SwiftUI - 11:14
```swift
struct ContentView: View {
    @State var isMoonHidden: Bool = false
    
    var body: some View {
        HStack {
            RoundedRectangle(cornerRadius: 5)

            Image(systemName: "moon.stars")
               .symbolEffect(.disappear, isActive: isMoonHidden)

            Circle()
        }
    }
}
```

### Indefinite Appear and Disappear symbol effects in AppKit and UIKit - 11:30
```swift
let imageView: UIImageView = ...
imageView.image = UIImage(systemName: "moon.stars")

imageView.addSymbolEffect(.disappear)
// Re-appear the symbol
imageView.addSymbolEffect(.appear)
```

Takeaway is that indefinite effects don't change layout at all.  Only rendering inside the imageview.

Transition behavior will appear/disappear the imageview.  Can be used with swiftUI's builtin transition modifier.

swiftui has a new transition modifier called `.symbolEffect`.    

### Transition symbol effects in SwiftUI - 12:38
```swift
struct ContentView: View {
    @State var isMoonHidden: Bool = false
    
    var body: some View {
        HStack {
            RoundedRectangle(cornerRadius: 5)

            if !isMoonHidden {
                Image(systemName: "moon.stars")
                    .transition(.symbolEffect(.disappear.down))
            }

            Circle()
        }
    }
}
```

### Appear and Disappear symbol effects in UIKit with completion handler - 12:59
```swift
let imageView: UIImageView = ...
imageView.image = UIImage(systemName: "moon.stars")

imageView.addSymbolEffect(.disappear) { context in
    if let imageView = context.sender as? UIImageView, context.isFinished {
        imageView.removeFromSuperview()
    } 
}
```


# Adoption tips

Here are some tips to take symbol fx to the next level.

First, uikit methods are also available on uibarbuttonitem.  Bring your toolbars to life using symbol animation.

Some UIKit controls also ahve built-in symbol animations on iOS 17.  UISlider for example now bounces with images.  See `isSymbolAnimationEnabled`.  

In SwiftUI, special considerations.
* effects propagate through the view hierarchy
* effects can be applied to multiple images with modifier on a parent view
* use `symbolEffectsRemoved` to avoid inheriting effects


### Symbol effect propagation in SwiftUI - 14:19
```swift
VStack {
    Image(systemName: "figure.walk")
        .symbolEffectsRemoved()
    Image(systemName: "car")
    Image(systemName: "tram")
}
.symbolEffect(.pulse)
```

You may be interested in havin ga symbol be initially shown or initially disappear without animation.  In SwiftUI you can use a transaction with animation disabled.
### Effects without animation in SwiftUI - 14:55
```swift
struct ContentView: View {
    @State var isScaledUp: Bool = false
    
    var body: some View {
        Image(systemName: "iphone.radiowaves.left.and.right")
            .symbolEffect(.scale.up, isActive: isScaledUp)
            .onAppear {
                var transaction = Transaction()
                transaction.disablesAnimations = true
                withTransaction(transaction) {
                    isScaledUp = true
                }
            }
    }
}
```

in appkit/uikit, use animated parameter on `addSymbolEffect` to apply without animation.

### Effects without animation in AppKit and UIKit - 15:06
```swift
// Effects without animation in AppKit and UIKit

let imageView: UIImageView = ...
imageView.image = UIImage(systemName: "iphone.radiowaves.left.and.right")

imageView.addSymbolEffect(.disappear, animated: false)
```

iOS16/ventura introduced variable values as another dimension.  Volume levels, etc.

In iOS 17/sonoma, we're making it super easy to use arbitrary value.  In swiftui, don't need to do anything.
### Variable value animations in SwiftUI - 15:44
```swift
struct ContentView: View {
    @State var signalLevel: Double = 0.5
    
    var body: some View {
        Image(systemName: "wifi", variableValue: signalLevel)
    }
}
```

As signal strength changes, wifi symbol automatically updates while also animating across variable values.

In appkit/uikit, use automatic symbol content transition. Detects that new symbol image just has a different variable value.

### Variable value animations in AppKit and UIKit - 16:07
```swift
let imageView: UIImageView = ...
imageView.image = UIImage(systemName: "wifi", variableValue: 1.0)

// Animate to a different Wi-Fi level
let currentSignalImage = UIImage(
    systemName: "wifi",
    variableValue: signalLevel
)!
imageView.setSymbolImage(currentSignalImage, contentTransition: .automatic)
```

# Next steps
* discover animations in the sf symbols app
* explore the symbol effects API
* Adopt symbol animations in your apps

[[What's new in SF Symbols 5]]
[[Create animated symbols]]
