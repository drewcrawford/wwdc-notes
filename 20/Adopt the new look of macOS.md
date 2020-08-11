#macOS 

# Sidebar and Toolbar
`NSSplitViewItem.Behavior.sidebar`
`NSWindow.StyleMask.fullSizeContentView`

"new" safe areas

```swift
class NSView {
	var safeAreaInsets
	var safeAreaLayoutSuides
	var saveAreaRect
}
```
You may want to opt out of sidebar when
* sidebar is typically collapsed
* toolbar is crowded

`NSSplitViewItem.allowsFullHeightLayout`

## Icon colors
```swift
outlineView(_ outlineView: NSLoutlineView, tintConfigurationForItem:)
```

Distinguish sections by function or prority
Give specific items a representative color
Use `.monochrome` to de-emphasize groups

## Toolbars
This is full-width, even for windows that are divided or otherwise appear to not be ful-width.

`NSWindow.toolbarStyles` "purpose built designs"

`.unified`
`.unifiedCompact`
use when 
* toolbar is sparse
* focus is on content

`.preference`
`.expanded` -> previously 'standard' toolbar, use when
* window title is long
* toolbar is heavily populated

`.automatic` default value , resolves based on window structure.

## controls in toolbar
* Text controls receive stroke border
* Define clickable area

`NSToolbarItem.minSize / maxSize` are deprecated

## Window subtitle
Great for seondary text
Saves safe in unified styles
Looks good in expanded style


## Navigational toolbar items
Leading edge of inline title
`.isNavigational`

## `NSSearchToolbarItem`
When the window shrinks, we collapse the search field into a button.
Supports existing search field
High visibility priority

When your app is run on an older system, a standard search field will be used.  NO runtime checks needed.

## Sections
Sections the toolbar
Content is customizable
Not removable
`NSTrackingSeparatorToolbarItem`

Allows positioning items over sidebar
`NSToolbarItem.Identifier.sidebarSeparator`.  

Shadow automatically appears over scrolled content.  "Titlebar separator"
NSScrollView must fill frame.  Requires `.fullSizeContentView`.  

If you'd like to request a separator instead of scroll shadow, or customize, `NSSplitViewItem.titleBarSeparatorSize`.  Or specify in `NSWindow`.

## Adopting system classes
Use `NSToolbar` and `NSSplitViewcontroller`

# Controls
## Accent colors
When this multicolor is chosen in system preferences, we use your custom color.
* Controls
* table selections
* sidebar icons
* menus
* focus rings

Define in your asset catalog.

## Custom accent color
Only when syspref is multicolor
People might prefer a different accent color
named system colors always match the preference
`NSColor.controlAccentColor`, etc.

## Large controls
Somtimes you just need a big button.
`button.controlSize = .large`

Unified toolbars -> Uses large control size
System alerts

## Sliders
New look
Unified layout for tick marks
Updated thumb shape/size

Baseline-align your label to the slider, because the height has changed.

## Table View styles
* New inset style
* New `style` proeprty
* Extra padding around cells
* Taller row default heights
* Wider spacing between columns

`.automatic` -> default
`.fullWidth` -> displays the selection edge-to-edge like catalina
`.inset`-> new style
`.sourceList` -> new appearance of sidebar source lists.

`.effectiveStyle` -> resolves automatic

| In sidebar    | bordered     | all others |
|---------------|--------------|------------|
| `.sourceList` | `.fullWidth` | `.inset`   |

## Text styles
Have come to macOS.  Sizes/weights are adjusted for this platform.
Not the same thing as dynamic type -> no slider that will adjust these system-wide.

# Symbol images

SFSymbols on the mac!
* Scale to different font sizes
* Adapt to various weights
* Toolbar automatically configure themselves for toolbar size/weight

Create with new image initializer

Prefer `NSImageView`

Knows how to do baseline.  

## Manually-drawn images
Image sits inside bounding box related to size
`image.alignmentRect` -> baseline to cap height and full width
Especially avoid using `NSImage` directly in layer contents.  Once an image is in layer contents, it loses its context.  Difficult to properly size the symbol.

Existing system images now map to symbols
Drawing and layout may be different
Only for apps built with new SDK

# Next steps
* Full-height sidebar
* Update toolbar and title bar
* Define a custom accet color
* Adopt symbol images

