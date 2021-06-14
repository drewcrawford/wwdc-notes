#catalyst

# Shims

Many iOS frameworks weren't available to Catalyst apps.  We've worked hard to bring over new frameworks.

If your application is dependent on ARKit, previously you would need to remove the framework.  Now, if you're targeting macOS big sur or later, it works the same as an iOS device that doesn't support AR.

# New APIs
Keyboard events.  `UIREsponder.pressesBegan` `ended` etc.
[[Support Hardware keyboards in your apps]]

Focus engine is now available.

UITableView/UIcollectionView `.selectionFollowsFocus`

Specify the window creation behavior

`NSCursor` was exposed to Mac catalyst, e.g. `hide` or `show`.

Color picker and color well.  In mac catalyst, this brings up the standard color picker.
UIDatePicker now uses the system date picker

[[Design with iOS pickers, menus, and actions]]

Sheets are now presented in their own `NSWindow`.  
Popovers are now presented in `NSPopover`
`UISplitViewController` now supports 3 columns, the leading column can be a sidebar.

# optimize ui

"Optimized for Mac" is a new mode
Changes the default scaling from 77% to 100%.  More crisp text and graphics, at the cost of disturbing your app's layout.

[[Optimize the interfaces of your Mac Catalyst app]]

# SwiftUI
If your app is written in SwiftUI, you don't need catalyst.

SwiftUI has a number of new features that work alongside, for ios apps that use swiftui

* SwiftUI standard commands.
* Toolbar support
* optimized for mac

[[20/what's new in swiftui]]

# Application lifecycle

On both iPad and mac, an app infront of the user is in foreground state.

[[taking iPad apps for mac to the next level - 19]]

We added more situatiosn where teh app may transition to the background state, when the user is perceiving it to be running.

e.g., when a window becomes minimized, window in a background space, app hidden, etc.

Your app will not transition to the background when it gains or loses the menubar or when it becomes occluded.

Shoudl your app pause when it moves to the background?

# Extensions
Photo editing extensions are now available on mac.

ios-style memory limits and memory pressure control

Widgets also work

[[Meet widgetkit]]

# Universal Purchase

When you bring your iPad app over to the mac, your existing users no longer have to purchase your app a second time.

All new mac catalyst apps are opted in, and may opt out if they choose.

If you have a non-catalyst mac app, or have an existing app and wish to adopt, see docs.

# Mac Catalyst and the New Look of macOS

New styles - unified, compact, expanded, preference.  
[[adopt the new look of macOS]]


Note that we use separators to control how toolbars are laid out

## Accent colors
Create a symbolic color in the assets catalog and then set as the color.
Use this accent color as an app-wide color.

## Sidebar

Sidebars can do drag reordering for collectionviews.  Use collectionviews where possible.

