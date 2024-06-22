Make your windows feel tailor-made for macOS. Fine-tune your app's windows for focused purposes, ease of use, and to express functionality. Use SwiftUI to style window toolbars and backgrounds. Arrange your windows with precision, and make smart decisions about restoration and minimization.

###  Style Toolbars - Removing Title - 3:11
```swift
.toolbar(removing: title)
```

###  Style Toolbars - Removing Toolbar Background - 3:14
```swift
.toolbarBackgroundVisibility(.hidden, for: .windowToolbar)
```

###  Refine Behaviors - Adding Container Background - 4:33
```swift
.containerBackground(.thickMaterial, for: .window)
```

###  Refine Behaviors - Minimize Behavior - 5:13
```swift
.windowMinimizeBehavior(.disabled)
```

###  Refine Behaviors - Restoration Behavior - 5:44
```swift
.restorationBehavior(.disabled)
```

###  Adjust Placement - Default Placement - 7:11
```swift
.defaultWindowPlacement { content, context in 
    var size = content.sizeThatFits(.unspecified) 
    let displayBounds = context.defaultDisplay.visibleRect 
    // modify size based on display bounds
    return WindowPlacement(size: size) 
}
```

###  Adjust Placement - Ideal Placement - 8:35
```swift
.windowIdealPlacement { content, context in 
    var size = content.sizeThatFits(.unspecified) 
    let displayBounds = context.defaultDisplay.visibleRect 
    // modify size based on display bounds
    return WindowPlacement(size: size) 
}
```

###  Borderless Window - 9:48
```swift
.windowStyle(.plain)
```

###  Default Launch Behavior - 9:53
```swift
.defaultLaunchBehavior(.presented)
```
# Resources
* https://developer.apple.com/documentation/SwiftUI/Customizing-window-styles-and-state-restoration-behavior-in-macOS
* https://developer.apple.com/documentation/visionOS/destination-video
* https://developer.apple.com/documentation/SwiftUI/Windows
