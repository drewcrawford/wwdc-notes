#swiftui #ipados 

A number of updates to build more productive, professional-grade apps.  In tihs session, I'm going to discuss a few features.

# Lists and tables
In iPadOS 16 there's a great solution: multi-column tables.

First introduced in monterey.  Now available on iPad, same api as on the mac.

SwiftuI n ow supports section on both platforms.  

[[SwiftUI on the Mac Build the fundamentals]].  Still applies on iPad.

```swift
struct PlacesList: View {
    @Binding var modelData: ModelData


    var body: some View {
        List(modelData.places) { place in
            PlaceCell(place)
        }
    }
}
```

switch to a table.  Different structure, instead ofa  view builder, they accept a column builder.

```swift
struct PlacesTable: View {
    @Binding var modelData: ModelData     @State private var sortOrder = [KeyPathComparator(\Place.name)]

    var body: some View {
        Table(modelData.places, sortOrder: $sortOrder) {
            TableColumn("Name", value: \.name) { place in
                PlaceCell(place)
            }
            TableColumn("Comfort Level", value: \.comfortDescription).width(200)
            TableColumn("Noise", value: \.noiseLevel) { place in
                NoiseLevelView(level: place.noiseLevel)
            }
        }
        .onChange(of: sortOrder) {
            modelData.sort(using: $0)
        }
    }
}
```

I can even re-use the place cell type from before.  In compact size classes, tables onyl show the first column.

Omit the viewbuilder when my value points to a string.

Sort order to store comparators.  All comparators for the table.  

need to use `.onChange` to implement sorting.

Tables **don't** scroll automatically, need to limit the number of columns.

In slideover, collapses into a single column.  

# Selection and menus
Includes a robust API for managing list and table selection.

EAch row has a tag.  Unique values to help the list manage selection.  Here the tags are shown in green circles.

Also some state that holds the selection.  Job of the list is to coordinate between tag and selection state.  

Where do tags come from?  
* hashable value associated with a view.
* can be synthesized automatically
* similar to identifiers but not quite the smae

ForEach will automatically tag content with its explicit identity
table will automatically tag content with the row value identity
`.id`. does not set the tag
[[Demystify SwiftUI]]

Manual tags
* Use `View.tag(_:)`.
	* foreach does this under the hood
* tag type is important
* Use only one tag type in each selectable container
* id does not set tag!!!

selection datastructure
|                                         | data structure     | use case   |
| --------------------------------------- | ------------------ | ---------- |
| single selection                        | Optional<Place.ID> | Navigation |
| single required selection (macos only) | Place.ID           | sidebars   |
| Multiple selection                      | Set<Place.ID>      | Bulk operations           |

* lightweight multiple selection with keyboard
	* works with common shortcuts, like shift and command
* edit mode without keyboard

single selection updates
* no longer requires edit mode
* works with updated navigation APIs

selection overview
|                                        | data structure     | use case        | edit mode required? |
| -------------------------------------- | ------------------ | --------------- | ------------------- |
| single selection                       | Optional<Place.ID> | navigation      | no                  |
| Single required selection (macos only) | Place.ID           | sidebars        | -                   |
| multiple selection                     | Set<Place.ID>      | bulk operations | without keyboard                    |
```swift
struct PlacesTable: View {
    @EnvironmentObject var modelData: ModelData
    @State private var sortOrder = [KeyPathComparator(\Place.name)]
    @State private var selection: Set<Place.ID> = []

    var body: some View {
        Table(modelData.places, selection: $selection, sortOrder: $sortOrder) {
            // columns
        }
    }
}
```

edit controls in toolbar button.

```swift
Table(modelData.places, selection: $selection, sortOrder: $sortOrder) {
    ...
}
.toolbar {
    ToolbarItemGroup(placement: .navigationBarTrailing) {
        if !selection.isEmpty {
           AddToGuideButton(selection)
        }
    }

    ToolbarItemGroup(placement: .navigationBarLeading) {
        EditButton()
    }
}
```

Multiselect context menus.  Operate on a set of identifiers.

* multiple items
* single item
* empty area!

```swift
// Item context menus

Table(modelData.places, selection: $selection, sortOrder: $sortOrder) {
    ...
}
.contextMenu(forSelectionType: Place.ID.self) { items in
    if items.isEmpty {
        // Empty area
        AddPlaceButton()
    } else {
        if items.count == 1 {
            // Single item
            FavoriteButton(isSet: $modelData.places[items.first!].isFavorite)
        }

        // Single and multiple items
        AddToGuideButton(items)
    }
}
```

# Split Views
In the previous sections, I created the places atable and added rich features.  But we're lacking structure.  Build the foundation of our app's structure.
navigation split views:
* supported on iOS, iPadOS, and macOS
* two or three-column layouts
* multiple styles
[[The SwiftUI cookbook for navigation]]

Sidebar column, detail column.  In landscape, swiftui offers this arrangement.
In portrait, we see only the detail column.  Tapping on sidebar shows the sidebar over the detail column which is dimmed underneath

* the detial column is often more important
* change style if weighting of columns differs
* Use `.balanced` to even priority between sidebar and detail
* use `.prominentDetail` to prefer detail always

With three column, we have an additional column between, called `content`.  In UIKit this is the suplementary.

In landscape, we show `Detail` and `Content`, and `Sidebar` can be toggled.  Detail slides out of the way.

In portrait, we only show Detail.  Tapping on toolbar shows Content.  Tapping again can show Sidebar.  Sidebar/Content overlay the Detail.

* prefer automatic style with three columns
* Specialized for larger iPads and macOS
* collapses to stack in compact size classes

# Wrap up
* leverage tables for rich display of data
* use item-based context menus
* avoid modality with split views
[[SwiftUI on iPad Add toolbars, titles, and more]]


* https://developer.apple.com/forums/tags/wwdc2022-10058
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=141&tag3=485030
* https://developer.apple.com/documentation/SwiftUI/NavigationSplitViewStyle
* https://developer.apple.com/documentation/SwiftUI/View/contextMenu(menuItems:preview:)
* https://developer.apple.com/documentation/SwiftUI/EditMode
* https://developer.apple.com/documentation/SwiftUI/Tables
* https://developer.apple.com/documentation/SwiftUI/List
* https://developer.apple.com/documentation/SwiftUI/NavigationSplitView







```swift
// Navigation Split View example

struct ContentView: View {
    var body: some View {
        NavigationSplitView {
            SidebarView()
       } detail: {
            Text("Select a place")
        }
    }
}
```

