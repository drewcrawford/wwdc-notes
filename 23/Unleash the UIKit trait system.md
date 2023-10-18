#uikit 

Discover powerful enhancements to the trait system in UIKit. Learn how you can define custom traits to add your own data to UITraitCollection, modify the data propagated to view controllers and views with trait override APIs, and adopt APIs to improve flexibility and performance. We'll also show you how to bridge UIKit traits with SwiftUI environment keys to seamlessly access data from both UIKit and SwiftUI components in your app.
# Understanding traits
traits

independent pieces of data that the system propagates to every Vc/view.
In iOS 17, you can define your own custom traits.  Powerful new way to provide data to vcs/views.

trait collections -> traits and their associated values.
##  Build a new trait collection instance from scratch - 1:51
```swift
let myTraits = UITraitCollection { mutableTraits in
    mutableTraits.userInterfaceIdiom = .phone
    mutableTraits.horizontalSizeClass = .regular
}
```


##  Get a new instance by modifying traits of an existing one
```swift
let otherTraits = myTraits.modifyingTraits { mutableTraits in
    mutableTraits.horizontalSizeClass = .compact
    mutableTraits.userInterfaceStyle = .dark
}
```

most of the time, you obtain trait collectison from trait environments.
windowscenes, windows, presentation controllers, vcs, views.
every one of these environment has its own trait collection.  Each trait collection may contain different values.
connected in the trait hierarchy.

always use the trait collection of the most specific trait environment possible.  I'll dive deeper into the way traits flow through VCs/views.

previously, VCs got traits from the parent VC.  And views inherited the VC's traits. Views without a VC inherited traits from superview.  This behavior meant that the flow of traits in the view hierarchy stopped at each vc's child.   This could be surprised, so we changed it.

In iOS 17, we unified trait hierarchy for vcs and views.  VCs now inherit from their view's superview instead of parent VC.  This creates a simple linear flow of traits through vcs and views.  Note how vcs still inherit traits from parent vc.  It just happens indirectly via the views inbetween them.

Because they now inherit traits from the vcs, a vc's view must be in the hierarchy to receive updated traits.  If you access a vc's trait collection before view is added to the hierarchy, vcs won't have up-to-date values.  ex `viewWillAppear` is called before the view is added to the hierarchy.  Use `viewIsAppearing` instead.  Called after `viewWillAppear` but once the view is added to the hierarchy and with up-to-date trait collections.  Drop in replacement for `viewWillAppear` today.  New method back-deploys back to iOS 13.

[[23/What's new in UIKit|What's new in UIKit]]

* views only update traits when in the hierarchy
* views update traits before layout
* layoutSubviews is the best place to use traits

# Defining custom traits
new in iOS 17.  New way to provide data to your vcs and views.

* propagate data to many children
* pass data to distant components
* Provide context about environment
	* containing vc, etc.
* use traits when they add value, avoid when you can easily pass data directly.
##  Implementing a simple custom trait - 9:06
```swift
struct ContainedInSettingsTrait: UITraitDefinition {
    static let defaultValue = false
}

let traitCollection = UITraitCollection { mutableTraits in
    mutableTraits[ContainedInSettingsTrait.self] = true
}

let value = traitCollection[ContainedInSettingsTrait.self]
// true
```

get/set values.  think of traits as a key.

adding two simple extensions will let me access the trait usign standard property syntax.  Here, declared a readonly property in an extension.


##  Implementing a simple custom trait with a property - 10:23
```swift
struct ContainedInSettingsTrait: UITraitDefinition {
    static let defaultValue = false
}

extension UITraitCollection {
    var isContainedInSettings: Bool { self[ContainedInSettingsTrait.self] }
}

extension UIMutableTraits
{
    var isContainedInSettings: Bool {
        get { self[ContainedInSettingsTrait.self] }
        set { self[ContainedInSettingsTrait.self] = newValue }
    }
}

let traitCollection = UITraitCollection { mutableTraits in
    mutableTraits.isContainedInSettings = true
}

let value = traitCollection.isContainedInSettings
// true
```


Now I can use standard property syntax.  Write these extension when you define your own custom traits.

Suppose I have support for custom color themes.


