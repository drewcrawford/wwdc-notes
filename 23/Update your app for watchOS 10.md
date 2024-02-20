Join us as we update an Apple Watch app to take advantage of the latest features in watchOS 10. In this code-along, we'll show you how to use the latest SwiftUI APIs to maximize glanceability and reorient app navigation around the Digital Crown.

New design paradigms, swiftUI apis, etc.

sample app, code along.  Backyard birds.
Each backyard has a detail view which shows us its condition.  

[[Meet watchOS 10]]
[[Design and build apps for watchOS 10]]

New blur material under navigation bar, blurred modal, etc.  

Since the detail view is the most important part of our app, 
* emphasize the detail view
* best for strong source and detail relationships
* always provide a default selection
### 4:02 - NavigationSplitView
```swift
NavigationSplitView {
    List(backyardsData.backyards, selection: $selectedBackyard) { backyard in
        BackyardCell(backyard: backyard)
    }
    .listStyle(.carousel)
} detail: {
    if let selectedBackyard {
        BackyardView(backyard: selectedBackyard)
    } else {
        BackyardUnavailableView()
    }
}
```

When providing a value for the selection part of a list, binding drives which view is selected.

### 6:18 - Vertical TabView
```swift
TabView {
    TodayView()
        .navigationTitle("Today")
    HabitatGaugeView(level: $waterLevel, habitatType: .water, tintColor: .blue)
        .navigationTitle("Water")
    HabitatGaugeView(level: $foodLevel, habitatType: .food, tintColor: .green)
        .navigationTitle("Food")
    List {
        VisitorView()
            .navigationTitle("Visitors")
    }
}
.tabViewStyle(.verticalPage)
```

* best for distinct full-screen views
* easily interacts with the digital crown
* expand to accommodate any content

If tab exceeds the height of an item, it is scrollable. place in last tab whenever possible.

toolbar has been updated for watchOS 10.

* updated placements are familiar across apps
* enables new ways to add fucntionality
* surfaces interactive elements
### 8:37 - Add refill button to Toolbar
```swift
.toolbar {
    ToolbarItemGroup(placement: .bottomBar) {
        Spacer()
        Button {
            level = Int(min(100, Double(level) + 5))
        } label: {
            Label("Add", systemImage: "plus")
        }
    }
}
```


full-screen backgrounds.

* enhances visible clarity
* reflects important information
* system gradient maintains contrast

If levels are low, bg will be red to indicate it's time for a refill.

Now, the computed property... 


### 9:48 - HabitatGaugeView background color function and variables
```swift
func backgroundColor(_ level: Int, for type: HabitatType) -> Color {
    let color: Color = type == .food ? .green : .blue
    return level < 40 ? .red : color
}

var waterColor: Color {
    backgroundColor(waterLevel, for: .water)
}

var foodColor: Color {
    backgroundColor(foodLevel, for: .food)
}
```

Let's pass in the same computed color variable.  Supply the container bg modifier using the app's bg color.

Gradient contrasts nicely against fg element.  See how the bg gradient changes.  As a final touch, i want to make use of materials to highlight important information.


### 10:10 - .containerBackground within TabView
```swift
TabView {
    TodayView()
        .navigationTitle("Today")
        .containerBackground(Color.accentColor.gradient, for: .tabView)
    HabitatGaugeView(level: $waterLevel, habitatType: .water, tintColor: waterColor)
        .navigationTitle("Water")
        .containerBackground(waterColor.gradient, for: .tabView)
    HabitatGaugeView(level: $foodLevel, habitatType: .food, tintColor: foodColor)
        .navigationTitle("Food")
        .containerBackground(foodColor.gradient, for: .tabView)
    List {
        VisitorView()
            .navigationTitle("Visitors")
            .containerBackground(Color.accentColor.gradient, for: .tabView)
    }
}
.tabViewStyle(.verticalPage)
.environmentObject(backyard)
.navigationTitle(backyard.displayName)
```

* distinguish foreground from bg
* add visual flourish
* surfaces information

### 11:38 - Add material to the backyard name
```swift
.foregroundStyle(.secondary)
.background(Material.ultraThin, in: RoundedRectangle(cornerRadius: 7))
```

### 12:15 - Visitor score overlay with materials
```swift
.overlay(alignment: .topTrailing) {
    Text("\(backyard.visitorScore)")
        .frame(width: 25, height: 25)
        .foregroundStyle(.secondary)
        .background(.ultraThinMaterial, in: .circle)
        .padding(.top, 5)
}
```

### 12:20 - Light materials
```swift
.environment(\.colorScheme, .light)
```

[[Design and build apps for watchOS 10]]
[[Build widgets for the Smart Stack on Apple Watch]]

# Resources
* https://developer.apple.com/design/Human-Interface-Guidelines/designing-for-watchos
* https://developer.apple.com/documentation/watchOS-Apps/updating-your-app-and-widgets-for-watchos-10
* 