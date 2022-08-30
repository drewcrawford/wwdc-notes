#xcode 
Single app target, multiple destinations.  

# Evaluate needs
* Potentially different codebases
* Few overlappign build settings
* Heavily customized for its destination

Consider continuing to use separate targets.

But a single app target
* support many destination sand platform
* common codebase and settings
* conditionalize individual settings and files

# Configure project
New targets, existing targets.

How to add toa ne xisting app?

If we study app target, see a list of all destiantions my app supports.  You can see I have a mac destination already.  This is for apple silicon.  

xcode wont' change my code, it will remove dependencies that aren't available.

Valid to have more than one mac destination.  Especially useful if I'm transitioning from catalyst.  etc.

Can customize a display name in settings->general tab for bet areleases, etc.

Both my iOS and macOS app products use thes same bundle id by default.  When I publish them to the appstore, they will be made available for universal purchase.

Single entitlement, push settings, etc.

# Resolve build issues
Let's take a look at common build issues
* framework availability
* API availability
```swift
#if canImport(ARKit)
import ARKit
#endif
```

After building, xcode reports a new issue.  A framework that is available on amc, swiftUI, has a feature marked as unavailable.  ex, edit mode
```swift
#if os(iOS)
@Environment(\.editMode) private var editMode
#endif
```

```swift
#if os(iOS)
.onChange(of: editMode?.wrappedValue) { newValue in
    if newValue?.isEditing == false {
        selection.removeAll()
    }
}
#endif
```

```swift
#if os(iOS)
EditButton()
#endif
```


# Customize experience
There will be cases where you watn to refine the experience to what users will expect.  Trimming out iOS only features isn't the end of our journey – all the features of the SDK to play with.

```swift
var thumnailSize: Double {
    #if os(iOS)
    return 120
    #else
    return 80
    #endif
}
```

Add our own UI element to the menubar.  Add a summary and give quick/easy access.

Add a new scheme for menu bar extra.  macOS only.

```swift
#if os(macOS)
MenuBarExtra {
    MiniTruckView(model: model)
} label: {
    Label("Food Truck", systemImage: "box.truck")
}
.menuBarExtraStyle(.window)
#endif
```

Platform experience
* utilize paltform features
* refine choices for new expectations
* rely on swiftUI for best practices
* refer to human interface guidelines


# Publish app
Archive the correct destination, lol

# Wrap up
* support many destinations
* Share code and settings
* Customize where needed
* Archive each product for upload

[[What's new in Xcode]]



* https://developer.apple.com/forums/tags/wwdc2022-110371
* https://developer.apple.com/forums/create/question?&tag1=266&tag2=572030







