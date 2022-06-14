#appkit 

# stage manager
cleans up inactive windows, while active window takes center stage.

can have sets
* newly presented windows replace those on stage
* auxiliary winodws cohabitate with primary windows
	* panels, popvers, settings
* behavior respects existing NSWindow APIs

we ignore
Floating panels
modal presentations
windows with toolbarStyle set to `.preference`
Windows with `collectionBehavior` flags.  `.auxiliary`, `.moveToActiveSpace`, `stationary`, or `transient`

# preferences
new design, renamed to system settings.

Custom settings bundles
In-app settings
Updated form design

## bundles

custom panes appear in the sidebar
plugin bundle is loaded on first access
interface is presented in the content area

## renaming
rename preferences to "Settings"
"Preferences..." menu is automatically updated
Check for window titles, labels, and controls

## design
New design alnguage for control-dense interfaces
many system controls draw with lower visual weight
```.formStyle(.insetGrouped)```

Give SwiftUI a try!

[[Use SwiftUI with AppKit]]

# Controls
## NSComboButton
Button?  Pull-down button?  Why not both?
Primary action + pull-down together.  Accept given selection or choose somethign with menu.

Previously, you might try this with segmented control but now we have a dedicated control.

`.split` => separate arrow portion for the menu
`.unified` => Looks like an ordinary button.  Menu is click+hold.
## NSColorWell
Color well has adopted a new style reminscent of other button bezels across the system.  It's automatic.

New styles.  Minimal style => disclosure arrow shown on rollover.  System standard grid of colors, can be customized if desired

Expanded style => like iWork.  Has quick picker and expanded picker segments.

Also the default style as original.

New target/action for pulldown.  By default, these are `nil`, use system standard popover.  You can customize with your own custom popover.

## NSToolBar
* more control over customization
	* Can prevent toolbars from being moved ("immovable")
	* `canBeInsertedAt` => veto power over reordering, insertion, removal, etc.
	* 
* multiple centered items
	* `centeredItemIdentifiers`.  Added or removed, but can only be reordered "within the center group"
* sizing for alternate labels
	* Since "mute" vs "unmute" have different sizes, things have to shift around.
	* Use the new `possibleLabels` to provide set of localized strings that you'll use.

## NSAlert
alerts work best with shorter text.  People will read them.  etc.

But what if you have something complex and subtle?  We've adapted `NSAlert` for a wider layout.  This happens automatically if your alerts are too long.

Will also use this style if your accessory view is too large to fit.

Wont' change if it's already on screen during mutation.

## NSTableView
To provide a good scrolling experience, table needs to provide its height and the location of each row, etc.  Historically, it does this by sizing every row.

In ventura, we lazily calculate row heights based on which rows are within or near the viewport.  For rows that haven't been measured, we use a running estimated height.  

significantly improves table load times.  Does alter timing of `tableView(_:,, heightOfRow)`.  Applies to `NSTableView` and swiftui `List`.  No adoption required.
# SF Symbols
SF Symbols 4.  450 new symbols.  

Rendering modes
* monochrome
* hierarchical (opacities)
* palette (distinct colors for each part)
* multicolor (colors in the symbol itself)

* symbols may have a *preferred rendering mode*
* Used automatically without any configuration
* Use `NSImageSymbolConfiguration` to override

## Variable symbols
Some symbols represent value or quanitty, signal strength, volume, etc
New type of drawing using numeric thresholds
Specify the value directly in the API

`init(syumbolName: String, variableValue: Double, ...)`

variable symbols work great in combination with palette color and multicolor.

# sharing
* all-new sharing experience on macOS
* Easily invite contacts to collaborate

## `NSSharingServicePicker`
* New sharing picker
* Header displays more info about the share
* Suggested contacts and services

* header automatically populated for shared files
* New protocol to provide metadata for custom shared types

`NSPreviewRepresentableActivityItem`
* item (e.g. NSItemProvider)
* title
* imageProvider
* iconProvider

`NSPreviewRepresentingActivityItem` to bundle up an existing sharing item with its metadata.  Might be performance-intensive to generate images upfront.

## Sharing menu item
`class NSSharingServicePicker` => use instead of building a custom sharing submenu


* start collaborating via share sheet, drag-and-drop, and facetime
* integrate your ucstom collaboration system with iMessage
* Support for sharing via CloudKit and iCloud Drive

[[Enhance collaboration experiences with Messages]]
[[Integrate your custom collaboration app with Messages]]

# Next steps
* get ready for stage manager
* Enhance your design with the latest controls
* use variable symbols and expanded symbol library
* adopt the latest sharing and collaboration APIs

