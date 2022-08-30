#swiftui #ipados 

Welcome to the second part of the swiftui on ipad team.

In the first part,
* lists ant ables
* selection and menus
* split views
[[SwiftUI on iPad Organize your interface]]

# Toolbars
In swiftui, the toolbar api configures many system apis.  navigation bar, bottom bar, etc.
window toolbar on macOS.

Quick actions to most common features.  Improves the productivity of people using your app.

I've been spending a lot of time thinking about toolbars.  Start by showing you waht I built.

Leading-aligned navigation titles, title menus, title-menu headers, etc.

You might be familiar with features liek toolbar customization, allows peopel to amek toolbars uniquely their own.  


```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItem(placement: .primaryAction) {
            Menu {
                FavoriteToggle(place: $place)
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            } label: {
                Label(
                    "More", 
                    systemImage: "ellipsis.circle")
            }
        }
    }
```

On iOS 16, toolbars can implement these menus on your behalf.  To make the best use of them, restructure the content of your toolbar.

```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItemGroup(placement: .primaryAction) {
            Menu {
                FavoriteToggle(place: $place)
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            } label: {
                Label("More", systemImage: "ellipsis.circle")
            }
        }
    }
```

Inserts individual items for each view in the group.  On iPad/Mac, this is all that's needed to move items into an overflow menu when needed.

Placements define thea rea into which a tolbar is rendered, different placements can resolve to the same area.
* leading
* center
* trailing area

```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItemGroup(placement: .secondaryAction) {
            FavoriteToggle(place: $place)
            AdjustImageButton(place: $place)
            AdjustMapButton(place: $place)
        }
    }
```

```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItemGroup(placement: .secondaryAction) {
            FavoriteToggle(place: $place)
            AdjustImageButton(place: $place)
            AdjustMapButton(place: $place)
        }
    }
    .toolbarRole(.editor)
```

```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItem(placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(placement: .secondaryAction) {
            AdjustImageButton(place: $place)
        }
        ToolbarItem(placement: .secondaryAction) {
            AdjustMapButton(place: $place)
        }
    }
    .toolbarRole(.editor)
```



leading and trailing typically contain controls.  Center contains navigation title.

primaryActionGroup => trailing area.  Most common actions available on a particular screen.  in ioS 16 we add secondaryActions.  Tehse represent less-used controls.  Actions like favoriting and editing.

By default, not visible in toolbar, live in overflow menu.  Can change by using `.toolbarRole` modifier.  This influences the behavior of the toolbar by assigning it a semantic role.  `.editor` will optimize for editing content.  Navigation bar interprets this as a desire to have more space to render toolbar items, moves navigation title from center to leading area.  This places secondary actions in center, before moving to the overflow menu.

In compact this doesn't change, continues to render in the center.

Customizable toolbars.  Now iPadOS supports this as well.
```swift
PlaceDetailContent(place: $place)
    .toolbar {
        ToolbarItemGroup(placement: .primaryAction) {
            FavoriteToggle(place: $place)
            AdjustImageButton(place: $place)
            AdjustMapButton(place: $place)
        }
    }
```



Note that there is no functional difference for this change, customization requires every item int he toolbar to be associated with a unique identifier.

```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            AdjustImageButton(place: $place)
        }
        ToolbarItem(id: "map", placement: .secondaryAction) {
            AdjustMapButton(place: $place)
        }
    }
    .toolbarRole(.editor)
```

Important for these to be unique/consistent across app launches.  SwiftuI persists these isds and uses them to look up the associated view to render.

Unique to customizable toolbars, toolbar items might not be initially present.  These items start their life in the customization popover.  Good option for actions tahta re more useful for specific workflows.

```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            AdjustImageButton(place: $place)
        }
        ToolbarItem(id: "map", placement: .secondaryAction) {
            AdjustMapButton(place: $place)
        }
        ToolbarItem(id: "share", placement: .secondaryAction) {
            ShareLink(item: place)
        }
    }
    .toolbarRole(.editor)
```