##  Implementing a custom theme trait - 11:00
```swift
enum MyAppTheme: Int {
    case standard, pastel, bold, monochrome
}

struct MyAppThemeTrait: UITraitDefinition {
    static let defaultValue = MyAppTheme.standard
    static let affectsColorAppearance = true //use sparingly, very expensive
    static let name = "Theme"
    static let identifier = "com.myapp.theme"
}

extension UITraitCollection {
    var myAppTheme: MyAppTheme { self[MyAppThemeTrait.self] }
}

extension UIMutableTraits {
    var myAppTheme: MyAppTheme {
        get { self[MyAppThemeTrait.self] }
        set { self[MyAppThemeTrait.self] = newValue }
    }
}
```

##  Using a custom theme trait - 12:33
```swift
let customBackgroundColor = UIColor { traitCollection in
switch traitCollection.myAppTheme {
    case .standard:    return UIColor(named: "StandardBackground")!
    case .pastel:      return UIColor(named: "PastelBackground")!
    case .bold:        return UIColor(named: "BoldBackground")!
    case .monochrome:  return UIColor(named: "MonochromeBackground")!
    }
}

let view = UIView()
view.backgroundColor = customBackgroundColor
```
best practices:
* use value types
* most efficient data types: bool, int, double, enum with int raw value.  Make sure to explicitly specify int as the raw datatype.
* custom types must implement `Equatable` efficiently.
* also available in objc
* defintions are separate for swift and objc
* custom trait for each language can access same underlying data
* refer to docs for details

# Applying overrides
modify data within trait hierarchy
new `traitOverrides` propperty on trait environments.
ex windowscene, view, vc, uiwindow, etc.

when you apply a trait override, it modifies the value for that trait in the trait collection of that object and all descendents.

think of trait overrides as optional inputs, and trait collections as the output.

Set override values for custom traits with standard property syntax, using the extension to UIMutable traits I explained earlier.  By setting the theme to pastel, all the windows, etc., get that property.

Because views update trait collection right before layout, modifications to overrides aren't reflected until it runs layoutSubviews.  traitoverrides lalows you to check whether overrides are applied, and remove altogether
##  Managing trait overrides - 18:05
```swift
func toggleThemeOverride(_ overrideTheme: MyAppTheme) {
    if view.traitOverrides.contains(MyAppThemeTrait.self) {
        // There's an existing theme override; remove it
        view.traitOverrides.remove(MyAppThemeTrait.self)
    } else {
        // There's no existing theme override; apply one
        view.traitOverrides.myAppTheme = overrideTheme
    }
}
```

Input mechanism to set values.  To read values, always use the traitCollection property.  Reading from traitOverrides when no override is set, raises an exception.

## performance considerations
* only set overrides where necessary
	* avoid unused overrides
* minimize trait override changes
* apply overrides as deep as possible


# Handling changes
* traitCollectionDidChange is deprecated on iOS 17.  It has to call that method every time any trait changes value.
* most classes only use a handful of traits and don't care about changes to any others.  
* doesn't scale
New trait registration APIS
* register for exact traits you depend on
* receive a callback via target-action or closure
* no need to override a method in a subclass
##  Trait change handling on older iOS versions - 21:00
```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    if traitCollection.horizontalSizeClass != previousTraitCollection?.horizontalSizeClass {
        updateViews(sizeClass: traitCollection.horizontalSizeClass)
    }
}

func updateViews(sizeClass: UIUserInterfaceSizeClass) {
    // Update views for the new size class...
}
```

make sure that your implementation checks specific traits that you care about, if you're backdeploying.

new registration methods
##  Registering for trait changes using a closure - 21:28
```swift
registerForTraitChanges(
    [UITraitHorizontalSizeClass.self]
) { (self: Self, previousTraitCollection: UITraitCollection) in
    self.updateViews(sizeClass: self.traitCollection.horizontalSizeClass)
}

let anotherView: MyView
anotherView.registerForTraitChanges(
    [UITraitHorizontalSizeClass.self, ContainedInSettingsTrait.self]
) { (view: MyView, previousTraitCollection: UITraitCollection) in
    // Handle the trait change for this view...
}
```
closure not called for other traits.  Don't need to compare old values, etc.
use parameter to avoid capturing a weak reference to the object.  Always use `self: Self` here.

##  Registering for trait changes using a target-action - 22:48
```swift
registerForTraitChanges(
    [UITraitHorizontalSizeClass.self],
    action: #selector(UIView.setNeedsLayout)
)

let anotherView: MyView
anotherView.registerForTraitChanges(
    [UITraitHorizontalSizeClass.self, ContainedInSettingsTrait.self],
    target: self,
    action: #selector(handleTraitChange(view:previousTraitCollection:))
)

@objc func handleTraitChange(view: MyView, previousTraitCollection: UITraitCollection) {
    // Handle the trait change for this view...
}
```

