iPadOS 18 introduces a new navigation system that gives people the flexibility to choose between using a tab bar or sidebar. The newly redesigned tab bar provides more space for content and other functionality. Learn how to use SwiftUI and UIKit to enable customization features – like adding, removing and reordering tabs – to enable a more personal touch in your app.

### TabView updates in SwiftUI - 4:27
```swift
TabView { 
    Tab("Watch Now", systemImage: "play") { 
        WatchNowView() 
    } 
    Tab("Library", systemImage: "books.vertical") { 
        LibraryView() 
    } 
    // ... 
}
```

### UITabBarController updates in UIKIt - 4:58
```swift
tabBarController.tabs = [ 
    UITab(title: "Watch Now", image: UIImage(systemName: "play"), identifier: "Tabs.watchNow") { _ in 
        WatchNowViewController() 
    }, 
    UITab(title: "Library", image: UIImage(systemName: "books.vertical"), identifier: "Tabs.library") { _ in 
        LibraryViewController() 
    }, 
    // ... 
]
```

### Search tab - 5:58
```swift
// Insert code snippet.
```

### Adding a sidebar in SwiftUI - 6:41
```swift
TabView { 
    Tab("Watch Now", systemImage: "play") { 
        // ... 
    } 
    Tab("Library", systemImage: "books.vertical") { 
        // ... 
    } 
    // ... 
    TabSection("Collections") { 
        Tab("Cinematic Shots", systemImage: "list.and.film") { 
            // ... 
        } 
        Tab("Forest Life", systemImage: "list.and.film") { 
            // ... 
        } 
        // ... 
    } 
    TabSection("Animations") { 
        // ... 
    } 
    Tab(role: .search) { 
        // ... 
    } 
} 
.tabViewStyle(.sidebarAdaptable)
```

### Adding a sidebar in UIKit - 7:16
```swift
let collectionsGroup = UITabGroup( 
    title: "Collections", 
    image: UIImage(systemName: "folder"), 
    identifier: "Tabs.CollectionsGroup", 
    children: self.collectionsTabs()) { _ in 
    // ... 
} 
tabBarController.mode = .tabSidebar 
tabBarController.tabs = [ 
    UITab(title: "Watch Now", ...) { _ in 
        // ... 
    }, 
    UITab(title: "Library", ...) { _ in 
        // ... 
    }, 
    // ... 
    collectionsGroup, 
    UITabGroup(title: "Animations", ...) { _ in 
        // ... 
    }, 
    UISearchTab { _ in 
        // ... 
    }, 
]
```

### Updating a tab group in UIKit - 7:35
```swift
let collectionsGroup = UITabGroup( 
    title: "Collections", 
    image: UIImage(systemName: "folder"), 
    identifier: "Tabs.CollectionsGroup", 
    children: self.collectionsTabs()) { _ in 
    // ... 
} 
let newCollection = UITab(...) 
collectionsGroup.children.append(newCollection)
```

### Sidebar actions - 7:45
```swift
TabSection(...) { 
    // ... 
} 
.sectionActions { 
    Button("New Station", ...) { 
        // action 
    } 
} 
// UIKit 
let tabGroup = UITabGroup(...) 
tabGroup.sidebarActions = [ 
    UIAction(title: "New Station", ...) { _ in 
        // action 
    }, 
]
```

### Drop destinations in SwiftUI - 8:12
```swift
Tab(collection.name, image: collection.image) { 
    CollectionDetailView(collection) 
} 
.dropDestination(for: Photo.self) { photos in 
    // Add 'photos' to the specified collection 
}
```

### Drop destinations in UIKit - 8:24
```swift
func tabBarController( 
    _ tabBarController: UITabBarController, 
    tab: UITab, 
    operationForAcceptingItemsFrom dropSession: any UIDropSession 
) -> UIDropOperation { 
    session.canLoadObjects(ofClass: Photo.self) ? .copy : .cancel 
} 

func tabBarController( 
    _ tabBarController: UITabBarController, 
    tab: UITab, 
    acceptItemsFrom dropSession: any UIDropSession
) { 
    session.loadObjects(ofClass: Photo.self) { photos in 
        // Add 'photos' to the specified collection 
    } 
}
```

### TabView customization in SwiftUI - 10:45
```swift
@AppStorage("MyTabViewCustomization") private var customization: TabViewCustomization 

TabView { 
    Tab("Watch Now", systemImage: "play", value: .watchNow) { 
        // ... 
    } 
    .customizationID("Tab.watchNow") 

    // ... 

    TabSection("Collections") { 
        ForEach(MyCollectionsTab.allCases) { tab in 
            Tab(...) { 
                // ... 
            } 
            .customizationID(tab.customizationID) 
        } 
    } 
    .customizationID("Tab.collections") 

    // ... 
} 
.tabViewCustomization($customization)
```

### Customization behavior and visibility in SwiftUI - 11:40
```swift
Tab("Watch Now", systemImage: "play", value: .watchNow) { 
    // ... 
} 
.customizationBehavior(.disabled, for: .sidebar, .tabBar) 

Tab("Optional Tab", ...) { 
    // ... 
} 
.customizationID("Tab.example.optional") 
.defaultVisibility(.hidden, for: .tabBar)
```

### Tab customization in UIKit - 12:38
```swift
let myTab = UITab(...) 
myTab.allowsHiding = true 
print(myTab.isHidden) 

// .default, .optional, .movable, .pinned, .fixed, .sidebarOnly 
myTab.preferredPlacement = .fixed 

let myTabGroup = UITabGroup(...) 
myTabGroup.allowsReordering = true 
myTabGroup.displayOrderIdentifiers = [...]
```

### Observing customization changes in UIKit - 12:39
```swift
func tabBarController(_ tabBarController: UITabBarController, visibilityDidChangeFor tabs: [UITab]) { 
    // Read 'tab.isHidden' for the updated visibility. 
} 

func tabBarController(_ tabBarController: UITabBarController, displayOrderDidChangeFor group: UITabGroup) { 
    // Read 'group.displayOrderIdentifiers' for the updated order. 
}
```

# REsources
* https://developer.apple.com/documentation/visionOS/destination-video
* https://developer.apple.com/documentation/uikit/app_and_environment/elevating_your_ipad_app_with_a_tab_bar_and_sidebar
* https://developer.apple.com/documentation/SwiftUI/Enhancing-your-app-content-with-tab-navigation