[[Meet Transferable]]

showsByDefault: false.  Not initially present.
```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            AdjustImageButton(place: $place)
        }
        ToolbarItem(id: "map", placement: .secondaryAction) {
            AdjustMapButton(place: $place)
        }
        ToolbarItem(id: "share", placement: .secondaryAction, showsByDefault: false) {
            ShareLink(item: place)
        }
    }
    .toolbarRole(.editor)
```

Now it's just in the popover.  Can drag from popover into the bar.

Think about the relationship between toolbar items.  

Now let's od groups.  Controls we move as a group.  Model this relationship using control group.  

```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            ControlGroup {
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            }
        }
        ToolbarItem(id: "share", placement: .secondaryAction, showsByDefault: false) {
            ShareLink(item: place)
        }
    }
    .toolbarRole(.editor)
```

Add/remove these as a unit.  By providing alabel for the control group, the items can collapse into a smalelr menu before moving into the overflow menu.


```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            ControlGroup {
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            } label: {
                Label("Edits", systemImage: "wand.and.stars")
            }
        }
    }
    .toolbarRole(.editor)
```

Adding a new place is an important and ocmmon action.  Add a new button.  This time, I'll use the `.primaryAction` placement.
```swift
PlaceDetailContent(place: $place)
    .toolbar(id: "place") {
        ToolbarItem(id: "new", placement: .primaryAction) {
            NewButton()
        }
        ToolbarItem(id: "favorite", placement: .secondaryAction) {
            FavoriteToggle(place: $place)
        }
        ToolbarItem(id: "image", placement: .secondaryAction) {
            ControlGroup {
                AdjustImageButton(place: $place)
                AdjustMapButton(place: $place)
            } label: {
                Label("Edits", systemImage: "wand.and.stars")
            }
        }
        ToolbarItem(id: "share", placement: .secondaryAction, showsByDefault: false) {
            ShareLink(item: place)
        }
    }
    .toolbarRole(.editor)
```

Important distinction between iOS/macOS.  All items support customization on macOS.  **But on iPadOS, only secondary actions do.**

# Titles and documents
Document groups come with built-in functionality for representing and managing documents.  All this comes ofr free using document groups.

In my app, a place is a document. Even though it's not a document group.  Let's look at how we can show off this relationship in a *non*-document-group based app.


```swift
PlaceDetailContent(place: $place)
    // toolbar customizations ...
    .navigationTitle(place.name)
```
Menu.  Kind of like a file menu on macOS.
```swift
PlaceDetailContent(place: $place)
    // toolbar customizations ...
    .navigationTitle(place.name) {
        MyPrintButton()
    }
```
Pass a binding to your navigation bar, tells it you support editing the title.
```swift
PlaceDetailContent(place: $place)
    // toolbar customizations ...
    .navigationTitle($place.name) {
        MyPrintButton()
    }
```

rename button to start editing.

```swift
PlaceDetailContent(place: $place)
    // toolbar customizations ...
    .navigationTitle($place.name) {
        MyPrintButton()
        RenameButton()
    }
```

Just like you can associate a navigation title to your view, can also associate a document.  Title will render a special header with a preview.  Share, drag and drop, etc.
```swift
PlaceDetailContent(place: $place)
    // toolbar customizations ...
    .navigationTitle($place.name) {
        MyPrintButton()
        RenameButton()
    }
    .navigationDocument(place.url)
```

Proxy icon for the window toolbar on macOS.  
# Wrap up
* toolbar overflow
* toolbar customization
* navigation title menus and renaming
* navigation documents


* https://developer.apple.com/documentation/SwiftUI/Configure-Your-Apps-Navigation-Titles
* https://developer.apple.com/documentation/SwiftUI/ControlGroup
* https://developer.apple.com/documentation/SwiftUI/ShareLink
* https://developer.apple.com/documentation/SwiftUI/ToolbarItem
* https://developer.apple.com/documentation/SwiftUI/ToolbarRole
