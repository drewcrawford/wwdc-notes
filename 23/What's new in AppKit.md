Discover the latest advances in Mac app development. We'll share improvements to controls and menus and explore the tools that can help you break free from your (view) bounds. Learn how to add motion to your user interface, take advantage of improvements to text input, and integrate your existing code with Swift and SwiftUI.

# Controls

TAble column customization
### Configure NSTableView column customization menu - 1:36
```swift
func tableView(_ tableView: NSTableView, 
               userCanChangeVisibilityOf column: NSTableColumn) -> Bool {
    return column.identifier != "Name"
}
```

Toggle visibility of columns in the table.  Would have required a custom implementation to create/present the menu.  Now you can add in as few as 3 loc.


### Configuring NSProgressIndicator to sync with Progress - 1:53
```swift
func fetchData() {
    let url = URL(string: "https://developer.apple.com/wwdc23/")!
    let task = URLSession.shared.dataTask(with: .init(url: url))
    progressIndicator.observedProgress = task.progress
    
    task.resume()
}
```

In macOS sonoma, can now use progress type from foundation with NSProgressIndicator.

NSButton has new bezel style `.automatic`
adapts appropriate style for context, ex window vs toolbar, tall content.  
new default bezel style

We made style names based on sematntic usage.  Discouraged styles are now deprecated.  See deprecation warnings.

Inspectors.  Tailing split view item that displays contextual information aobut selected content.

Use full height of window when full-size content mask is set.  Apply macOS big sur.

### Adding an inspector to your app - 3:48
```swift
let inspectorItem = NSSplitViewItem(inspectorWithViewController: inspectorViewController)
splitViewController.addSplitViewItem(inspectorItem)

func toolbarDefaultItemIdentifiers(_ toolbar: NSToolbar) -> [NSToolbarItem.Identifier] {
    [.toggleSidebar, .sidebarTrackingSeparator, .flexibleSpace, .addPlant, 
     .inspectorTrackingSeparator, .flexibleSpace, .toggleInspector]
}
```

NSPopover
* anchoring to toolbar items
* full-size popover content

### Show a NSPopover relative to a NSToolbarItem - 4:38
```swift
func toolbarAction(_ toolbarItem: NSToolbarItem) {
    let popover = NSPopover()
    popover.contentViewController = PopoverViewController()
    popover.show(relativeTo: toolbarItem)
}
```


`hasFullSizeContent`
Content extends into the chevron
Use safe area for layout
# Menus

We rewrote menus.

## Section headers


## Palette menus
Lay out items in a horizontal series, such as a colorpicker.

Turn any menu into a palette menu by choosing `.palette` for `.presentationStyle`.

Can set offstate/onstate image.  selectAny, selectOne.  


Titles used to AX, make sure you add them.

Closure parameter called on updates.  Can get array of items in on state.  

## SElection behaviors

selectionMode and selectedItems group by target/action pair.  Give each item the same target action pair to take advantage of new selection modes, items.

Alsow orks for menu items with same target action pair, not just palette menus.
## badges

Can use as tring, count, etc. to badge menu tiems.  3 specialized count badges.  new items, alerts, updates.  When using, appkit will automatically add the appropriate text.  Localize as well.

# Cooperative app activation
* activate is a request, not a command
* New yield API

ignoringOtherApps parameter and options are ignored.  Old apis are deprecated.  Replace with activate bare method.

Only active app can influence the activate context.  Does so by yielding to explicit target application b4 the target app is activated.

When target app requests activation, system uses yield as part of a context when making this decision.  If request honored, active app will deactivate and target app will activate.  Otherwise active apps remain active.

# Graphics
Initialize NSBezierPath with CGPath
NSBezierPath.cgPath property.  Initing with, setting, or getting the CGPath alwaysr esults ina  copy of the path.  Further mutations of the bezier patha renot represented in copied instances, not toll-free bridge.

Makes NSBezierPath with CGPath API, a single loc.  

Can now create CADisplayLink on macOS.  Same one on iOS.  
* display synchronized timer
* view, window, or screen creation functions
* follows views across displays
* see docs

