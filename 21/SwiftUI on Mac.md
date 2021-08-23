#swiftui  #macOS 

* Flexible
	* Adjusting how each of us use them
	* Keyboards/mice/trackpads/switch controls/iPads
* Familiar
	* Using controls and design patterns consistent with the system makes an app approachable.
	* Search bar has a consistent look
	* Consistency can still leave room for apps to be unique
* Expansive
	* Large, multiple displays.  More info can be organized onscreen.
	* Sidebars, previews, tabs, disclosure groups.
* Precise
	* Title margins and spacing
	* Simple, single purpose

https://developer.apple.com/documentation/swiftui/building_a_great_mac_app_with_swiftui

Cmd-click to extract this view into its own subview.

Table offers precise way to view, filter, sort, data.  

Menus are nice.

# Part two
* Accent color
* Drag and drop
* Data export
* Continuity camera

# Wrap up
* Sidebar
* Table
* toolbar
* Main menu

https://developer.apple.com/documentation/swiftui/building_a_great_mac_app_with_swiftui

# Part two
Use our apps in many different ways.  a particularly great macos app will account for this.

## Customization
Developers to configure an app-specific accent color.  OS will use this for buttons, selection highlight, etc.

Set AccentColor in asset catalog.

We've seen how our app automatically reacts to changes which affect the whole OS.  What about app specific settings?

On macOS it's common to provide tab icons for settings.  

`@AppStorage` to persist selection value using userdefaults.


## Additional workflows
Drag and drop

Great way to move data around *inside* our app.  What about between apps?
## File handling
`CommandGroup(replacing: .importExport)` will add our menu item in the expected place of the menu.

Elipses indicate we will ask for more options.


## Continuity

# Resoruces
https://developer.apple.com/documentation/swiftui/building_a_great_mac_app_with_swiftui