can omit target: default target is self.

0/1/2 parameters.
* first parameter: object whose traits are changing
* second parameter: previous trait collection for that object before change
* ?

register for semantic sets of system traits.
##  Registering for changes to system traits affecting color appearance - 24:20
```swift
registerForTraitChanges(
    UITraitCollection.systemTraitsAffectingColorAppearance,
    action: #selector(handleColorAppearanceChange)
)

@objc func handleColorAppearanceChange() {
    // Handle the color appearance trait changes...
}
```

also `imageLookup` (ex UIImage(named)).  Perform custom invalidation.

## Unregistration
* cleaned up automatically
* optional for advanced situations

##  Manually unregistering for trait changes - 24:37
```swift
let registration = registerForTraitChanges([UITraitHorizontalSizeClass.self], action: #selector(handleTraitChange))

unregisterForTraitChanges(registration)

@objc func handleTraitChange() {
    // Handle the trait change...
}
```

these cases are very rare, generally just ignore this

## Maximize perf
* only register for traits you depend on
* invalidate in response to changes (don't update immediately)

# SwiftUI bridging

* bridge UIKit traits and swiftui environment keys
* data propagates across uikit and swiftui boundaries
* works in both directions

##  Implementing a bridged UIKit trait and SwiftUI environment key - 26:19
```swift
enum MyAppTheme: Int {
    case standard, pastel, bold, monochrome
}

struct MyAppThemeTrait: UITraitDefinition {
    static let defaultValue = MyAppTheme.standard
    static let affectsColorAppearance = true
}

extension UITraitCollection {
    var myAppTheme: MyAppTheme { self[MyAppThemeTrait.self] }
}

extension UIMutableTraits {
    var myAppTheme: MyAppTheme {
        get { self[MyAppThemeTrait.self] }
        set { self[MyAppThemeTrait.self] = newValue }
    }
}

struct MyAppThemeKey: EnvironmentKey {
    static let defaultValue = MyAppTheme.standard
}

extension EnvironmentValues {
    var myAppTheme: MyAppTheme {
        get { self[MyAppThemeKey.self] }
        set { self[MyAppThemeKey.self] = newValue }
    }
}

extension MyAppThemeKey: UITraitBridgedEnvironmentKey {
    static func read(from traitCollection: UITraitCollection) -> MyAppTheme {
        traitCollection.myAppTheme
    }

    static func write(to mutableTraits: inout UIMutableTraits, value: MyAppTheme) {
        mutableTraits.myAppTheme = value
    }
}
```

basically you bridge these yourself with these `read` and `write` methods.

use example

##  Setting a UIKit trait and reading the bridged environment value from SwiftUI - 27:01
```swift
let windowScene: UIWindowScene
windowScene.traitOverrides.myAppTheme = .monochrome

let cell: UICollectionViewCell
cell.contentConfiguration = UIHostingConfiguration {
    CellView()
}

struct CellView: View {
    @Environment(\.myAppTheme) var theme: MyAppTheme

    var body: some View {
        Text("Settings")
            .foregroundStyle(theme == .monochrome ? .gray : .blue)
    }
}
```

bridging works in the other direction


##  Setting a SwiftUI environment value and reading the bridged trait from UIKit - 28:16
```swift
// SwiftUI environment value applied to a UIViewControllerRepresentable
struct SettingsView: View {
    var body: some View {
        SettingsControllerRepresentable()
            .environment(\.myAppTheme, .standard)
    }
}

final class SettingsControllerRepresentable: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> SettingsViewController {
        SettingsViewController()
    }
    
    func updateUIViewController(_ uiViewController: SettingsViewController, context: Context) {
        // Update the view controller...
    }
}

// UIKit view controller contained in the SettingsControllerRepresentable
class SettingsViewController: UIViewController {
    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        title = settingsTitle(for: traitCollection.myAppTheme)
    }
    
    func settingsTitle(for theme: MyAppTheme) -> String {
        switch theme {
        case .standard:   return "Standard"
        case .pastel:     return "Pastel"
        case .bold:       return "Bold"
        case .monochrome: return "Monochrome"
        }
    }
}
```

# Next steps
* define your own traits
* adopt improved trait override and registration APIs
* bridge UIKit traits with SwiftUI environment keys
* https://developer.apple.com/documentation/uikit/uimutabletraits
* https://developer.apple.com/documentation/uikit/uitraitdefinition
