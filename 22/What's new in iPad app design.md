#design 

New UIKit and swift updates in iOS 16

# Toolbars 
Organize your app's functionality.
Apps can elevate commonly-used optiosni n the center.

You can enable customizable toolbar, so peopel can curate the items most important to their workflow.  Consider enabling customization.

Toolbar items can be grouped or collapsed.  Keep related items together.  Whent here's nto enough room, these items collapse into the plus button.  

Toolbar items will collapse into overflow menu.  make sure to put important items int he trailing section.

* add more items for common workflows
* Arrange important items in the trailing section
* Enable customization for advanced use cases
* Organize similar items into groups

## toolbar document menu
Consider using the new menu if you are primarily a document app.

Menu should contain *actions that affect the document as a whole*.  
actiosn that take content otuside of your app should be placed under share
directly affect content inside the document should appear in toolbars
# Edit menus

Edit menus appear horiziontally for touch.  When using pointer, shows comprehensive list of options in a vertical layout.  Support both styles for touch and pointer interactions.

* don't remove standard actions like cut, copy, paste.  Important to many workflows
* Organize custom actions close to related system actions.  In iOS 16, notes groups formatting and checklist in the same section.

Can apply to objects on a canvas, not just text.  Apply to any editable content.


# Find and replace
Search for words, phrases, etc.  Consider adding an item inside the overflow menu as well as a system keyboard shortcut.


# navigation
Browser-stye navigation.  ex, files app uses back/forward buttons to browse betwen folders that might have come from different places in the sidebar.  Up to your app to choose the buttons left of title.  Keep to just those navigational buttons, or a sidebar button.

When your app is a complex hierarchy, people jumping between different levels.  File browser, web browser.  If your app has a shallow or flat hierarchy like photos, you might not need browser-styl e navigation because all destinations are already available.

Putting search in top right of nav bar.  When your search bar is used to filter the content that you're lookinga t on the same screen.  It supports suggestions.  recent searches, recommendations, filters.  This is meant for searching content showing below.

To search the whole app, keep that in a search tab.  So people can get to search from wherever they are.
# Selection and menus
Band selection => with the pointer
You still had to use the toolbar to act on the selection.  In iOS 16, we no longer enter editing mode.  Use keyboard modifiers like command and shift to select/deselect, without going into editing mode.  Secondary click to act on all of them together.

Long press to get a context menu for those same actions.  Works well with list as well.  Cmd to select multiple nodes, drag and drop, context menu, etc.

Context menus in empty areas.  Right click to create new folder, or paste event to a location in calendar.

* keyboard focus
* band selection
* multiseelect without modes
* menus for multiple selection
* empty area menus

Use submenus only when you really need them.  On iOS 16, they eopen horizontally when there's space.  

Consider including submenus in your app's context menus to make quick changes such as these.

iPadOS 16 also adds a new control for popup buttons in lists.  Just like any other button, we show a menu to let you choose an option.  This replaces navigation pushes in popovers and modals.  Choose places where you have a well-defined set of options to pick from.  Only use a popup button if your options fit nicely into a menu.  If better as a switch, use a switch.

Consider a "custom" option if you sometimes need more controls.  Your app can reveal additional controls.  You can put an epxlanation underneath the popup button.
# Tables
You may have used a control called a table.  But it's not much of a table if it only has a single column.  iPadOS 16 has a real table.  Tables in swiftui show multiple columns of information, and you sort just by tapping a header.

Sortable tables support all selection features, etc.

Tables for content, not data.  Extended version of listviews.  In fact, when you're resize an app to a compact width, tables switch back into single-column lists.  When they do, we recommend taking deteails from the secondary column and moving that into a secondary line of text within each row, sot he information is still available.  Can use a toolbar button to reveal sort options.

# Wrap up
* Make common and advanced workflows more efficient
* Evaluate your app design in resizable windows and larger screens
* Ensure your app works seamlessly with touch and pointer
[[Designed for iPad]]


* https://developer.apple.com/forums/tags/wwdc2022-10009
* https://developer.apple.com/forums/create/question?&tag1=140&tag2=141&tag3=300&tag4=438030