Get a CADispalyLink object directly from the most specific element, usually a view.  Because, when created from a view/window, the CADisplayLink will track the display/window, etc.  Including supspending itself when not on a display.

System fill colors.  use a higher level of emphasis to stand out, such as system fill, secondary fill, etc.  Larger shapes like group boxes prefer a more subtle level of emphasis.  These are dynamic, they automatically adapt to different appearances, including increased contrast and dark mode.

Help you fit in with system design and support AX.

Clipped views clip to bounds.  May be a problem for certain fonts.

Ways to solve, ex embed views as siblings in larger views.  In this case, combining the view with a button and simple horizontal stack doesn't line up baselines of text by default.

Now most views no longer clip by default.  hitTest remains unchanged.  Now that we draw outside the bounds, visibleRect may change.  Review code that uses visibleRect and adjust accordingly.

dirtyRect is not constrained by view's bounds.  AppKit reserves the right to pass a larger rect.  Also reserves the right to subdivide drawing into as many rectangle as it needs.  Use the dirtyRect to decide what to draw, not where to draw.

ex use bounds to determine where to draw.

.clipsToBoudns back-deploys to 10.9
Most views are indifferent to clipping
deviate from default with intention

Consider which views require explicit value on a case-by-case basis.  

symbol effects.  Bounce, replacement transitions, etc.
### Adding symbol effects to a image view - 18:30
```swift
wifiImageView.image = NSImage(systemSymbolName: "wifi", accessibilityDescription: "wifi icon")
wifiImageView.addSymbolEffect(.variableColor.iterative, options: .repeating)
```

[[Animate symbols in your app]]

Locale-specific images
Locale-specialized symbols and asset catalog iamges
follows the system locale by default
Access alterantive variants using `image(locale:locale)`.  

HDRContent in NSImageView.

To display HDR content in standard dynamic range, use `preferredImageDynamicRange = .standard`.

[[Support HDR images in your app]]

Asset catalog resources.  Automatically created symbols for asset catalog assets.
Compiler catches catalog changes.
non-optional to remove your guard checks.
# Text improvements

The isnertion indicator has a brand new look.  Leaves a trailing glow as you dictate.

Cursor accessory.  Displays key info such as input mode, capslock state, etc.  Tracks current insertion position and will be pinned to bottom of the document.

Add `NSTextInsertionIndicator` as a subview
Use `displayMode` to manage visibility

Wrapping and hyphenation.macOS sonoma will perform different line breaking based on text style and font, ex in korea we don't want to wrap titles probably.

Now we hyphenate at morpheme boundaries in languages such as german.

# Swift and SwiftUI

More things are sendable.  Certain appkit classes such as NSColor and NSShadow, in Sonoma these conform to Sendable to transfer across actor boundaries.

Transferable is a swift protocol to describe serialization/deserialization.  This year we added NSImage, NSColor, NSSound.

Remove optionals for view loading.  `loadView` and `loadWindow` are called on access.  By using `@ViewLoading` ?
### Using @ViewLoading to remove optionality on properties - 24:56
```swift
class ViewController: NSViewController {
    @ViewLoading var datePicker: NSDatePicker
    var date = Date() {
        didSet {
            datePicker.dateValue = date
        }
    }

    override func loadView() {
        super.loadView()
        datePicker = NSDatePicker()
        datePicker.dateValue = date
        view.addSubview(datePicker)
    }
}
```

Xcode previews.  [[Build programmatic UI with Xcode Previews]]

### Preview NSView and NSViewController using the Preview macro - 25:26
```swift
#Preview("Tree Species") {
    let treeCellView = TreeCellView()
    treeCellView.species = .spruce
    return treeCellView
}
```

NSHOstingView can manage aspects of windows
Default if hosting view is window's contentView
Can be changed with `sceneBridgingOptions` property.

# Next steps
* audit for clipping issues
* Adopt new controls API
* Add symbol effects to your views
* Take advantage of better SwiftUI interoperability
* 
# Resources