#swiftui 

Growing alongside our OS.  We continue to be amazed and delighted by what you are making with swiftui.  All flavors of feedback.

This comprehensive adoption within apple further pushes evolution of swiftUI.  Many of these new designs and features are only possible because of how swiftui has evolved how we write apps at apple.

# Swift Charts
#swiftcharts 

Declarative framweork for building beautiful, state-driven charts.  Fundamental design principles that make swiftui great, and the process of plotting data have been composed.

```swift
import Foundation
import SwiftUI

// MARK: - Party Planner Models
enum PartyTask: String, Identifiable, CaseIterable, Hashable {
    case food = "Food"
    case music = "Music"
    case supplies = "Supplies"
    case invitations = "Invitations"
    case eventDetails = "Event Details"
    case activities = "Activities"
    case funProjection = "Fun Projection"
    case vips = "VIPs"
    case photosFilter = "Photos Filter"

    var name: String { rawValue }
  
   var color: Color {
       switch self {
       case .food:
            return palette[0]
       case .supplies:
            return palette[1]
       case .invitations:
            return palette[2]
       case .eventDetails:
            return palette[3]
       case .funProjection:
            return palette[4]
       case .activities:
            return palette[5]
       case .vips:
            return palette[6]
       case .music:
            return palette[7]
       case .photosFilter:
            return palette[8]
       }
    }

    var imageName: String {
        switch self {
        case .food:
            return "birthday.cake"
        case .supplies:
            return  "party.popper"
        case .invitations:
            return "envelope.open"
        case .eventDetails:
            return "calendar.badge.clock"
        case .funProjection:
            return "gauge.medium"
        case .activities:
            return "bubbles.and.sparkles"
        case .vips:
            return "person.2"
        case .music:
            return  "music.mic"
        case .photosFilter:
            return "camera.filters"
        }
    }

    var id: String { rawValue }

    var subtitle: String {
        switch self {
        case .food:
            return "Apps, 'Zerts and Cakes"
        case .supplies:
            return "Streamers, Plates, Cups"
        case .invitations:
            return "Sendable, Non-Transferable"
        case .eventDetails:
            return "Date, Duration, And Placement"
        case .funProjection:
            return "Beta — How Fun Will Your Party Be?"
        case .activities:
            return "Dancing, Paired Programing"
        case .vips:
            return "User Interactive Guests"
        case .music:
            return "Song Requests & Karaoke"
        case .photosFilter:
            return "Filtering and Mapping"
        }
    }

    var emoji: String {
        switch self {
        case .food:
            return "🎂"
        case .music:
            return "🎤"
        case .supplies:
            return "🎉"
        case .invitations:
            return "📨"
        case .eventDetails:
            return "🗓"
        case .funProjection:
            return "🧭"
        case .activities:
            return "💃"
        case .vips:
            return "⭐️"
        case .photosFilter:
            return "📸"
        }
    }
}

private let palette: [Color] = [
    Color(red: 0.73, green: 0.20, blue: 0.20),
    Color(red: 0.95, green: 0.66, blue: 0.24),
    Color(red: 0.14, green: 0.29, blue: 0.49),
    Color(red: 0.46, green: 0.76, blue: 0.67),
    Color(red: 0.30, green: 0.33, blue: 0.22),
    Color(red: 0.49, green: 0.55, blue: 0.64),
    Color(red: 0.92, green: 0.53, blue: 0.30),
    Color(red: 0.20, green: 0.45, blue: 0.55),
    Color(red: 0.41, green: 0.45, blue: 0.45),
    Color(red: 0.87, green: 0.67, blue: 0.61)
]

// MARK: - Swift Charts Models

struct RemainingPartyTask: Identifiable {
    let category: PartyTask
    let date: Date
    let remainingCount: Int

    let id = UUID()
}

let remainingSupplies: [RemainingPartyTask] = [
    RemainingPartyTask(category: .supplies, date: .daysAgo(4), remainingCount: 10),
    RemainingPartyTask(category: .supplies, date: .daysAgo(3), remainingCount: 11),
    RemainingPartyTask(category: .supplies, date: .daysAgo(2), remainingCount: 9),
    RemainingPartyTask(category: .supplies, date: .daysAgo(1), remainingCount: 4),
    RemainingPartyTask(category: .supplies, date: .daysAgo(0), remainingCount: 1),
]

let remainingInvitations: [RemainingPartyTask] = [
    RemainingPartyTask(category: .invitations, date: .daysAgo(4), remainingCount: 14),
    RemainingPartyTask(category: .invitations, date: .daysAgo(3), remainingCount: 13),
    RemainingPartyTask(category: .invitations, date: .daysAgo(2), remainingCount: 11),
    RemainingPartyTask(category: .invitations, date: .daysAgo(1), remainingCount: 6),
    RemainingPartyTask(category: .invitations, date: .daysAgo(0), remainingCount: 4),
]

let remainingActivities: [RemainingPartyTask] = [
    RemainingPartyTask(category: .activities, date: .daysAgo(4), remainingCount: 6),
    RemainingPartyTask(category: .activities, date: .daysAgo(3), remainingCount: 7),
    RemainingPartyTask(category: .activities, date: .daysAgo(2), remainingCount: 4),
    RemainingPartyTask(category: .activities, date: .daysAgo(1), remainingCount: 2),
    RemainingPartyTask(category: .activities, date: .daysAgo(0), remainingCount: 1),
]

let remainingVenue: [RemainingPartyTask] = [
    RemainingPartyTask(category: .eventDetails, date: .daysAgo(4), remainingCount: 4),
    RemainingPartyTask(category: .eventDetails, date: .daysAgo(3), remainingCount: 5),
    RemainingPartyTask(category: .eventDetails, date: .daysAgo(2), remainingCount: 7),
    RemainingPartyTask(category: .eventDetails, date: .daysAgo(1), remainingCount: 4),
    RemainingPartyTask(category: .eventDetails, date: .daysAgo(0), remainingCount: 2)
]

let partyTasksRemaining: [RemainingPartyTask] = [remainingVenue,
                                                remainingActivities,
                                                remainingInvitations,
                                                remainingSupplies
].flatMap { $0 }

// MARK: Date Utilities

extension Date {
    static func daysAgo(_ daysAgo: Int) -> Date {
        Calendar.current.date(byAdding: .day, value: -daysAgo, to: Date())!
    }

    func daysEqual(_ other: Date) -> Bool {
        Calendar.current.dateComponents([.day], from: self, to: other).day == 0
    }
}

extension Date {
    static let wwdc22: Date = DateComponents(
        calendar: .autoupdatingCurrent,
        timeZone: TimeZone(identifier: "PST"),
        year: 2022,
        month: 6,
        day: 6,
        hour: 9,
        minute: 41,
        second: 00).date!
}
```

```swift
Chart(partyTasksRemaining) {
    BarMark(
        x: .value("Date", $0.date, unit: .day),
        y: .value("Tasks Remaining", $0.remainingCount)
    )
}
.padding()
```

Here the framework chose round numbers for y axis and provided a default color for the bar marks.  You can already read the declarative, state-driven syntax.

For this chart, I've chosen a barmark.  But I can switch to a line mark.


```swift
var body: some View {
    Chart(partyTasksRemaining) {
        BarMark(
            x: .value("Date", $0.date, unit: .day),
            y: .value("Tasks Remaining", $0.remainingCount)
        )
    }
    .padding()
}
```

```swift
var body: some View {
    Chart(partyTasksRemaining) {
        LineMark(
            x: .value("Date", $0.date, unit: .day),
            y: .value("Tasks Remaining", $0.remainingCount)
        )
        .foregroundStyle(by: .value("Category", $0.category))
    }
    .padding()
}
```

symbol
```swift
var body: some View {
    Chart(partyTasksRemaining) {
        LineMark(
            x: .value("Date", $0.date, unit: .day),
            y: .value("Tasks Remaining", $0.remainingCount)
        )
        .foregroundStyle(by: .value("Category", $0.category))
        .symbol(by: .value("Category", $0.category))
    }
    .padding()
}
```

swiftui views within a chart.  ForEach.  

Localization, dark mode, dynamic type, etc. etc.  Wroks across all platforms.

```swift
var body: some View {
    Chart {
        ForEach(partyTasksRemaining) { task in
            LineMark(
                x: .value("Date", task.date, unit: .day),
                y: .value("Tasks Remaining", task.remainingCount)
            )
            .foregroundStyle(by: .value("Category", task.category))
            .symbol(by: .value("Category", task.category))
            .annotation(position: .leading) {
                Text("\(task.category.emoji)")
            }
        }

        RuleMark(y: .value("Value", 5))
            .foregroundStyle(.red)
            .lineStyle(StrokeStyle(lineWidth: 2.0, dash: [4, 5]))
            .annotation(position: .top, alignment: .trailing) {
                VStack(alignment: .trailing) {
                    Text("Today's Goal")
                    Text("Status: ✔︎")
                }
                .font(.caption)
                .foregroundColor(.gray)
                .padding(.trailing, 2)
            }
    }
}
```

Add to your watch list
* plotting fundamentals
* marks and composition
* customization
[[Hello Swift Charts]]
[[Swift Charts Raise the bar]]

# navigation and windows
We already support immersive push/pop, expansive detial-=rich split views, and multiwindow experiences.  

This year, we're adding big updates for all 3.

## Stacks
NavigationStack container view.
```swift
import Foundation

// MARK: Food Models

/// A model representing a food with a price and quantity.
struct FoodItem: Hashable, Identifiable, Codable, Equatable {
    let emoji: String
    let name: String
    var description: String = ""
    let price: Decimal
    var quantity: Int = 0
    var id: String { name }
}

let donut = FoodItem(emoji: "🍩", name: "Doughnut", description: "Yeast, Old-fashioned, Cake, and the dubious Apple Fritter", price: 2.35, quantity: 6)
let moonCake = FoodItem(emoji: "🥮", name: "Moon Cake", description: "Lotus seed paste — plenty of crust", price: 2.20, quantity: 4)
let shavedIce = FoodItem(emoji: "🍧", name: "Shaved Ice", description: "Shave your own ice!", price: 3.25, quantity: 1)
let cupcake = FoodItem(emoji: "🧁", name: "Cupcake", description: "Also goes by the name Cake Nano", price: 4.00, quantity: 5)
let flan = FoodItem(emoji: "🍮", name: "Flan", description: "What's in a flan? That which we call milk, eggs, and sugar by any other name would taste just as sweet.", price: 6.50, quantity: 2)
let taffy = FoodItem(emoji: "🍬", name: "Taffy", description: "Freshwater, actually.", price: 1.00, quantity: 11)
let cake = FoodItem(emoji: "🎂", name: "Cake Cake", description: "The real deal", price: 15.00, quantity: 1)
let cookie = FoodItem(emoji: "🍪", name: "Cookie Cake", description: "The ultimate dessert", price: 4.30, quantity: 1)

let relatedFoods = [donut, moonCake, shavedIce, cupcake, flan, taffy, cake, cookie]

extension Array where Element: Equatable {

    /// A quick-and-dirty way of getting a random few elements from an Array that don't include a single,
    /// particular element.
    /// - Parameters:
    ///   - count: The number of desired random elements, must be less than `Array.count`
    ///   - except: Filter out this particular element    
    func random(_ count: Int, except: Element) -> [Element] {
        assert(count >= count)
        var copy = self
        copy.shuffle()
        copy.removeAll(where: { $0 == except })
        return Array(copy[0..<count])
    }
}

let partyFoods = [
    FoodItem(emoji: "🍨", name: "Ice Cream",
             price: 3.50, quantity: 4),
    flan,
    taffy,
    donut,
    FoodItem(emoji: "🍉", name: "Watermelon",
             price: 3.65, quantity: 1),
    FoodItem(emoji: "🍒", name: "Cherries",
             price: 8.00, quantity: 1),
    cupcake,
    cookie,
    FoodItem(emoji: "🍥", name: "Fish Cake",
             price: 5.00, quantity: 2),
    moonCake,
    cake,
    FoodItem(emoji: "🍘", name: "Rice Cracker",
             price: 0.25, quantity: 16),
    FoodItem(emoji: "🥨", name: "Pretzels",
             price: 3.00, quantity: 3),
    shavedIce,
    FoodItem(emoji: "🥧", name: "Apple Pie",
             price: 4.10, quantity: 1)
]
```

```swift
// MARK: NavigationStack with View-based NavigationLinks

struct FoodsListView: View {
    fileprivate var foodItems = partyFoods
    @State private var selectedFoodItems: [FoodItem] = []

    var body: some View {
        NavigationStack {
            List(foodItems) { item in
                NavigationLink {
                    FoodDetailView(item: item)
                } label: {
                    FoodRow(food: item)
                }
            }
            .navigationTitle("Party Food")

        }
    }
}

struct FoodRow: View {
    let food: FoodItem

    var body: some View {
        HStack {
            Text(food.emoji)
                .font(.system(size: 15))
                .foregroundStyle(.secondary)
            Text(food.name)
                .font(.caption)
                .bold()
            Spacer()
            Text("\(food.quantity)")
        }
    }
}

struct FoodDetailView: View {
    let item: FoodItem

    var body: some View {
        ScrollView {
            VStack {
                HStack {
                    Text(item.emoji)
                        .font(.system(size: 30))
                    Text(item.name)
                        .font(.title3)
                }
                .padding(.bottom, 4)
                Text(item.description)
                    .font(.caption)
                Divider()
                RelatedFoodsView(relatedFoods: relatedFoods.random(3, except: item))
            }
        }
    }
}

struct RelatedFoodsView: View {
    @State var relatedFoods: [FoodItem]

    var body: some View {
        VStack {
            Text("Related Foods")
                .background(.background, in: RoundedRectangle(cornerRadius: 2))
            HStack {
                ForEach(relatedFoods) { food in                    
                    NavigationLink {
                        FoodDetailView(item: food)
                    } label: { Text(food.emoji) }
                }
            }
        }
    }
}
```

This approach might be all you need.  But tehre'ss a new way to present views and have programmatic ocntrol over presented state.

Data driven APIs.

```swift
// MARK: NavigationStack with Value-based Navigation Links

struct FoodsListView: View {
    fileprivate var foodItems = partyFoods
    @State private var selectedFoodItems: [FoodItem] = []

    var body: some View {
        NavigationStack(path: $selectedFoodItems) {
            List(foodItems) { item in
                NavigationLink(value: item) {
                    FoodRow(food: item)
                }
            }
            .navigationTitle("Party Food")
            .navigationDestination(for: FoodItem.self) { item in
                FoodDetailView(item: item, path: $selectedFoodItems)
            }
        }
    }
}

struct FoodDetailView: View {
    let item: FoodItem
    @Binding var path: [FoodItem]

    var body: some View {
        ScrollView {
            VStack {
                HStack {
                    Text(item.emoji)
                        .font(.system(size: 30))
                    Text(item.name)
                        .font(.title3)
                }
                .padding(.bottom, 4)
                Text(item.description)
                    .font(.caption)
                Divider()
                RelatedFoodsView(relatedFoods: relatedFoods.random(3, except: item))
                if path.count > 1 {
                    Button("Back to First Item") { path.removeSubrange(1...) }
                }
            }
        }
    }
}

struct RelatedFoodsView: View {
    @State var relatedFoods: [FoodItem]

    var body: some View {
        VStack {
            Text("Related Foods")
                .background(.background, in: RoundedRectangle(cornerRadius: 2))
            HStack {
                ForEach(relatedFoods) { food in
                    NavigationLink(value: food) {
                        Text(food.emoji)
                    }
                }
            }
        }
    }
}
```

Instead of a view, it can take a value that represents a destination.  When tapping ona  link, swiftui will use the value's type to find the right destination and push onto the stack.

Because we now use data to drive our stack, it's possible to represent current navigation pathas excplicit state.

## Split views
```swift
// MARK: NavigationSplitView Demo

struct PartyPlannerHome: View {
    @State private var selectedTask: PartyTask?

    var body: some View {
        NavigationSplitView {
            List(PartyTask.allCases, selection: $selectedTask) { task in
                NavigationLink(value: task) {
                    TaskLabel(task: task)
                }
						    .listItemTint(task.color)
            }
        } detail: {
            selectedTask.flatMap { $0.color } ?? .white
        }
    }
}

struct TaskLabel: View {
    let task: PartyTask

    var body: some View {
        Label {
            VStack(alignment: .leading) {
                Text(task.name)
                Text(task.subtitle)
                    .font(.footnote)
                    .foregroundStyle(.secondary)
            }
        } icon: {
            Image(systemName: task.imageName)
                .symbolVariant(.circle.fill)
        }
    }
}
```

Declare two and three-column layouts.  Automatically collapses when smaller size classes.  Adaptive, multiplatform apps.

Directly compose navigationstack and navigationsplitview.  e.g. detail column can be its own, self-contained navigation stack.

```swift
struct PartyPlannerHome: View {
    @State private var selectedTask: PartyTask?

    var body: some View {
        NavigationSplitView {
            List(PartyTask.allCases, selection: $selectedTask) { task in
                NavigationLink(value: task) {
                    TaskLabel(task: task)
                }
                .listItemTint(task.color)
            }
        } detail: {
            if case .food = selectedTask {
                FoodsListView()
            } else {
                selectedTask.flatMap { $0.color } ?? .white
            }
        }
    }
}
```

add to your watch list
* heterogeneous navigation paths
* advanced deep-linking
* state restoration
* migrating from navigationView

[[The SwiftUI cookbook for navigation]]

## Scene
WindowGroup
```swift
@main
struct PartyPlanner: App {
    var body: some Scene {
        WindowGroup("Party Planner") {
            PartyPlannerHome()
        }

        Window("Party Budget", id: "budget") {
            Text("Budget View")
        }
        .keyboardShortcut("0")
    }
}
```

New this year, we're adding `Window` which declares a single unique window for your app.  Here I've added a separate budget window.

By default, the window is available and can be shown from Window menu.  We can make that even easier by assigning a cmd-0 keyboard shortcut.
```swift
struct DetailView: View {
    @Environment(\.openWindow) var openWindow

    var body: some View {
        Text("Detail View")
            .toolbar {
                Button {
                    openWindow(id: "budget")
                } label: {
                    Image(systemName: "dollarsign")
                }
            }
    }
}
```


toolbar button.  Using environment action `openWindow` I can programatically open.

```swift
@main
struct PartyPlanner: App {
    var body: some Scene {
        WindowGroup("Party Planner") {
            PartyPlannerHome()
        }

        Window("Party Budget", id: "budget") {
            Text("Budget View")
        }
        .keyboardShortcut("0")
        .defaultPosition(.topLeading)
        .defaultSize(width: 220, height: 250)
    }
}
```

New window customizations.  Modifiers, default size, etc.

Little auxiliary windows.  But we're a multiplatform app, and we need a better design for smaller screens.  On iOS we chose to display our budget within a resizable sheet instead.

```swift
struct PartyPlannerHome: View {
    @State private var selectedTask: PartyTask?
    @State private var presented: Bool = false

    var body: some View {
        NavigationSplitView {
            List(PartyTask.allCases, selection: $selectedTask) { task in
                NavigationLink(value: task) {
                    TaskLabel(task: task)
                }
                .listItemTint(task.color)
            }
        } detail: {
            if case .food = selectedTask {
                FoodsListView()
            } else {
                selectedTask.flatMap { $0.color } ?? .white
            }
        }
        .sheet(isPresented: $presented) {
            Text("Budget View")
                .presentationDetents([.height(250), .medium])
                .presentationDragIndicator(.visible)
        }
    }
}
```

sticks to two different sizes: 250 poitns, and medium height.

Simple to iterate between platforms with multiplatform targets in xcode.  One target can be deployed to multiple platforms.  Pick your platfrom from the usual pulldown menu.

[[What's new in Xcode]]
[[Use Xcode to build a multiplatform app]]

Let's look at the menu bar.  With ventura, you can now build menubar extras in swiftui.

```swift
@main
struct PartyPlanner: App {
    var body: some Scene {
        Window("Party Budget", id: "budget") {
            Text("Budget View")
        }

        MenuBarExtra("Bulletin Board", systemImage: "quote.bubble") {
            BulletinBoard()
        }
        .menuBarExtraStyle(.window)
    }
}



private let allPosts: [String] = [
    "Did you know: On your third birthday, you are celebrating your 4.0 release.",
]

struct BulletinBoard: View {

    @State var currentPostIndex: Int = 0

    var currentPost: String {
        allPosts[currentPostIndex]
    }

    var body: some View {
        VStack(spacing: 16) {

            VStack(spacing: 12) {
                HStack(alignment: .firstTextBaseline) {
                    Text("“")
                        .font(.custom("Helvetica", size: 50).bold())
                        .baselineOffset(-23)
                        .foregroundStyle(.tertiary)

                    Text("Party Bulletin Board")
                        .font(.headline.weight(.semibold))
                        .foregroundStyle(.secondary)

                    Spacer()

                    Text("June 6, 2022")
                        .font(.headline.weight(.regular))
                        .foregroundStyle(.secondary)
                }
                .frame(height: 20)


                Text(currentPost)
                    .font(.system(size: 18))
                    .multilineTextAlignment(.center)
            }
            .padding(.bottom, 4)

            Divider()

            HStack {
                Button {

                } label: {
                    Label("Calendar", systemImage: "calendar")
                }
                Button {
                    currentPostIndex = (currentPostIndex + 1) % allPosts.count
                } label: {
                    Text("Previous")
                        .frame(maxWidth: .infinity)
                }

                ShareLink(items: [currentPost])
            }
            .labelStyle(.iconOnly)
            .controlSize(.large)
        }
        .padding(16)
    }
}
```

Can build an entire app using just a menu bar extra.  

```swift
@main
struct MessageBoard: App {
    var body: some Scene {
        MenuBarExtra("Bulletin Board", systemImage: "quote.bubble") {
            BulletinBoard()
        }
        .menuBarExtraStyle(.window)
    }
}
```

add to your watch list
* scene basics
* auxiliary scenes
* scene navigation
* scene customizations
[[Bring multiple windows to your SwiftUI app]]


# Advanced controls
## Forms
We adopted this new settings app design.  in our party app.

```swift
struct ContentView: View {
    enum Theme: String, CaseIterable, Identifiable {
        var id: String { self.rawValue }
        case blue, gold, black, white

        var swatch: some View {
            Circle()
                .fill(color)
                .overlay {
                    Circle().stroke(.tertiary)
                }
                .frame(width: 15, height: 15)
        }

        var color: Color {
            switch self {
            case .blue: return .blue
            case .gold: return .yellow
            case .black: return .black
            case .white: return .white
            }
        }
    }

    enum ColorScheme: String {
        case light, dark
    }

    enum Decoration: String, CaseIterable {
        case balloon, confetti, inflatables, noisemakers, all, none
    }

    private let address = "One Apple Park Way"

    @State private var date: Date = DateComponents(
        calendar: .current, timeZone: .current, year: 2022, month: 6, day: 6
    ).date!
    @State private var eventDescription: String =
        "Come and join us celebrate SwiftUI's birthday party!\n🎉🎂"

    @State private var scheme: ColorScheme = .light
    @State private var accent: Theme = .blue
    @State private var extraGuests = false
    @State private var spacesCount: Float = 2

    @State private var includeBalloons = false
    @State private var includeConfetti = false
    @State private var includeInflatables = false
    @State private var includeBlowers = false

    @State private var selectedDecorations: [Decoration] = []
    @State private var decorationThemes: [Decoration: Theme] = [
        .balloon : .blue,
        .confetti: .gold,
        .inflatables: .black,
        .noisemakers: .white,
        .none: .black
    ]

    private var themes: [Binding<Theme>] {
        if selectedDecorations.count == 0 {
            return [Binding($decorationThemes[.none])!]
        }
        return selectedDecorations.compactMap {
            Binding($decorationThemes[$0])
        }
    }

    var body: some View {
        Form {
            Section {
                LabeledContent("Location", value: address)
                DatePicker("Date", selection: $date)
                TextField("Description", text: $eventDescription, axis: .vertical)
                    .lineLimit(3, reservesSpace: true)
            }

            Section("Vibe") {
                Picker("Accent color", selection: $accent) {
                    ForEach(Theme.allCases) { theme in
                        Text(theme.rawValue.capitalized).tag(theme)
                    }
                }
                Picker("Color scheme", selection: $scheme) {
                    Text("Light").tag(ColorScheme.light)
                    Text("Dark").tag(ColorScheme.dark)
                }
                #if os(macOS)
                .pickerStyle(.inline)
                #endif
                Toggle(isOn: $extraGuests) {
                    Text("Allow extra guests")
                    Text("The more the merrier!")
                }
                if extraGuests {
                    Stepper("Guests limit", value: $spacesCount, format: .number)
                }
            }

            Section("Decorations") {
                Section {
                    List(selection: $selectedDecorations) {
                        DisclosureGroup {
                            HStack {
                                Toggle("Balloons 🎈", isOn: $includeBalloons)
                                Spacer()
                                decorationThemes[.balloon].map { $0.swatch }
                            }
                            .tag(Decoration.balloon)

                            HStack {
                                Toggle("Confetti 🎊", isOn: $includeConfetti)
                                Spacer()
                                decorationThemes[.confetti].map { $0.swatch }
                            }
                            .tag(Decoration.confetti)

                            HStack {
                                Toggle("Inflatables 🪅", isOn: $includeInflatables)
                                Spacer()
                                decorationThemes[.inflatables].map { $0.swatch }
                            }
                            .tag(Decoration.inflatables)

                            HStack {
                                Toggle("Party Horns 🥳", isOn: $includeBlowers)
                                Spacer()
                                decorationThemes[.noisemakers].map { $0.swatch }
                            }
                            .tag(Decoration.noisemakers)
                        } label: {
                            Toggle("All Decorations", isOn: [
                                $includeBalloons, $includeConfetti,
                                $includeInflatables, $includeBlowers
                            ])
                            .tag(Decoration.all)
                        }
                        #if os(macOS)
                        .toggleStyle(.checkbox)
                        #endif
                    }

                    Picker("Decoration theme", selection: themes) {
                        Text("Blue").tag(Theme.blue)
                        Text("Black").tag(Theme.black)
                        Text("Gold").tag(Theme.gold)
                        Text("White").tag(Theme.white)
                    }
                    #if os(macOS)
                    .pickerStyle(.radioGroup)
                    #endif
                }
            }

        }
        .formStyle(.grouped)
    }
}
```

content and controls within the form will automatically adapt to the new style.  ex, sections will basically group with headers.  Controls will align labels. 

Some ocntrols may adapt their visual appearance. ex toggles show as switches.
Lighter-widgth appearance.

SwiftuI makes it easy to align other types of content.  `LabeledContent` view.  New controls or even just display some information.  

But it can wrap any kind of view:

```swift
struct ContentView: View {
    enum Theme: String, CaseIterable, Identifiable {
        var id: String { self.rawValue }
        case blue, gold, black, white

        var swatch: some View {
            Circle()
                .fill(color)
                .overlay {
                    Circle().stroke(.tertiary)
                }
                .frame(width: 15, height: 15)
        }

        var color: Color {
            switch self {
            case .blue: return .blue
            case .gold: return .yellow
            case .black: return .black
            case .white: return .white
            }
        }
    }

    enum ColorScheme: String {
        case light, dark
    }

    enum Decoration: String, CaseIterable {
        case balloon, confetti, inflatables, noisemakers, all, none
    }

    private let location = Location(
        firstLine: "One Apple Park Way", secondLine: "Cupertino, CA 95014")

    @State private var date: Date = DateComponents(
        calendar: .current, timeZone: .current, year: 2022, month: 6, day: 6
    ).date!
    @State private var eventDescription: String =
        "Come and join us celebrate SwiftUI's birthday party!\n🎉🎂"

    @State private var scheme: ColorScheme = .light
    @State private var accent: Theme = .blue
    @State private var extraGuests = false
    @State private var spacesCount: Float = 2

    @State private var includeBalloons = false
    @State private var includeConfetti = false
    @State private var includeInflatables = false
    @State private var includeBlowers = false

    @State private var selectedDecorations: [Decoration] = []
    @State private var decorationThemes: [Decoration: Theme] = [
        .balloon : .blue,
        .confetti: .gold,
        .inflatables: .black,
        .noisemakers: .white,
        .none: .black
    ]

    private var themes: [Binding<Theme>] {
        if selectedDecorations.count == 0 {
            return [Binding($decorationThemes[.none])!]
        }
        return selectedDecorations.compactMap {
            Binding($decorationThemes[$0])
        }
    }

    var body: some View {
        Form {
            Section {
                LabeledContent("Location") {
                    AddressView(location)
                }
                DatePicker("Date", selection: $date)
                TextField("Description", text: $eventDescription, axis: .vertical)
                    .lineLimit(3, reservesSpace: true)
            }

            Section("Vibe") {
                Picker("Accent color", selection: $accent) {
                    ForEach(Theme.allCases) { accent in
                        Text(accent.rawValue.capitalized).tag(accent)
                    }
                }
                Picker("Color scheme", selection: $scheme) {
                    Text("Light").tag(ColorScheme.light)
                    Text("Dark").tag(ColorScheme.dark)
                }
                #if os(macOS)
                .pickerStyle(.inline)
                #endif
                Toggle(isOn: $extraGuests) {
                    Text("Allow extra guests")
                    Text("The more the merrier!")
                }
                if extraGuests {
                    Stepper("Guests limit", value: $spacesCount, format: .number)
                }
            }

            Section("Decorations") {
                Section {
                    List(selection: $selectedDecorations) {
                        DisclosureGroup {
                            HStack {
                                Toggle("Balloons 🎈", isOn: $includeBalloons)
                                Spacer()
                                decorationThemes[.balloon].map { $0.swatch }
                            }
                            .tag(Decoration.balloon)

                            HStack {
                                Toggle("Confetti 🎊", isOn: $includeConfetti)
                                Spacer()
                                decorationThemes[.confetti].map { $0.swatch }
                            }
                            .tag(Decoration.confetti)

                            HStack {
                                Toggle("Inflatables 🪅", isOn: $includeInflatables)
                                Spacer()
                                decorationThemes[.inflatables].map { $0.swatch }
                            }
                            .tag(Decoration.inflatables)

                            HStack {
                                Toggle("Party Horns 🥳", isOn: $includeBlowers)
                                Spacer()
                                decorationThemes[.noisemakers].map { $0.swatch }
                            }
                            .tag(Decoration.noisemakers)
                        } label: {
                            Toggle("All Decorations", isOn: [
                                $includeBalloons, $includeConfetti,
                                $includeInflatables, $includeBlowers
                            ])
                            .tag(Decoration.all)
                        }
                        #if os(macOS)
                        .toggleStyle(.checkbox)
                        #endif
                    }

                    Picker("Decoration theme", selection: themes) {
                        Text("Blue").tag(Theme.blue)
                        Text("Black").tag(Theme.black)
                        Text("Gold").tag(Theme.gold)
                        Text("White").tag(Theme.white)
                    }
                    #if os(macOS)
                    .pickerStyle(.radioGroup)
                    #endif
                }
            }

        }
        .formStyle(.grouped)
    }
}


struct AddressView: View {
    private let location: Location

    init(_ location: Location) {
        self.location = location
    }

    var body: some View {
        VStack {
            Text(location.firstLine)
            Text(location.secondLine)
        }
    }
}

struct Location {
    let firstLine: String
    let secondLine: String
}
```

Hierarchically compose text with titles and subtitles.

Build shared interfaces for every platform.

## Controls
Multiline text fields. `axis:.vertical`

```swift
struct ContentView: View {
    @State private var activityDates: Set<DateComponents> = [
        DateComponents(calendar: .current, year: 2022, month: 6, day: 6),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 9),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 10)
    ]
    @State private var title: String = .init()
    @State private var description: String = """
                Join us, and let's force unwrap SwiftUl's
                birthday presents. Note that although
                this activity is optional, we may have
                guards at the entry.
                """

    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Title", text: $title)
                    TextField("Description", text: $description, axis: .vertical)
                }
                Section("Dates") {
                    MultiDatePicker("Activities Dates", selection: $activityDates)
                }
            }
            .navigationTitle("New Activity")
            .toolbar {
                Button("Save") {}
            }
        }
    }
}
```

`lineLimit` behaviors

```swift
struct ContentView: View {
    @State private var activityDates: Set<DateComponents> = [
        DateComponents(calendar: .current, year: 2022, month: 6, day: 6),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 9),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 10)
    ]
    @State private var title: String = .init()
    @State private var description: String = """
                Join us, and let's force unwrap SwiftUl's
                birthday presents. Note that although
                this activity is optional, we may have
                guards at the entry.
                """

    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Title", text: $title)
                    TextField("Description", text: $description, axis: .vertical)
                  		.lineLimit(5)
                }
                Section("Dates") {
                    MultiDatePicker("Activities Dates", selection: $activityDates)
                }
            }
            .navigationTitle("New Activity")
            .toolbar {
                Button("Save") {}
            }
        }
    }
}
```

```swift
struct ContentView: View {
    @State private var activityDates: Set<DateComponents> = [
        DateComponents(calendar: .current, year: 2022, month: 6, day: 6),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 9),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 10)
    ]
    @State private var title: String = .init()
    @State private var description: String = """
                Join us, and let's force unwrap SwiftUl's
                birthday presents. Note that although
                this activity is optional, we may have
                guards at the entry.
                """

    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Title", text: $title)
                    TextField("Description", text: $description, axis: .vertical)
                  		.lineLimit(5...10)
                }
                Section("Dates") {
                    MultiDatePicker("Activities Dates", selection: $activityDates)
                }
            }
            .navigationTitle("New Activity")
            .toolbar {
                Button("Save") {}
            }
        }
    }
}
```

```swift
struct ContentView: View {
    @State private var activityDates: Set<DateComponents> = [
        DateComponents(calendar: .current, year: 2022, month: 6, day: 6),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 9),
        DateComponents(calendar: .current, year: 2022, month: 6, day: 10)
    ]
    @State private var title: String = .init()
    @State private var description: String = """
                Join us, and let's force unwrap SwiftUl's
                birthday presents. Note that although
                this activity is optional, we may have
                guards at the entry.
                """

    var body: some View {
        NavigationStack {
            Form {
                Section {
                    TextField("Title", text: $title)
                    TextField("Description", text: $description, axis: .vertical)
                }
                Section("Dates") {
                    MultiDatePicker("Activities Dates", selection: $activityDates)
                }
            }
            .navigationTitle("New Activity")
            .toolbar {
                Button("Save") {}
            }
        }
    }
}
```
Mixed-state toggles

```swift
struct ContentView: View {
    enum Theme: String, CaseIterable, Identifiable {
        var id: String { self.rawValue }
        case blue, gold, black, white

        var swatch: some View {
            Circle()
                .fill(color)
                .overlay {
                    Circle().stroke(.tertiary)
                }
                .frame(width: 15, height: 15)
        }

        var color: Color {
            switch self {
            case .blue: return .blue
            case .gold: return .yellow
            case .black: return .black
            case .white: return .white
            }
        }
    }

    enum ColorScheme: String {
        case light, dark
    }

    enum Decoration: String, CaseIterable {
        case balloon, confetti, inflatables, noisemakers, all, none
    }

    private let location = Location(
        firstLine: "One Apple Park Way", secondLine: "Cupertino, CA 95014")

    @State private var date: Date = DateComponents(
        calendar: .current, timeZone: .current, year: 2022, month: 6, day: 6
    ).date!
    @State private var eventDescription: String =
        "Come and join us celebrate SwiftUI's birthday party!\n🎉🎂"

    @State private var scheme: ColorScheme = .light
    @State private var accent: Theme = .blue
    @State private var extraGuests = false
    @State private var spacesCount: Float = 2

    @State private var includeBalloons = false
    @State private var includeConfetti = false
    @State private var includeInflatables = false
    @State private var includeBlowers = false

    @State private var selectedDecorations: [Decoration] = []
    @State private var decorationThemes: [Decoration: Theme] = [
        .balloon : .blue,
        .confetti: .gold,
        .inflatables: .black,
        .noisemakers: .white,
        .none: .black
    ]

    private var themes: [Binding<Theme>] {
        if selectedDecorations.count == 0 {
            return [Binding($decorationThemes[.none])!]
        }
        return selectedDecorations.compactMap {
            Binding($decorationThemes[$0])
        }
    }

    var body: some View {
        Form {
            Section {
                LabeledContent("Location") {
                    AddressView(location)
                }
                DatePicker("Date", selection: $date)
                TextField("Description", text: $eventDescription, axis: .vertical)
                    .lineLimit(3, reservesSpace: true)
            }

            Section("Vibe") {
                Picker("Accent color", selection: $accent) {
                    ForEach(Theme.allCases) { accent in
                        Text(accent.rawValue.capitalized).tag(accent)
                    }
                }
                Picker("Color scheme", selection: $scheme) {
                    Text("Light").tag(ColorScheme.light)
                    Text("Dark").tag(ColorScheme.dark)
                }
                #if os(macOS)
                .pickerStyle(.inline)
                #endif
                Toggle(isOn: $extraGuests) {
                    Text("Allow extra guests")
                    Text("The more the merrier!")
                }
                if extraGuests {
                    Stepper("Guests limit", value: $spacesCount, format: .number)
                }
            }

            Section("Decorations") {
                Section {
                    List(selection: $selectedDecorations) {
                        DisclosureGroup {
                            HStack {
                                Toggle("Balloons 🎈", isOn: $includeBalloons)
                                Spacer()
                                decorationThemes[.balloon].map { $0.swatch }
                            }
                            .tag(Decoration.balloon)

                            HStack {
                                Toggle("Confetti 🎊", isOn: $includeConfetti)
                                Spacer()
                                decorationThemes[.confetti].map { $0.swatch }
                            }
                            .tag(Decoration.confetti)

                            HStack {
                                Toggle("Inflatables 🪅", isOn: $includeInflatables)
                                Spacer()
                                decorationThemes[.inflatables].map { $0.swatch }
                            }
                            .tag(Decoration.inflatables)

                            HStack {
                                Toggle("Party Horns 🥳", isOn: $includeBlowers)
                                Spacer()
                                decorationThemes[.noisemakers].map { $0.swatch }
                            }
                            .tag(Decoration.noisemakers)
                        } label: {
                            Toggle("All Decorations", isOn: [
                                $includeBalloons, $includeConfetti,
                                $includeInflatables, $includeBlowers
                            ])
                            .tag(Decoration.all)
                        }
                        #if os(macOS)
                        .toggleStyle(.checkbox)
                        #endif
                    }

                    Picker("Decoration theme", selection: themes) {
                        Text("Blue").tag(Theme.blue)
                        Text("Black").tag(Theme.black)
                        Text("Gold").tag(Theme.gold)
                        Text("White").tag(Theme.white)
                    }
                    #if os(macOS)
                    .pickerStyle(.radioGroup)
                    #endif
                }
            }

        }
        .formStyle(.grouped)
    }
}


struct AddressView: View {
    private let location: Location

    init(_ location: Location) {
        self.location = location
    }

    var body: some View {
        VStack {
            Text(location.firstLine)
            Text(location.secondLine)
        }
    }
}

struct Location {
    let firstLine: String
    let secondLine: String
}
```

```swift
struct ContentView: View {
    enum Theme: String, CaseIterable, Identifiable {
        var id: String { self.rawValue }
        case blue, gold, black, white

        var swatch: some View {
            Circle()
                .fill(color)
                .overlay {
                    Circle().stroke(.tertiary)
                }
                .frame(width: 15, height: 15)
        }

        var color: Color {
            switch self {
            case .blue: return .blue
            case .gold: return .yellow
            case .black: return .black
            case .white: return .white
            }
        }
    }

    enum ColorScheme: String {
        case light, dark
    }

    enum Decoration: String, CaseIterable {
        case balloon, confetti, inflatables, noisemakers, all, none
    }

    private let location = Location(
        firstLine: "One Apple Park Way", secondLine: "Cupertino, CA 95014")

    @State private var date: Date = DateComponents(
        calendar: .current, timeZone: .current, year: 2022, month: 6, day: 6
    ).date!
    @State private var eventDescription: String =
        "Come and join us celebrate SwiftUI's birthday party!\n🎉🎂"

    @State private var scheme: ColorScheme = .light
    @State private var accent: Theme = .blue
    @State private var extraGuests = false
    @State private var spacesCount: Float = 2

    @State private var includeBalloons = false
    @State private var includeConfetti = false
    @State private var includeInflatables = false
    @State private var includeBlowers = false

    @State private var swiftastic = false
    @State private var wwdcParty = true
    @State private var offTheCharts = true
    @State private var oneMoreThing = false

    @State private var selectedDecorations: [Decoration] = []
    @State private var decorationThemes: [Decoration: Theme] = [
        .balloon : .blue,
        .confetti: .gold,
        .inflatables: .black,
        .noisemakers: .white,
        .none: .black
    ]

    private var themes: [Binding<Theme>] {
        if selectedDecorations.count == 0 {
            return [Binding($decorationThemes[.none])!]
        }
        return selectedDecorations.compactMap {
            Binding($decorationThemes[$0])
        }
    }

    var body: some View {
        Form {
            Section {
                LabeledContent("Location") {
                    AddressView(location)
                }
                DatePicker("Date", selection: $date)
                TextField("Description", text: $eventDescription, axis: .vertical)
                    .lineLimit(3, reservesSpace: true)
            }

            Section("Vibe") {
                Picker("Accent color", selection: $accent) {
                    ForEach(Theme.allCases) { accent in
                        Text(accent.rawValue.capitalized).tag(accent)
                    }
                }
                Picker("Color scheme", selection: $scheme) {
                    Text("Light").tag(ColorScheme.light)
                    Text("Dark").tag(ColorScheme.dark)
                }
                #if os(macOS)
                .pickerStyle(.inline)
                #endif
                Toggle(isOn: $extraGuests) {
                    Text("Allow extra guests")
                    Text("The more the merrier!")
                }
                if extraGuests {
                    Stepper("Guests limit", value: $spacesCount, format: .number)
                }
            }

            Section("Decorations") {
                Section {
                    List {
                        DisclosureGroup {
                            HStack {
                                Toggle("Balloons 🎈", isOn: $includeBalloons)
                                Spacer()
                                decorationThemes[.balloon].map { $0.swatch }
                            }
                            .tag(Decoration.balloon)

                            HStack {
                                Toggle("Confetti 🎊", isOn: $includeConfetti)
                                Spacer()
                                decorationThemes[.confetti].map { $0.swatch }
                            }
                            .tag(Decoration.confetti)

                            HStack {
                                Toggle("Inflatables 🪅", isOn: $includeInflatables)
                                Spacer()
                                decorationThemes[.inflatables].map { $0.swatch }
                            }
                            .tag(Decoration.inflatables)

                            HStack {
                                Toggle("Party Horns 🥳", isOn: $includeBlowers)
                                Spacer()
                                decorationThemes[.noisemakers].map { $0.swatch }
                            }
                            .tag(Decoration.noisemakers)
                        } label: {
                            Toggle("All Decorations", isOn: [
                                $includeBalloons, $includeConfetti,
                                $includeInflatables, $includeBlowers
                            ])
                            .tag(Decoration.all)
                        }
                        #if os(macOS)
                        .toggleStyle(.checkbox)
                        #endif
                    }

                    Picker("Decoration theme", selection: themes) {
                        Text("Blue").tag(Theme.blue)
                        Text("Black").tag(Theme.black)
                        Text("Gold").tag(Theme.gold)
                        Text("White").tag(Theme.white)
                    }
                    #if os(macOS)
                    .pickerStyle(.radioGroup)
                    #endif
                }
            }

            Section("Hashtags") {
                VStack(alignment: .leading) {
                    HStack {
                        Toggle("#Swiftastic", isOn: $swiftastic)
                        Toggle("#WWParty", isOn: $wwdcParty)
                    }
                    HStack {
                        Toggle("#OffTheCharts", isOn: $offTheCharts)
                        Toggle("#OneMoreThing", isOn: $oneMoreThing)
                    }
                }
                .toggleStyle(.button)
                .buttonStyle(.bordered)
            }

        }
        .formStyle(.grouped)
    }
}

struct AddressView: View {
    private let location: Location

    init(_ location: Location) {
        self.location = location
    }

    var body: some View {
        VStack {
            Text(location.firstLine)
            Text(location.secondLine)
        }
    }
}

struct Location {
    let firstLine: String
    let secondLine: String
}
```

picker changes to match the current decoration.  Can show mixed state indicator.

formatted steppers.  And now available on watchOS.

Accessibility quick actions.


```swift
struct ContentView: View {
    @State private var isInCart: Bool = false

    var body: some View {
        VStack(alignment: .leading) {
            ItemDescriptionView()
            addToCartButton
        }
        .accessibilityQuickAction(style: .prompt) {
            addToCartButton
        }
    }

    var addToCartButton: some View {
        Button(isInCart ? "Remove from cart" : "Add to cart") {
            isInCart.toggle()
        }
    }
}

struct ItemDescriptionView: View {
    var body: some View {
        ScrollView {
            VStack {
                HStack {
                    Text("🎈")
                        .font(.title2)
                    Text("Balloons")
                        .font(.title3)
                    Spacer()
                }
                .padding(.bottom, 4)
                Text(
                    """
                    This is perhaps our funniest product! It is made up of a
                    rubber fabric and comes in various unique colors.
                    """)
                .font(.caption)
            }
        }
    }
}
```

by clenching your hand.  Any other action.  Share the same code.

## tables
Now supported on iPadOS.  As you'd expect, tables are defined using same table API we introduced last year for macOS.  Making it easy to share code between platforms.

Three columns.  name, city, and invitation status.
```swift
struct ContentView: View {
    @StateObject private var attendeeStore = AttendeeStore()
    var body: some View {
        NavigationStack {
            Table(attendeeStore.attendees) {
                TableColumn("Name") { attendee in
                    AttendeeRow(attendee)
                }
                TableColumn("City", value: \.city)
                TableColumn("Status") { attendee in
                    StatusRow(attendee)
                }
            }
            .navigationTitle("Invitations")
            .toolbar(id: "toolbar") {
                ToolbarItem(id: "new", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("New Invitation", systemImage: "envelope")
                    }
                }
                ToolbarItem(id: "edit", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Edit", systemImage: "pencil.circle")
                    }
                }
                ToolbarItem(id: "share", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Share", systemImage: "square.and.arrow.up")
                    }
                }
                ToolbarItem(id: "tag", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Tags", systemImage: "tag")
                    }
                }
                ToolbarItem(
                    id: "reminder", placement: .secondaryAction, showsByDefault: false
                ) {
                    Button(action: {}) {
                        Label("Set reminder", systemImage: "bell")
                    }
                }
            }
            .toolbarRole(.editor)
        }
    }
}

class AttendeeStore: ObservableObject {
    @Published var attendees: [Attendee] = [/* Default attendees */]
}


struct Attendee: Identifiable, Hashable {
    enum Status: String {
        case accepted, declined, maybe

        func displayText() -> Text {
            switch self {
            case .accepted: return Text(
                "Accepted \(Image(systemName: "person.crop.circle.badge.checkmark"))")
            case .maybe: return Text(
                "Maybe \(Image(systemName: "person.crop.circle.badge.questionmark"))")
            case .declined: return Text(
                "Declined \(Image(systemName: "person.crop.circle.badge.minus"))")
            }
        }
    }
    
    let id = UUID()
    let memojiName: String
    let name: String
    let city: String
    let status: Status

    init(memojiName: String, name: String, cities: String, status: Status) {
        self.memojiName = memojiName
        self.name = name
        self.city = cities
        self.status = status
    }
}

struct AttendeeRow: View {
    let attendee: Attendee

    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        HStack {
            Image(attendee.memojiName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                #if os(macOS)
                .frame(width: 20, height: 20)
                .overlay {
                    Circle()
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #else
                .frame(width: 32, height: 32)
                .overlay {
                    RoundedRectangle(cornerRadius: 6)
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #endif
            Text(attendee.name)
        }
    }
}

struct StatusRow: View {
    let attendee: Attendee
    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        attendee.status.displayText()
            .symbolVariant(.fill)
            .symbolRenderingMode(.multicolor)
    }
}
```

This table also renders appropriately in compact size.  Just the primary column.  Let's switch contexts and check otu the table on macOS.

Add context menus for performing common actions.
```swift
struct ContentView: View {
    @StateObject private var attendeeStore = AttendeeStore()
    @State private var selection = Set<Attendee.ID>()

    var body: some View {
        NavigationStack {
            Table(attendeeStore.attendees, selection: $selection) {
                TableColumn("Name") { attendee in
                    AttendeeRow(attendee)
                }
                TableColumn("City", value: \.city)
                TableColumn("Status") { attendee in
                    StatusRow(attendee)
                }
            }
            .navigationTitle("Invitations")
            #if os(macOS)
            .contextMenu(forSelectionType: Attendee.ID.self) { selection in
                if selection.isEmpty {
                    Button("New Invitation") { addInvitation() }
                } else if selection.count == 1 {
                    Button("Mark as VIP") { markVIPs(selection) }
                } else {
                    Button("Mark as VIPs") { markVIPs(selection) }
                }
            }
            #endif
            .toolbar(id: "toolbar") {
                ToolbarItem(id: "new", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("New Invitation", systemImage: "envelope")
                    }
                }
                ToolbarItem(id: "edit", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Edit", systemImage: "pencil.circle")
                    }
                }
                ToolbarItem(id: "share", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Share", systemImage: "square.and.arrow.up")
                    }
                }
                ToolbarItem(id: "tag", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Tags", systemImage: "tag")
                    }
                }
                ToolbarItem(
                    id: "reminder", placement: .secondaryAction, showsByDefault: false
                ) {
                    Button(action: {}) {
                        Label("Set reminder", systemImage: "bell")
                    }
                }
            }
            .toolbarRole(.editor)
        }
    }

    private func addInvitation() {}

    private func markVIPs(_ items: Set<String>) {}
}


class AttendeeStore: ObservableObject {
    @Published var attendees: [Attendee] = [/* Default attendees */]
}


struct Attendee: Identifiable, Hashable {
    enum Status: String {
        case accepted, declined, maybe

        func displayText() -> Text {
            switch self {
            case .accepted: return Text(
                "Accepted \(Image(systemName: "person.crop.circle.badge.checkmark"))")
            case .maybe: return Text(
                "Maybe \(Image(systemName: "person.crop.circle.badge.questionmark"))")
            case .declined: return Text(
                "Declined \(Image(systemName: "person.crop.circle.badge.minus"))")
            }
        }
    }

    let id = UUID()
    let memojiName: String
    let name: String
    let city: String
    let status: Status

    init(memojiName: String, name: String, cities: String, status: Status) {
        self.memojiName = memojiName
        self.name = name
        self.city = cities
        self.status = status
    }
}

struct AttendeeRow: View {
    let attendee: Attendee

    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        HStack {
            Image(attendee.memojiName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                #if os(macOS)
                .frame(width: 20, height: 20)
                .overlay {
                    Circle()
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #else
                .frame(width: 32, height: 32)
                .overlay {
                    RoundedRectangle(cornerRadius: 6)
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #endif
            Text(attendee.name)
        }
    }
}

struct StatusRow: View {
    let attendee: Attendee
    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        attendee.status.displayText()
            .symbolVariant(.fill)
            .symbolRenderingMode(.multicolor)
    }
}
```

Revel actions directly within the table, which is great for speed/efficiency.  But i'll also make them more discoverable.  Improve discoverability by displaying common actions in the toolbar.

```swift
struct ContentView: View {
    @StateObject private var attendeeStore = AttendeeStore()
    @State private var selection = Set<Attendee.ID>()

    var body: some View {
        NavigationStack {
            Table(attendeeStore.attendees, selection: $selection) {
                TableColumn("Name") { attendee in
                    AttendeeRow(attendee)
                }
                TableColumn("City", value: \.city)
                TableColumn("Status") { attendee in
                    StatusRow(attendee)
                }
            }
            .navigationTitle("Invitations")
            #if os(macOS)
            .contextMenu(forSelectionType: Attendee.ID.self) { selection in
                if selection.isEmpty {
                    Button("New Invitation") { addInvitation() }
                } else if selection.count == 1 {
                    Button("Mark as VIP") { markVIPs(selection) }
                } else {
                    Button("Mark as VIPs") { markVIPs(selection) }
                }
            }
            #endif
            .toolbar(id: "toolbar") {
                ToolbarItem(id: "new", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("New Invitation", systemImage: "envelope")
                    }
                }
                ToolbarItem(id: "edit", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Edit", systemImage: "pencil.circle")
                    }
                }
                ToolbarItem(id: "share", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Share", systemImage: "square.and.arrow.up")
                    }
                }
                ToolbarItem(id: "tag", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Tags", systemImage: "tag")
                    }
                }
                ToolbarItem(
                    id: "reminder", placement: .secondaryAction, showsByDefault: false
                ) {
                    Button(action: {}) {
                        Label("Set reminder", systemImage: "bell")
                    }
                }
            }
            .toolbarRole(.editor)
        }
    }

    private func addInvitation() {}

    private func markVIPs(_ items: Set<String>) {}
}


class AttendeeStore: ObservableObject {
    @Published var attendees: [Attendee] = [/* Default attendees */]
}


struct Attendee: Identifiable, Hashable {
    enum Status: String {
        case accepted, declined, maybe

        func displayText() -> Text {
            switch self {
            case .accepted: return Text(
                "Accepted \(Image(systemName: "person.crop.circle.badge.checkmark"))")
            case .maybe: return Text(
                "Maybe \(Image(systemName: "person.crop.circle.badge.questionmark"))")
            case .declined: return Text(
                "Declined \(Image(systemName: "person.crop.circle.badge.minus"))")
            }
        }
    }

    let id = UUID()
    let memojiName: String
    let name: String
    let city: String
    let status: Status

    init(memojiName: String, name: String, cities: String, status: Status) {
        self.memojiName = memojiName
        self.name = name
        self.city = cities
        self.status = status
    }
}

struct AttendeeRow: View {
    let attendee: Attendee

    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        HStack {
            Image(attendee.memojiName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                #if os(macOS)
                .frame(width: 20, height: 20)
                .overlay {
                    Circle()
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #else
                .frame(width: 32, height: 32)
                .overlay {
                    RoundedRectangle(cornerRadius: 6)
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #endif
            Text(attendee.name)
        }
    }
}

struct StatusRow: View {
    let attendee: Attendee
    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        attendee.status.displayText()
            .symbolVariant(.fill)
            .symbolRenderingMode(.multicolor)
    }
}
```

Note that on iPadOS, not every toolbar requires cusotomization.

Search.

```swift
struct ContentView: View {
    public struct AttendeeToken: Identifiable, Equatable, Hashable {
        enum Guts {
            case name
            case location
            case status
        }

        let guts: Guts
        var query: String = .init()

        var id: String {
            self.systemImage
        }

        static let allCases: [AttendeeToken] = [.name, .location, .status]

        mutating func displayName(_ query: String) -> String {
            self.query = query
            switch guts {
            case .name: return "Name contains: \(query)"
            case .location: return "City contains: \(query)"
            case .status: return "Status contains: \(query)"
            }
        }

        var systemImage: String {
            switch guts {
            case .name: return "person"
            case .location: return "location.square"
            case .status: return "person.crop.circle.badge"
            }
        }

        static let name: AttendeeToken = .init(guts: .name)
        static let location: AttendeeToken = .init(guts: .location)
        static let status: AttendeeToken = .init(guts: .status)
    }

    @StateObject private var attendeeStore = AttendeeStore()
    @State private var selection = Set<Attendee.ID>()

    @State private var tokens: [AttendeeToken] = .init()
    @State private var query: String = .init()

    var body: some View {
        NavigationStack {
            Table(attendeeStore.attendees, selection: $selection) {
                TableColumn("Name") { attendee in
                    AttendeeRow(attendee)
                }
                TableColumn("City", value: \.city)
                TableColumn("Status") { attendee in
                    StatusRow(attendee)
                }
            }
            .navigationTitle("Invitations")
            #if os(macOS)
            .contextMenu(forSelectionType: Attendee.ID.self) { selection in
                if selection.isEmpty {
                    Button("New Invitation") { addInvitation() }
                } else if selection.count == 1 {
                    Button("Mark as VIP") { markVIPs(selection) }
                } else {
                    Button("Mark as VIPs") { markVIPs(selection) }
                }
            }
            #endif
            .searchable(text: $query, tokens: $tokens) { token in
                Label(token.query, systemImage: token.systemImage)
            } suggestions: {
                suggestions
            }
            .toolbar(id: "toolbar") {
                ToolbarItem(id: "new", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("New Invitation", systemImage: "envelope")
                    }
                }
                ToolbarItem(id: "edit", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Edit", systemImage: "pencil.circle")
                    }
                }
                ToolbarItem(id: "share", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Share", systemImage: "square.and.arrow.up")
                    }
                }
                ToolbarItem(id: "tag", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Tags", systemImage: "tag")
                    }
                }
                ToolbarItem(
                    id: "reminder", placement: .secondaryAction, showsByDefault: false
                ) {
                    Button(action: {}) {
                        Label("Set reminder", systemImage: "bell")
                    }
                }
            }
            .toolbarRole(.editor)
        }
    }

    @ViewBuilder
    private var suggestions: some View {
        ForEach(attendeeStore.attendees) {
            Text($0.name)
                .foregroundColor(.black)
        }

        if !query.isEmpty {
            ForEach(AttendeeToken.allCases) { token in
                var _token = token
                Label(_token.displayName(query), systemImage: _token.systemImage)
                    .searchCompletion(_token)
            }
        }
    }

    private func addInvitation() {}

    private func markVIPs(_ items: Set<String>) {}
}

class AttendeeStore: ObservableObject {
    @Published var attendees: [Attendee] = [/* Default attendees */]
}

struct Attendee: Identifiable, Hashable {
    enum Status: String {
        case accepted, declined, maybe

        func displayText() -> Text {
            switch self {
            case .accepted: return Text(
                "Accepted \(Image(systemName: "person.crop.circle.badge.checkmark"))")
            case .maybe: return Text(
                "Maybe \(Image(systemName: "person.crop.circle.badge.questionmark"))")
            case .declined: return Text(
                "Declined \(Image(systemName: "person.crop.circle.badge.minus"))")
            }
        }
    }

    let id = UUID()
    let memojiName: String
    let name: String
    let city: String
    let status: Status

    init(memojiName: String, name: String, cities: String, status: Status) {
        self.memojiName = memojiName
        self.name = name
        self.city = cities
        self.status = status
    }
}

struct AttendeeRow: View {
    let attendee: Attendee

    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        HStack {
            Image(attendee.memojiName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                #if os(macOS)
                .frame(width: 20, height: 20)
                .overlay {
                    Circle()
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #else
                .frame(width: 32, height: 32)
                .overlay {
                    RoundedRectangle(cornerRadius: 6)
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #endif
            Text(attendee.name)
        }
    }
}

struct StatusRow: View {
    let attendee: Attendee
    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        attendee.status.displayText()
            .symbolVariant(.fill)
            .symbolRenderingMode(.multicolor)
    }
}
```

Also support search scopes, which appear below toolbar on macos, or as a segmented control beneath search on ios

```swift
struct ContentView: View {
    enum AttendanceScope {
        case inPerson
        case online
    }

    public struct AttendeeToken: Identifiable, Equatable, Hashable {
        enum Guts {
            case name
            case location
            case status
        }

        let guts: Guts
        var query: String = .init()

        var id: String {
            self.systemImage
        }

        static let allCases: [AttendeeToken] = [.name, .location, .status]

        mutating func displayName(_ query: String) -> String {
            self.query = query
            switch guts {
            case .name: return "Name contains: \(query)"
            case .location: return "City contains: \(query)"
            case .status: return "Status contains: \(query)"
            }
        }

        var systemImage: String {
            switch guts {
            case .name: return "person"
            case .location: return "location.square"
            case .status: return "person.crop.circle.badge"
            }
        }

        static let name: AttendeeToken = .init(guts: .name)
        static let location: AttendeeToken = .init(guts: .location)
        static let status: AttendeeToken = .init(guts: .status)
    }

    @StateObject private var attendeeStore = AttendeeStore()
    @State private var selection = Set<Attendee.ID>()

    @State private var tokens: [AttendeeToken] = .init()
    @State private var query: String = .init()
    @State private var scope: AttendanceScope = .inPerson

    var body: some View {
        NavigationStack {
            Table(attendeeStore.attendees, selection: $selection) {
                TableColumn("Name") { attendee in
                    AttendeeRow(attendee)
                }
                TableColumn("City", value: \.city)
                TableColumn("Status") { attendee in
                    StatusRow(attendee)
                }
            }
            .navigationTitle("Invitations")
            #if os(macOS)
            .contextMenu(forSelectionType: Attendee.ID.self) { selection in
                if selection.isEmpty {
                    Button("New Invitation") { addInvitation() }
                } else if selection.count == 1 {
                    Button("Mark as VIP") { markVIPs(selection) }
                } else {
                    Button("Mark as VIPs") { markVIPs(selection) }
                }
            }
            #endif
            .searchable(
                text: $query, tokens: $tokens, scope: $scope
            ) { token in
                Label(
                    token.query,
                    systemImage: token.systemImage)
            } scopes: {
                Text("In Person").tag(AttendanceScope.inPerson)
                Text("Online").tag(AttendanceScope.online)
            } suggestions: {
                suggestions
            }
            .toolbar(id: "toolbar") {
                ToolbarItem(id: "new", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("New Invitation", systemImage: "envelope")
                    }
                }
                ToolbarItem(id: "edit", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Edit", systemImage: "pencil.circle")
                    }
                }
                ToolbarItem(id: "share", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Share", systemImage: "square.and.arrow.up")
                    }
                }
                ToolbarItem(id: "tag", placement: .secondaryAction) {
                    Button(action: {}) {
                        Label("Tags", systemImage: "tag")
                    }
                }
                ToolbarItem(
                    id: "reminder", placement: .secondaryAction, showsByDefault: false
                ) {
                    Button(action: {}) {
                        Label("Set reminder", systemImage: "bell")
                    }
                }
            }
            .toolbarRole(.editor)
        }
    }

    @ViewBuilder
    private var suggestions: some View {
        ForEach(attendeeStore.attendees) {
            Text($0.name)
                .foregroundColor(.black)
        }

        if !query.isEmpty {
            ForEach(AttendeeToken.allCases) { token in
                var _token = token
                Label(_token.displayName(query), systemImage: _token.systemImage)
                    .searchCompletion(_token)
            }
        }
    }

    private func addInvitation() {}

    private func markVIPs(_ items: Set<String>) {}
}

class AttendeeStore: ObservableObject {
    @Published var attendees: [Attendee] = [/* Default attendees */]
}


struct Attendee: Identifiable, Hashable {
    enum Status: String {
        case accepted, declined, maybe

        func displayText() -> Text {
            switch self {
            case .accepted: return Text(
                "Accepted \(Image(systemName: "person.crop.circle.badge.checkmark"))")
            case .maybe: return Text(
                "Maybe \(Image(systemName: "person.crop.circle.badge.questionmark"))")
            case .declined: return Text(
                "Declined \(Image(systemName: "person.crop.circle.badge.minus"))")
            }
        }
    }

    let id = UUID()
    let memojiName: String
    let name: String
    let city: String
    let status: Status

    init(memojiName: String, name: String, cities: String, status: Status) {
        self.memojiName = memojiName
        self.name = name
        self.city = cities
        self.status = status
    }
}

struct AttendeeRow: View {
    let attendee: Attendee

    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        HStack {
            Image(attendee.memojiName)
                .resizable()
                .aspectRatio(contentMode: .fill)
                #if os(macOS)
                .frame(width: 20, height: 20)
                .overlay {
                    Circle()
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #else
                .frame(width: 32, height: 32)
                .overlay {
                    RoundedRectangle(cornerRadius: 6)
                        .stroke(Color.gray.opacity(0.2), lineWidth: 1)
                }
                #endif
            Text(attendee.name)
        }
    }
}

struct StatusRow: View {
    let attendee: Attendee
    init(_ attendee: Attendee) {
        self.attendee = attendee
    }

    var body: some View {
        attendee.status.displayText()
            .symbolVariant(.fill)
            .symbolRenderingMode(.multicolor)
    }
}
```

Add to your watch list

[[SwiftUI on iPad Organize your interface]]
[[SwiftUI on iPad Add toolbars, titles, and more]]
[[What's new in iPad app design]]
[[SwiftUI on the Mac Build the fundamentals]]

# Sharing
Makes your app more integrated into workflows of people who use them

## Photos
```swift
import PhotosUI
import CoreTransferable

struct ContentView: View {
    @ObservedObject var viewModel: FilterModel = .shared
    
    var body: some View {
        NavigationStack {
            Gallery()
                .navigationTitle("Birthday Filter")
                .toolbar {
                    PhotosPicker(
                        selection: $viewModel.imageSelection,
                        matching: .images
                    ) {
                        Label("Pick a photo", systemImage: "plus.app")
                    }
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                }
        }
    }
}

struct Gallery: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        VStack {
            switch viewModel.imageState {
            case .success(let image):
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .draggable(image)
            case .loading:
                ProgressView()
            case .empty:
                Text("No Photo \(Image(systemName: "photo"))")
                    .font(.title2)
                    .fontWeight(.semibold)
                Text("Drag and drop a photo or press\n \(Image(systemName: "plus.app")) to choose a photo manually.")
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            case .failure:
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.system(size: 40))
                    .foregroundColor(.white)
            }
        }
        .padding()
    }
}

@MainActor
class FilterModel: ObservableObject {
    static let shared = FilterModel()

    enum ImageState {
        case empty, loading(Progress), success(Image), failure(Error)
    }

    @Published private(set) var processedImage: Image?
    @Published var imageState: ImageState = .empty
    @Published var imageSelection: PhotosPickerItem? = nil {
        didSet {
            if let imageSelection = imageSelection {
                let progress = loadTransferable(from: imageSelection)
                imageState = .loading(progress)
            } else {
                imageState = .empty
            }
        }
    }

    func applyFilter() { /* Apply your filter */ }

    private func loadTransferable(from imageSelection: PhotosPickerItem) -> Progress {
        return imageSelection.loadTransferable(type: Image.self) { result in
            DispatchQueue.main.async {
                guard imageSelection == self.imageSelection else { return }
                switch result {
                case .success(let image?):
                    self.imageState = .success(image)
                case .success(nil):
                    self.imageState = .empty
                case .failure(let error):
                    self.imageState = .failure(error)
                }
            }
        }
    }
}
```

Filtering the type of content, and more.  

## Sharing
Each paltform has a standard interface for allowing peopel to share content from your apps.  With watchOS 9, you can present sthe share sheet.

```swift
import PhotosUI
import CoreTransferable

struct ContentView: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        NavigationStack {
            Gallery()
                .navigationTitle("Birthday Filter")
                .toolbar {
                    PhotosPicker(
                        selection: $viewModel.imageSelection,
                        matching: .images
                    ) {
                        Label("Pick a photo", systemImage: "plus.app")
                    }
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                    if let item = viewModel.processedImage {
                        ShareLink(
                            item: item, preview: SharePreview("Birthday Effects"))
                    }
                }
        }
    }
}

struct Gallery: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        VStack {
            switch viewModel.imageState {
            case .success(let image):
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .draggable(image)
            case .loading:
                ProgressView()
            case .empty:
                Text("No Photo \(Image(systemName: "photo"))")
                    .font(.title2)
                    .fontWeight(.semibold)
                Text("Drag and drop a photo or press\n \(Image(systemName: "plus.app")) to choose a photo manually.")
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            case .failure:
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.system(size: 40))
                    .foregroundColor(.white)
            }
        }
        .padding()
    }
}

@MainActor
class FilterModel: ObservableObject {
    static let shared = FilterModel()

    enum ImageState {
        case empty, loading(Progress), success(Image), failure(Error)
    }

    @Published private(set) var processedImage: Image?
    @Published var imageState: ImageState = .empty
    @Published var imageSelection: PhotosPickerItem? = nil {
        didSet {
            if let imageSelection = imageSelection {
                let progress = loadTransferable(from: imageSelection)
                imageState = .loading(progress)
            } else {
                imageState = .empty
            }
        }
    }

    func applyFilter() { /* Apply your filter */}

    private func loadTransferable(from imageSelection: PhotosPickerItem) -> Progress {
        return imageSelection.loadTransferable(type: Image.self) { result in
            DispatchQueue.main.async {
                guard imageSelection == self.imageSelection else { return }
                switch result {
                case .success(let image?):
                    self.imageState = .success(image)
                case .success(nil):
                    self.imageState = .empty
                case .failure(let error):
                    self.imageState = .failure(error)
                }
            }
        }
    }
}
```
Share links adapt to the context, such as context menus.

```swift
import PhotosUI
import CoreTransferable

struct ContentView: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        NavigationStack {
            Gallery()
                .navigationTitle("Birthday Filter")
                .toolbar {
                    PhotosPicker(
                        selection: $viewModel.imageSelection,
                        matching: .images
                    ) {
                        Label("Pick a photo", systemImage: "plus.app")
                    }
                    if let item = viewModel.processedImage {
                        ShareLink(
                            item: item, preview: SharePreview("Birthday Effects"))
                    }
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                }
                .contextMenu {
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                    if let item = viewModel.processedImage {
                        ShareLink(
                            item: item, preview: SharePreview("Birthday Effects"))
                    }
                    Button(role: .destructive) {
                        viewModel.deleteCurrentPhoto()
                    } label: {
                        Label("Delete", systemImage: "trash")
                    }
                }
        }
    }
}

struct Gallery: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        VStack {
            switch viewModel.imageState {
            case .success(let image):
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .draggable(image)
            case .loading:
                ProgressView()
            case .empty:
                Text("No Photo \(Image(systemName: "photo"))")
                    .font(.title2)
                    .fontWeight(.semibold)
                Text("Drag and drop a photo or press\n \(Image(systemName: "plus.app")) to choose a photo manually.")
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            case .failure:
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.system(size: 40))
                    .foregroundColor(.white)
            }
        }
        .padding()
    }
}

@MainActor
class FilterModel: ObservableObject {
    static let shared = FilterModel()

    enum ImageState {
        case empty, loading(Progress), success(Image), failure(Error)
    }

    @Published private(set) var processedImage: Image?
    @Published var imageState: ImageState = .empty
    @Published var imageSelection: PhotosPickerItem? = nil {
        didSet {
            if let imageSelection = imageSelection {
                let progress = loadTransferable(from: imageSelection)
                imageState = .loading(progress)
            } else {
                imageState = .empty
            }
        }
    }

    func applyFilter() { /* Apply your filter */}

    func deleteCurrentPhoto() {}

    private func loadTransferable(from imageSelection: PhotosPickerItem) -> Progress {
        return imageSelection.loadTransferable(type: Image.self) { result in
            DispatchQueue.main.async {
                guard imageSelection == self.imageSelection else { return }
                switch result {
                case .success(let image?):
                    self.imageState = .success(image)
                case .success(nil):
                    self.imageState = .empty
                case .failure(let error):
                    self.imageState = .failure(error)
                }
            }
        }
    }
}
```

## Transferable

Used to power SwiftUI features like drag and drop.  Makes it easy to drop iamges from other apps into our gallery.

```swift
import PhotosUI
import CoreTransferable

struct ContentView: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        NavigationStack {
            Gallery()
                .navigationTitle("Birthday Filter")
                .toolbar {
                    PhotosPicker(
                        selection: $viewModel.imageSelection,
                        matching: .images
                    ) {
                        Label("Pick a photo", systemImage: "plus.app")
                    }
                    if let item = viewModel.processedImage {
                        ShareLink(
                            item: item, preview: SharePreview("Birthday Effects"))
                    }
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                }
                .contextMenu {
                    Button {
                        viewModel.applyFilter()
                    } label: {
                        Label("Apply Filter", systemImage: "camera.filters")
                    }
                    if let item = viewModel.processedImage {
                        ShareLink(
                            item: item, preview: SharePreview("Birthday Effects"))
                    }
                    Button(role: .destructive) {
                        viewModel.deleteCurrentPhoto()
                    } label: {
                        Label("Delete", systemImage: "trash")
                    }
                }
                .dropDestination(payloadType: Image.self) { receivedImages, location in
                    guard let image = receivedImages.first else {
                        return false
                    }
                    viewModel.imageState = .success(image)
                    return true
                }
        }
    }
}

struct Gallery: View {
    @ObservedObject var viewModel: FilterModel = .shared

    var body: some View {
        VStack {
            switch viewModel.imageState {
            case .success(let image):
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .draggable(image)
            case .loading:
                ProgressView()
            case .empty:
                Text("No Photo \(Image(systemName: "photo"))")
                    .font(.title2)
                    .fontWeight(.semibold)
                Text("Drag and drop a photo or press\n \(Image(systemName: "plus.app")) to choose a photo manually.")
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            case .failure:
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.system(size: 40))
                    .foregroundColor(.white)
            }
        }
        .padding()
    }
}

@MainActor
class FilterModel: ObservableObject {
    static let shared = FilterModel()

    enum ImageState {
        case empty, loading(Progress), success(Image), failure(Error)
    }

    @Published private(set) var processedImage: Image?
    @Published var imageState: ImageState = .empty
    @Published var imageSelection: PhotosPickerItem? = nil {
        didSet {
            if let imageSelection = imageSelection {
                let progress = loadTransferable(from: imageSelection)
                imageState = .loading(progress)
            } else {
                imageState = .empty
            }
        }
    }

    func applyFilter() { /* Apply your filter */}

    func deleteCurrentPhoto() {}

    private func loadTransferable(from imageSelection: PhotosPickerItem) -> Progress {
        return imageSelection.loadTransferable(type: Image.self) { result in
            DispatchQueue.main.async {
                guard imageSelection == self.imageSelection else { return }
                switch result {
                case .success(let image?):
                    self.imageState = .success(image)
                case .success(nil):
                    self.imageState = .empty
                case .failure(let error):
                    self.imageState = .failure(error)
                }
            }
        }
    }
}
```

[[Meet Transferable]]

# Graphics and layout
## Shape Styles
New APIS for rich effects.  we'll use these APIs to give this guest card some pop.

color has a new graident property that addss a subtle gradient property to the color.

```swift
struct CalendarIcon: View {
    var body: some View {
        VStack {
            Image(systemName: "calendar")
                .font(.system(size: 80, weight: .medium))
            Text("June 6")
        }
        .background(in: Circle().inset(by: -20))
        .backgroundStyle(
            .blue
            .gradient
        )
        .foregroundStyle(.white.shadow(.drop(radius: 1, y: 1.5)))
        .padding(20)
    }
}
```

also a new shadow modifier.  Applied to every element of the symbol.  With the whole world of SFSymbols and the new swiftUI shape style extensions, you can make some great icons.

Previews ahve been a way to see different ocnfigurations.  We're making this easier with preview variants.  Lets you develop your view in multipel apeparances, type sizes, etc. at the same time.

Previews now run in live mode by default.

```swift
struct Icon: View {
    let systemSymbolName: String
    let color: Color
    let shadow: ShadowStyle
    var foregroundColor: Color = .white

    var body: some View {
        VStack {
            Image(systemName: systemSymbolName)
                .resizable()
                .aspectRatio(1.0, contentMode: .fit)
                .padding(2)
        }
        .background(in: Circle().inset(by: -20))
        .backgroundStyle(
            color
            .gradient
        )
        .foregroundStyle(foregroundColor.shadow(shadow))
        .padding(20)
    }
}

private let dropStyle = ShadowStyle.drop(radius: 1, y: 1.5)
private let innerStyle = ShadowStyle.inner(radius: 1.5)

let icons: [Icon]  = [
    Icon(systemSymbolName: "person", color: .red, shadow: dropStyle),
    Icon(systemSymbolName: "basketball", color: .orange, shadow: dropStyle),
    Icon(systemSymbolName: "globe.central.south.asia", color: .yellow, shadow: innerStyle),
    Icon(systemSymbolName: "carrot", color: .green, shadow: innerStyle, foregroundColor: .orange),
    Icon(systemSymbolName: "sailboat", color: .mint, shadow: innerStyle),
    Icon(systemSymbolName: "figure.open.water.swim", color: .teal, shadow: dropStyle),
    Icon(systemSymbolName: "ladybug.fill", color: .cyan, shadow: innerStyle),
    Icon(systemSymbolName: "calendar", color: .blue, shadow: dropStyle),
    Icon(systemSymbolName: "moon.stars", color: .indigo, shadow: dropStyle),
    Icon(systemSymbolName: "brain.head.profile", color: .purple, shadow: innerStyle),
    Icon(systemSymbolName: "birthday.cake", color: .pink, shadow: dropStyle),
    Icon(systemSymbolName: "house.circle.fill", color: .white, shadow: dropStyle),
    Icon(systemSymbolName: "lizard", color: .brown, shadow: dropStyle),
    Icon(systemSymbolName: "flag.checkered", color: .black, shadow: dropStyle),
    Icon(systemSymbolName: "character.book.closed", color: .gray, shadow: dropStyle),
]

struct IconGrid: View {
    var body: some View {
        Grid(horizontalSpacing: 16, verticalSpacing: 16) {
            ForEach(0..<3) { i in
                GridRow {
                    ForEach(0..<5) { j in
                        icons[i * 5 + j]
                    }
                }
            }
        }
        .background(.black.opacity(0.8))
    }
}
```

```swift
// MARK: - Dancing Symbol Grid

struct SymbolSquare: View {
    let color: Color
    let imageName: String
    var image: some View {
        Image(systemName: imageName)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .padding()
            .frame(maxWidth: .infinity, maxHeight: .infinity)
    }

    var body: some View {
        image
            .background {
                RoundedRectangle(cornerRadius: 6, style: .continuous)
                    .fill(
                        .ellipticalGradient(
                            color
                                .gradient
                        )
                    )
            }
    }
}

/// If `true`, the party will commence. 
private let startTheParty = false

private let partySymbols = ["party.popper", "balloon", "balloon.2", "birthday.cake"]

struct DancingSymbolSquare: View {
    let color: Color
    let imageName: String
  
    /// Allows staggered dancing — doesn't look quite as nice.
    let seed: Int
    private let timer = Timer.publish(every: 0.234378662, on: .main, in: .default)
    @State private var cancellable: Cancellable? = nil
    @State private var heavy = false
    @State var fontSize = 20 as CGFloat

    var body: some View {
        SymbolSquare(color: color, imageName: imageName)
            .font(.body.weight(heavy ? .black : .thin))
            .onReceive(timer) { date in
                if heavy {
                    withAnimation(.easeOut(duration: 0.468757324 - 0.1)) {
                        heavy.toggle()
                    }
                } else {
                    withAnimation(.easeIn(duration: 0.1)) {
                        heavy.toggle()
                    }
                }
            }
            .onAppear {
                if startTheParty {
                    DispatchQueue.main.asyncAfter(deadline: .now()  + Double(seed) * 0.25) {
                        cancellable = timer.connect()
                    }
                }
            }
            .drawingGroup(opaque: true)
    }
}

struct SymbolGrid: View {
    var body: some View {
        Grid {
            GridRow {
                DancingSymbolSquare(color: .yellow, imageName:partySymbols[0], seed: 0)
                DancingSymbolSquare(color: .green, imageName: partySymbols[1], seed: 0)
            }

            GridRow {
                DancingSymbolSquare(color: .indigo, imageName: partySymbols[2], seed: 0)
                DancingSymbolSquare(color: .purple, imageName: partySymbols[3],  seed: 0)
            }
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```


```swift
struct TextTransitionsView: View {
    @State private var expandMessage = true
    private let mintWithShadow: AnyShapeStyle = AnyShapeStyle(Color.mint.shadow(.drop(radius: 2)))
    private let primaryWithoutShadow: AnyShapeStyle = AnyShapeStyle(Color.primary.shadow(.drop(radius: 0)))

    var body: some View {
        Text("Happy Birthday SwiftUI!")
            .font(expandMessage ? .largeTitle.weight(.heavy) : .body)
            .foregroundStyle(expandMessage ? mintWithShadow : primaryWithoutShadow)
            .onTapGesture { withAnimation { expandMessage.toggle() }}
            .frame(maxWidth: expandMessage ? 160 : 250)
            .drawingGroup()
            .padding(20)
            .background(.pink.opacity(0.3), in: RoundedRectangle(cornerRadius: 6))
    }
}
```

Has taken text and image animations to the next level.  Text can now be beautifully animated between weights, styles, and even layouts.  This takes advantage of the same animation APIs used throughout the rest of swiftUI.

## Layout
Grid, is a new ocntianer view that arranges views in a 2d grid.  It will measure subviews up front to enable cells that sapan multiple colors and enable autoamtic alignments across roews and column.

```swift
struct VIPDetailView: View {
    var body: some View {
        Grid {
            GridRow {
                NameHeadline()
                    .gridCellColumns(2)
            }
            GridRow {
                CalendarIcon()
                SymbolGrid()
            }
        }
        .frame(width: 300, height: 300)
    }
}

struct NameHeadline: View {
    var body: some View {
        HStack {
            Color.green.background(in: RoundedRectangle(cornerRadius: 8))
                .frame(maxWidth: .infinity, maxHeight: .infinity)
            VStack(alignment: .leading) {
                Text("Franck Ndame Mpouli")
                    .font(.title2)
                    .foregroundStyle(.shadow(.drop(radius: 2, y: 3)))
                Text("Party Planning Committee").bold()
            }
        }
        .padding()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(
            .white.gradient,
            in: RoundedRectangle(cornerRadius: 12, style: .continuous)
        )
    }
}

struct CalendarIcon: View {
    var body: some View {
        VStack {
            Image(systemName: "calendar")
                .font(.system(size: 80, weight: .medium))
            Text("June 6")
        }
        .background(in: Circle().inset(by: -20))
        .backgroundStyle(
            .blue
            .gradient
        )
        .foregroundStyle(.white.shadow(dropStyle))
        .padding(20)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

Build upa  grid piecemeal.  Like all layouts, they're built for composition.

We introduced SwiftUI's layuot model with the first release providing a toolbox of primitive layout types to achieve most common layouts.  most of the time,y ou can get the job done with these primitive layout types.  But sometimes you want that imperative layout types.  

in times like these, reach for new `Layout` protocol.  With it, you have the full power and flexibility we used to implement stacks and grids to build your own first-class layout abstractions.  Using layouts, I built this seating chart layout.


```swift
// MARK: Custom Table Layout

private let tableSize = CGSize(width: 130, height: 90)
private let guestSize = CGSize(width: 40, height: 40)

/// Which of 6 tables this view represents
private struct TableViewLayoutKey: LayoutValueKey {
    static let defaultValue: Int? = nil
}

extension View {
    fileprivate func tableViewLayoutKey(_ value: Int) -> some View  {
        return layoutValue(key: TableViewLayoutKey.self, value: value)
    }
}

/// Which of 36 guests this view represents
private struct GuestViewLayoutKey: LayoutValueKey {
    static let defaultValue: Int? = 0
}

extension View {

    /// Guests 1 - 36
    fileprivate func guestViewLayoutKey(_ value: Int) -> some View  {
        return layoutValue(key: GuestViewLayoutKey.self, value: value)
    }
}

let initials = [
"Ju",
"As",
"Ma",
"As",
"Ly",
"Ga",
"Ni",
"Ar",
"Ca",
"Do",
"Je",
"Ca",
"Em",
"Ma",
"Ze",
"Jo",
"Da",
"Sh",
"Sa",
"Pl",
"Pa",
"Sc",
"Ma",
"Je",
"Li",
"Ma",
"Ta",
"Je",
"Cu",
"Lu",
"Ra",
"Na",
"Sa",
"Pa",
"Le",
"Pi",
]

struct SeatingChartView: View {

    /// If true, the guests will be positioned in "pods" of tables. No table will touch another table. Otherwise
    /// the guests will side in two longs rows.
    @State private var usePods = true

    var body: some View {
        ZStack(alignment: .bottomTrailing) {
            GeometryReader { proxy in
                SeatingLayout(usePods: usePods).callAsFunction {
                    TableView(tableNumber: 1)
                    TableView(tableNumber: 2)
                    TableView(tableNumber: 3)
                    TableView(tableNumber: 4)
                    TableView(tableNumber: 5)
                    TableView(tableNumber: 6)
                    ForEach(1..<37) { i in
                        SeatedGuestOption2(guestNumber: i - 1)
                    }
                }
                .animation(.default, value: proxy.size)
            }
            .background(.black.opacity(0.13))
            Picker("Arrangement", selection: $usePods.animation()) {
                Text("Pods").tag(true)
                Text("Rows  ").tag(false)
            }
            .fixedSize()
            .pickerStyle(.segmented)
            .padding()
        }
    }
}

/// heh.
struct TableView: View {
    let tableNumber: Int

    var body: some View {
        ZStack(alignment: .bottomTrailing) {
            HStack {
                Image(systemName: "table.furniture")
                    .background(.quaternary.shadow(.inner(radius: 1, y: 1.5)),
                                in: Circle().inset(by: -8))
                    .padding(5)
                Text("Table \(tableNumber)")
            }
            .foregroundStyle(.secondary)
            .padding(8)
            .frame(width: tableSize.width, height: tableSize.height)
            #if os(macOS) || os(iOS)
            .background(.regularMaterial.shadow(.drop(radius: 1, y: 1.5)),
                        in: RoundedRectangle(cornerRadius: 12, style: .continuous))
            #endif
        }

        .tableViewLayoutKey(tableNumber)
    }
}

private let colors: [Color] = [
    .red, .orange, .yellow, .green, .mint, .teal, .cyan, .blue,
    .indigo, .purple, .pink, .gray, .black, .white, .brown,
    .red, .orange, .yellow, .green, .mint, .teal, .cyan, .blue,
    .indigo, .purple, .pink, .gray, .black, .white, .brown, .red,
    .orange, .yellow, .green, .mint, .teal, .cyan
]

struct SeatedGuest: View {

    let guestNumber: Int

    var body: some View {
        Image(systemName: "person")
            .resizable()
            .aspectRatio(contentMode: .fit)
            .padding(9)
            .background(in: Circle())
            .backgroundStyle(
                colors[guestNumber].gradient
            )
            .foregroundStyle(guestNumber == 13 ? .black : .white)
            .frame(width: 40, height: 40)
            .guestViewLayoutKey(guestNumber + 1)
    }
}

struct SeatedGuestOption2: View {
    let guestNumber: Int

    var body: some View {
        Circle()
            .stroke(colors[guestNumber], style: StrokeStyle(lineWidth: 3))
            .background(.white.gradient, in: Circle())
            .frame(width: guestSize.width, height: guestSize.height)
            .guestViewLayoutKey(guestNumber + 1)
            .overlay {
                Text(initials[guestNumber])
                    .foregroundColor(.secondary)
                    .font(.callout)

            }
    }
}

struct SeatingChartView_Previews: PreviewProvider {
    static var previews: some View {
        SeatingChartView()
            .frame(width: 600, height: 600)
    }
}

struct SeatingLayout: Layout {

    /// If true, the guests will be positioned in "pods" of tables. No table will touch another table. Otherwise
    /// the guests will side in two longs rows.
    let usePods: Bool

    struct Cache {
        ///  The width proposed to the view. We assume a certain height, otherwise, overlapping views
        var width: CGFloat?
    }

    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: LayoutSubviews,
        cache: inout Cache
    ) -> CGSize {
        cache.width = proposal.width
        return proposal.replacingUnspecifiedDimensions()
    }

    func makeCache(subviews: Subviews) -> Cache { Cache() }

    func placeSubviews(in bounds: CGRect,
                       proposal: ProposedViewSize,
                       subviews: Subviews,
                       cache: inout Cache) {
        guard let width = cache.width else { return }

        /// Helper function: Place 6 guests around all edges of a table.
        func seat(_ guests: [LayoutSubview], around table: CGRect) {
            guests[0].place(
                at: .init(
                    x: table.origin.x + 3 - guestSize.width,
                    y: table.origin.y + (table.height / 2.0) - (guestSize.height / 2.0)),
                proposal: .infinity)
            guests[1].place(
                at: .init(
                    x: table.origin.x + (table.width / 4.0) - guestSize.width / 2.0,
                    y: table.origin.y + 5 - guestSize.height),
                proposal: .infinity)
            guests[2].place(
                at: .init(
                    x: table.origin.x + table.width * 0.75 - guestSize.width / 2.0,
                    y: table.origin.y + 5 - guestSize.height),
                proposal: .infinity)
            guests[3].place(
                at: .init(
                    x: table.maxX - 5,
                    y: table.origin.y + (table.height / 2.0) - (guestSize.height / 2.0)),
                proposal: .infinity)
            guests[4].place(
                at: .init(
                    x: table.origin.x + table.width * 0.75 - guestSize.width / 2.0,
                    y: table.maxY - 5),
                proposal: .infinity)
            guests[5].place(
                at: .init(
                    x: table.origin.x + (table.width / 4.0) - guestSize.width / 2.0,
                    y: table.maxY - 5),
                proposal: .infinity)
        }

        /// Helper function: Place 6 guests, dining hall style (not along the shorter sides of a table)
        func seat(_ guests: [LayoutSubview], along table: CGRect) {
            guests[0].place(
                at: .init(
                    x: table.minX + tableSize.width / 3 - guestSize.width - 4,
                    y: table.origin.y + 5 - guestSize.height),
                proposal: .infinity)
            guests[1].place(
                at: .init(
                    x: table.minX + tableSize.width * 2/3 - guestSize.width - 4,
                    y: table.origin.y + 5 - guestSize.height),
                proposal: .infinity)
            guests[2].place(
                at: .init(
                    x: table.minX + tableSize.width - guestSize.width - 4,
                    y: table.origin.y + 5 - guestSize.height),
                proposal: .infinity)
            guests[3].place(
                at: .init(
                    x: table.minX + tableSize.width / 3 - guestSize.width - 4,
                    y: table.maxY - 5),
                proposal: .infinity)
            guests[4].place(
                at: .init(
                    x: table.minX + tableSize.width * 2/3 - guestSize.width - 4,
                    y: table.maxY - 5),
                proposal: .infinity)
            guests[5].place(
                at: .init(
                    x: table.minX + tableSize.width - guestSize.width - 4,
                    y: table.maxY - 5),
                proposal: .infinity)
        }

        // Get tables
        let table1 = subviews.first(where: { $0[TableViewLayoutKey.self] == 1 })!
        let table2 = subviews.first(where: { $0[TableViewLayoutKey.self] == 2 })!
        let table3 = subviews.first(where: { $0[TableViewLayoutKey.self] == 3 })!
        let table4 = subviews.first(where: { $0[TableViewLayoutKey.self] == 4 })!
        let table5 = subviews.first(where: { $0[TableViewLayoutKey.self] == 5 })!
        let table6 = subviews.first(where: { $0[TableViewLayoutKey.self] == 6 })!

        // Get guests
        let table1Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 1 && guestNumber <= 6
            }
        let table2Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 7 && guestNumber <= 12
            }
        let table3Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 13 && guestNumber <= 18
            }
        let table4Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 19 && guestNumber <= 24
            }
        let table5Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 25 && guestNumber <= 30
            }
        let table6Guests = subviews
            .filter {
                guard let guestNumber = $0[GuestViewLayoutKey.self] else { return false }
                return guestNumber >= 31 && guestNumber <= 36
            }

        if usePods {
            let table1Origin = CGPoint(x: 60, y: 120)
            let table2Origin = CGPoint(x: 200, y: 280)
            let table3Origin = CGPoint(x: 50, y: 450)
            let table4Origin = CGPoint(x: 300, y: 120)
            let table5Origin = CGPoint(x: 440, y: 280)
            let table6Origin = CGPoint(x: 290, y: 450)
            table1.place(at: table1Origin, proposal: .infinity)
            table2.place(at: table2Origin, proposal: .infinity)
            table3.place(at: table3Origin, proposal: .infinity)
            table4.place(at: table4Origin, proposal: .infinity)
            table5.place(at: table5Origin, proposal: .infinity)
            table6.place(at: table6Origin, proposal: .infinity)
            seat(table1Guests, around: CGRect(origin: table1Origin, size: tableSize))
            seat(table2Guests, around: CGRect(origin: table2Origin , size: tableSize))
            seat(table3Guests, around: CGRect(origin: table3Origin, size: tableSize))
            seat(table4Guests, around: CGRect(origin: table4Origin, size: tableSize))
            seat(table5Guests, around: CGRect(origin: table5Origin , size: tableSize))
            seat(table6Guests, around: CGRect(origin: table6Origin, size: tableSize))
        } else {
            let table1Origin = CGPoint(x: width / 2.0 - 6 - tableSize.width * 1.5, y: 130)
            let table2Origin = CGPoint(x: table1Origin.x + tableSize.width + 6, y: 130)
            let table3Origin = CGPoint(x: table2Origin.x + tableSize.width + 6, y: 130)
            let table4Origin = CGPoint(x: width / 2.0 - 6 - tableSize.width * 1.5, y: 360)
            let table5Origin = CGPoint(x: table1Origin.x + tableSize.width + 6, y: 360)
            let table6Origin = CGPoint(x: table2Origin.x + tableSize.width + 6, y: 360)
            table1.place(at: table1Origin, proposal: .infinity)
            table2.place(at: table2Origin, proposal: .infinity)
            table3.place(at: table3Origin, proposal: .infinity)
            table4.place(at: table4Origin, proposal: .infinity)
            table5.place(at: table5Origin, proposal: .infinity)
            table6.place(at: table6Origin, proposal: .infinity)
            seat(table1Guests, along: CGRect(origin: table1Origin, size: tableSize))
            seat(table2Guests, along: CGRect(origin: table2Origin , size: tableSize))
            seat(table3Guests, along: CGRect(origin: table3Origin, size: tableSize))
            seat(table4Guests, along: CGRect(origin: table4Origin, size: tableSize))
            seat(table5Guests, along: CGRect(origin: table5Origin , size: tableSize))
            seat(table6Guests, along: CGRect(origin: table6Origin, size: tableSize))
        }
    }
}
```

To learn more,

* grid
* layout
* viewthatfits
* anylayout

[[Compose custom layouts with SwiftUI]]

Using new AnyLayout I can switch between gridl ayout and my custom layout.

```swift
import SwiftUI
import GameplayKit
import Combine

@main
struct InvitationApp: App {
    var body: some Scene {
        WindowGroup {
            PolygonDesignerView()
                .environmentObject(PolygonModel())
            #if os(iOS)
                .statusBar(hidden: true)
            #endif
                .edgesIgnoringSafeArea(.all)
        }
    }
}

// MARK: Views

/// A view that arranges polygons in a grid, or a custom, scattered layout.
private struct DynamicPolygonView: View {
    @EnvironmentObject var model: PolygonModel
    @Binding var cycleLayouts: Bool

    private var sideLength: Int {
        Int(CGFloat(model.polygonGeometries.count).squareRoot())
    }

    /// Timer whose ticking dictates how often to regenerate and animate-to a new scattered layout.
    /// - Note: The layout will only transition if `cycleLayouts` is `true`.
    private let layoutChangingTimer = Timer
        .publish(every: 1.2, on: .current, in: .default).autoconnect()

    /// Animation used to transition layouts
    private let animation = Animation.easeInOut(duration: 1.3)

    /// Timer that ticks at 128 beats per minute, matching the beat of the song in the WWDC session.
    let musicBeatTimer = Timer
        .publish(every: 0.234378662, tolerance: 0,  on: .main, in: .default)

    @State private var musicBeatTimerCancellable: (any Cancellable)? = nil

    /// Whether or not the font should be rendered heavy.
    @State private var heavy: Bool = false

    @State private var scatteredLayout = newScatteredLayout(
        Date(timeIntervalSince1970: 0)
    )

    /// By providing a seed value, the `ScatteredLayout` struct will know when to bust its cache and
    /// generate new layout data.
    private static func newScatteredLayout(_ seed: Date) -> ScatteredLayout {
        ScatteredLayout(count: PolygonModel.total,
                        seed: seed.timeIntervalSinceReferenceDate,
                        textAvoidanceRect: CGRect(
                            x: 152,
                            y: 245,
                            width: 220,
                            height: 40)
        )
    }

    var body: some View {
        let layout = model.usesGridLayout
        ? AnyLayout(Grid(alignment: .center,
                         horizontalSpacing: 0,
                         verticalSpacing: 0))
        : AnyLayout(scatteredLayout)

        ZStack(alignment: .center) {
            Label(title:  {
                Text("You're Invited")
            }, icon: { Image(systemName: "party.popper.fill")})
            .font(.system(size:100).weight(heavy ? .black : .thin))
            .onTapGesture {
                musicBeatTimerCancellable = musicBeatTimer.connect()
            }
            .zIndex(-1)

            layout {
                ForEach((0..<sideLength), id: \.self) { row in
                    GridRow { // GridRow is a no-op in non-Grid layouts
                        ForEach((0..<sideLength), id: \.self) { column in
                            let polygon = model
                                .polygonGeometries[sideLength * row + column]
                            PolygonView(polygonGeometry: polygon)
                                .polygonViewLayoutKey(polygon)
                        }
                    }
                }
            }
        }
        .drawingGroup()
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .onReceive(musicBeatTimer) { date in
            if heavy {
                // Transitioning to a thin font happens slowly
                withAnimation(.easeOut(duration: 0.468757324 - 0.1)) {
                    heavy.toggle()
                }
            } else {
                // Transitioning to thick happens quickly, to give the
                // appearance of a "strong" downbeat
                withAnimation(.easeIn(duration: 0.1)) {
                    heavy.toggle()
                }
            }
        }
        .onReceive(layoutChangingTimer) { date in
            guard cycleLayouts else { return }
            withAnimation(animation) {
                scatteredLayout = DynamicPolygonView.newScatteredLayout(date)
            }

        }
    }
}

private struct PolygonDesignerView: View {
    @EnvironmentObject var model: PolygonModel
    @State var cycleLayouts = false
    @State var hideDesignerView = true

    var body: some View {
        ZStack(alignment: .bottom) {
            DynamicPolygonView(cycleLayouts: $cycleLayouts)
                .onTapGesture(count: 2) {
                    withAnimation {
                        hideDesignerView.toggle()
                    }
                }
            ControlView(cycleLayouts: $cycleLayouts)
                .padding()
                .background(.thickMaterial)
                .offset(CGSize(width: 0, height: hideDesignerView ? 300 : 0))
        }
    }
}

/// Tunes the parameters of a `PolygonModel`
private struct ControlView: View {

    /// The instance `self` tunes the parameters of.
    @EnvironmentObject var model: PolygonModel

    /// Can be used by a parent view to cycle through instances of layouts.
    @Binding var cycleLayouts: Bool

    var body: some View {
        VStack {
            Button("Reset", action: model.reset)
            let layout = HStack()
            layout {
                Toggle("Tiled", isOn: Binding(get: {
                    model.tiled
                }, set: { tile in
                    // After toggled, wait 5 seconds, then transition back to a
                    // scattered layout
                    DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
                        withAnimation(.linear(duration: 1.4)) {
                            model.usesGridLayout = false
                            model.drawAsRandomPolygons = true
                        }
                    }

                    withAnimation(.linear(duration: 1.8)) {
                        model.usesGridLayout = tile
                        model.drawAsRandomPolygons = !tile
                    }
                }))

                Toggle("Cycle Layouts", isOn: $cycleLayouts)
            }
        }
        .padding(2)
    }
}

// MARK: PolygonView

/// Wraps a ``Polygon`` shape applying a fill.
private struct PolygonView: View {
    var polygonGeometry: PolygonGeometry

    var body: some View {
        Polygon(polygonGeometry: polygonGeometry)
            .fill(polygonGeometry.color)
    }
}

/// A Polygon shape that supports any number of sides as defined by `polygonGeometry`
private struct Polygon: Shape {
    var polygonGeometry: PolygonGeometry

    typealias AnimatableData = AnimatableVector

    var animatableData: AnimatableVector {
        get { polygonGeometry.vectorPath }
        set { polygonGeometry.points = newValue.points }
    }

    func path(in rect: CGRect) -> Path {
        // Scale up the shape's path to fill as much space as it is given
        let path = polygonGeometry.path
        let boundingRect = path.boundingRect

        let xScale = rect.width / boundingRect.width
        let yScale = rect.height / boundingRect.height

        let translate = CGAffineTransform(
            translationX: -boundingRect.origin.x * xScale,
            y: -boundingRect.origin.y * yScale
        )
        let scale = CGAffineTransform(scaleX: xScale, y: yScale)
        return path.applying(scale.concatenating(translate))
    }

    func sizeThatFits(_ proposal: ProposedViewSize) -> CGSize {
        if proposal == .infinity {
            // If proposed infinite space, use the preferred, absolute size.
            return CGSize(width: polygonGeometry.sideLength,
                          height: polygonGeometry.sideLength)
        } else {
            // If we don't have infinite space, assume we've been given all the
            // space the parent view can afford, and take all of it.
            return proposal.replacingUnspecifiedDimensions()
        }
    }
}

// MARK: ScatteredLayout

private struct PolygonViewLayoutKey: LayoutValueKey {
    static let defaultValue: PolygonGeometry? = nil
}

extension View {
    fileprivate func polygonViewLayoutKey(_ value: PolygonGeometry)
    -> some View {
        return layoutValue(key: PolygonViewLayoutKey.self, value: value)
    }
}

/// ScatteredLayout assumes a certain standard size and lays out its views
/// (tagged with `PolygonViewLayoutKey` data) such that they don't collide
/// within that size. As the size grows, the shapes stay the same size,
/// but get farther or closer.
private struct ScatteredLayout: Layout {

    /// Cache data for a `ScatteredLayout`.
    struct Cache {

        /// Maps a `PolygonGeometry.id` to its position in a `standardSize`
        /// coordinate space.
        var rects: [UUID: CGRect]

        /// Used as a cache buster.
        var seed: TimeInterval?
    }

    /// The smallest size a view using this layout can be.
    private let minimumBaseSize: CGSize

    /// The base coordinate system this view assumes when laying out.
    private let standardSize: CGSize = CGSize(width: 500, height: 500)

    /// Clients can pass a value here and polygons won't be placed in that rect.
    var textAvoidanceRect: CGRect = .zero

    /// If different, we've been requested to bust the cache, and create a new
    /// one.
    /// - Note the cache can persist across different instances of a
    ///  `ScatteredLayout`
    private let seed: TimeInterval

    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: LayoutSubviews,
        cache: inout Cache
    ) -> CGSize {
        let proposedSize = proposal
            .replacingUnspecifiedDimensions(by: minimumBaseSize)
        return CGSize(
            width: proposedSize.width
                .clamped(
                    to: minimumBaseSize.width..<CGFloat.greatestFiniteMagnitude
                ),
            height: proposedSize.height
                .clamped(
                    to: minimumBaseSize.height..<CGFloat.greatestFiniteMagnitude
                )
        )
    }

    init(count: Int, seed: TimeInterval, textAvoidanceRect: CGRect = .zero) {
        self.seed = seed
        minimumBaseSize = CGSize(width: CGFloat(count), height: CGFloat(count))
        self.textAvoidanceRect = textAvoidanceRect
    }

    func makeCache(subviews: Subviews) -> Cache {
        var cache: Cache =  Cache(rects: [:], seed: self.seed)
        var placedPolygons: [CGRect] = []

        for subview in subviews {

            guard let polygon = subview[PolygonViewLayoutKey.self] else {
                // This is the title text view, skip it.
                continue
            }

            var subviewsPreferredSize = subview.sizeThatFits(.infinity)
            var counter = 20

            while counter > 0 {
                counter -= 1
                let randomX = CGFloat.random(in: 0..<standardSize.width)
                let randomY: CGFloat
                if randomX > textAvoidanceRect.minX
                    && randomX < textAvoidanceRect.maxX {
                    // Pick from either above or below the avoidance rect
                    if Bool.random() {
                        randomY = CGFloat.random(
                            in: 0..<textAvoidanceRect.minY
                        )
                    } else {
                        randomY = CGFloat.random(
                            in: textAvoidanceRect.maxY..<standardSize.height
                        )
                    }
                } else {
                    randomY = CGFloat.random(in: 0..<standardSize.height)
                }

                let origin = CGPoint(x: randomX, y: randomY)
                let rect = CGRect(origin: origin, size: subviewsPreferredSize)

                if placedPolygons.allSatisfy({ placed in
                    !placed.intersects(rect)
                }) && !rect.intersects(textAvoidanceRect) {
                    // The shape found a non-overlapping place to be. Lock in
                    // it's position
                    placedPolygons.append(rect)
                    cache.rects[polygon.id] =
                    CGRect(origin: origin,
                           size: subviewsPreferredSize)
                    break
                } else  {
                    if (counter == 0) {
                        if rect.intersects(textAvoidanceRect) {
                            subviewsPreferredSize = .zero
                        }
                        placedPolygons.append(rect)
                        cache.rects[polygon.id] =
                        CGRect(origin: origin,
                               size: subviewsPreferredSize)
                    }
                }
            }
        }
        return cache
    }

    func placeSubviews(in bounds: CGRect,
                       proposal: ProposedViewSize,
                       subviews: Subviews,
                       cache: inout Cache) {
        // We have the frame value cached (via makeCache())
        // for every view to be placed in a `standardSize` coordinate system.
        // Now we need to map that `standardSize` to the size was proposed.
        let proposedSize = proposal
            .replacingUnspecifiedDimensions(by: minimumBaseSize)
        let xProposedToBaseRatio = proposedSize.width / standardSize.width
        let yProposedToBaseRatio = proposedSize.height / standardSize.height

        for subview in subviews {
            guard let uuid = subview[PolygonViewLayoutKey.self]?.id, let rect =
                    cache.rects[uuid] else {
                let desiredSize = subview.sizeThatFits(.zero)
                let centered = desiredSize.centered(in: bounds)
                subview.place(
                    at: centered.origin,
                    proposal: ProposedViewSize(
                        width: desiredSize.width,
                        height: desiredSize.height
                    )
                )
                continue
            }

            let mappedPoint = CGPoint(x: rect.origin.x * xProposedToBaseRatio,
                                      y: rect.origin.y * yProposedToBaseRatio)

            subview.place(at: mappedPoint,
                          proposal: ProposedViewSize(width: rect.size.width,
                                                     height:rect.size.height)
            )
        }
    }

    func updateCache(_ cache: inout Cache, subviews: Subviews) {

        // Bust the cache if we've been given a new seed value
        // or if our subviews have been swapped out from underneath us.
        if self.seed != cache.seed
            || !cache.rects.contains(where: { (key: UUID, value: CGRect) in
                subviews.first?[PolygonViewLayoutKey.self]?.id == key
            })  {
            cache = makeCache(subviews: subviews)
            return
        }
    }

}

/// This struct facilitates animation of point-based `Path`s so long as said
/// source and destination `Path` have an equal number of vertices.
private struct AnimatableVector: VectorArithmetic {

    static var zero: AnimatableVector = AnimatableVector(points: [])

    private(set) var points: [CGPoint]

    var magnitudeSquared: Double {
        let squared = points.map { point in
            CGPoint(x: point.x * point.x, y: point.y * point.y)
        }
        let sumOfSquares = squared.map { point in // dot product?
            sqrt(point.x + point.y)
        }
        let sum = sumOfSquares.reduce(0, +)
        return Double(sum)
    }

    /// Facilitates a valid `.zero` value, no matter the dimension of the vector
    subscript(safe index: Int) -> CGPoint {
        return (self.points.count <= index) ? .zero : points[index]
    }

    static func - (lhs: AnimatableVector, rhs: AnimatableVector)
    -> AnimatableVector {
        let negated = rhs.points.map { CGPoint(x: -$0.x, y: -$0.y) }
        return lhs + AnimatableVector(points: negated)
    }

    static func + (lhs: AnimatableVector, rhs: AnimatableVector)
    -> AnimatableVector {
        var output: [CGPoint] = []
        for i in 0..<lhs.points.count {
            output.append(CGPoint(x: lhs[safe: i].x + rhs[safe: i].x,
                                  y:lhs[safe: i].y + rhs[safe: i].y ))
        }
        return AnimatableVector(points: output)
    }

    mutating func scale(by rhs: Double) {
        points = points.map { CGPoint(x: $0.x * CGFloat(rhs),
                                      y: $0.y * CGFloat(rhs)) }
    }
}

// MARK: Random Polygon Generation & Geometry

private let mean: Float = 10
private let deviation: Float = 3
private let gaussian = GKGaussianDistribution(
    randomSource: GKARC4RandomSource(),
    mean: mean,
    deviation: deviation)

/// Factory type for creating points describing a random Polygon
private struct PolygonGeometry: Identifiable, Equatable, Hashable {

    /// The horizontal and vertical side lengths of the polygon's bounding box.
    let sideLength: CGFloat

    /// A constant count of the total points that comprise this
    /// `PolygonGeometry`'s path. Clients can set `points` to a new value, but
    /// the new value should have the same `count` for smooth `Path` animations
    let numberOfVertices: Int

    /// Supports animation of point-based `Path`s by providing an array of
    /// points that can be interpolated.
    var vectorPath: AnimatableVector {
        AnimatableVector(points: points)
    }

    /// If `false`, this instance will present itself as a rectangular shape
    /// (not necessarily with 4 vertices) that fills available space.
    private(set) var drawsAsPolygon: Bool = true

    /// Points describing the `Path` used to render `self`.
    var points: [CGPoint] {
        willSet {
            assert(points.count == polygonPathPoints.count)
        }
    }

    /// Delineate the path of the random polygon.
    private let polygonPathPoints: [CGPoint]

    let color: Color = [
        Color(red: 0.73, green: 0.20, blue: 0.20),
        Color(red: 0.95, green: 0.66, blue: 0.24),
        Color(red: 0.14, green: 0.29, blue: 0.49),
        Color(red: 0.46, green: 0.76, blue: 0.67),
        Color(red: 0.30, green: 0.33, blue: 0.22),
        Color(red: 0.49, green: 0.55, blue: 0.64),
        Color(red: 0.92, green: 0.53, blue: 0.30),
        Color(red: 0.20, green: 0.45, blue: 0.55),
        Color(red: 0.41, green: 0.45, blue: 0.45),
        Color(red: 0.87, green: 0.67, blue: 0.61)
    ].randomElement()!

    private var spikiness: CGFloat = 0.2
    private var irregularity: CGFloat = 0.2

    let id = UUID()

    /// Owning `Shape` instances should use this to draw.
    var path: Path { Path(from: points) }

    init(pointsVector: [CGPoint], sideLength: CGFloat) {
        self.numberOfVertices = pointsVector.count
        self.points = pointsVector
        self.polygonPathPoints = points
        self.sideLength = sideLength
    }

    func drawn(asRandomizedPolygon: Bool) -> Self {
        var copy = self
        copy.drawsAsPolygon = asRandomizedPolygon
        copy.points = asRandomizedPolygon
        ? copy.polygonPathPoints
        : CGRect(x: 0, y: 0, width: 1, height: 1)
            .pointSequence(of: copy.numberOfVertices)
        return copy
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}

/// A namespace around functionality to generate a  path drawn in a 1x1 square
/// with configurable "irregularity" and "spikiness".
/// The closer both are to zero, the closer the generated polygon is to a
/// [regular polygon](https://mathworld.wolfram.com/RegularPolygon.html)
private enum UnitPolygonGeometryFactory {

    /// The maximum possible radius. A value of 0.5 restricts the algorithm
    /// to the unit square.
    private static let maxRadius: CGFloat = 0.5

    /// A — by no means definitive — algorithm for creating an arbitrary
    /// polygon of `vertexCount` vertices
    /// - Parameters:
    ///   - vertexCount: How many vertices (and edges) the polygon will have
    ///   - irregularity: A subjective term for how "irregular" the polygon is.
    ///   A fully regular polygon has all equal sides, assuming 0 `spikinesss`.
    ///   - spikiness: A subjective term for how "spiky" the polygon is.
    ///   A polygon with high spikiness will have more vertices closer and
    ///   farther from where the vertex would be on a regular polygon.
    /// - Returns: An array of points representing the point-based path of
    /// the polygon
    static func random(vertexCount: Int,
                       irregularity: CGFloat = 0.2,
                       spikiness: CGFloat = 0.2)
    -> [CGPoint] {

        let floatVertices = CGFloat(vertexCount)

        // Irregularity is how much we're willing to allow the angular steps to
        // vary from "perfect". For example, in a regular (all sides equal)
        // six-sided polygon, each angular step is 2𝜋 / 6. Irregularity
        // defines the range that value can take, centered around a mean of
        // 2𝜋 / 6. We accept an irregularity between 0 and 1, and then
        // scale it for how much that represents out of a circle's radians.
        let scaledIrregularity = irregularity * 2.0 * CGFloat.pi / floatVertices

        // Spikiness describes how often we want to see values that are very
        // far from where a vertex of a regular polygon would be. For example,
        // a high positive spikiness might push a vertex radially very far from
        // the center, leading to a big "spike". Meanwhile, a spikiness of 0
        // will yield more circular polygons.
        let denormalizedSpikiness = spikiness * maxRadius

        let gaussian = GKGaussianDistribution(
            randomSource: GKARC4RandomSource(),
            mean: Float(maxRadius * 1024),
            deviation: Float(denormalizedSpikiness * 1024))

        // Generate the angular steps
        var raidanAngleSteps: [CGFloat] = []

        // Both of these measured in radians
        let minimumSliceWidth =
        (2.0 * CGFloat.pi / floatVertices) - scaledIrregularity
        let maximumSliceWidth =
        (2.0 * CGFloat.pi / floatVertices) + scaledIrregularity

        var sum: CGFloat = 0

        for _ in (0..<vertexCount) {
            let radians = CGFloat
                .random(in: minimumSliceWidth...maximumSliceWidth)
            raidanAngleSteps.append(radians)
            sum += radians
        }

        // Re-divide these steps so the point 0 and n+1 are the same.
        // I.e. if the random angle generation from the above loop yielded
        // more or less than 2𝜋 radians, reapportion those divisions to sum to
        // 2𝜋.
        let k = sum / (2 * CGFloat.pi)
        (0..<vertexCount).forEach { i in
            raidanAngleSteps[i] /= k
        }

        let maximumPossibleGaussianSample = CGFloat(
            gaussian.mean + Float(denormalizedSpikiness * 1024)*3
        )

        // Finally, make all of the normalized points within a 1x1 square
        // Unlike the unit circle of traditional geometry, because (0, 0) is in
        // the top left, (0.5, 0.5) is in the middle. Thus, positively
        // incrementing the angle moves us clockwise around the circle
        var points: [CGPoint] = []
        let center = CGPoint(x: maxRadius, y: maxRadius)
        var cumulativeAngle: CGFloat = 0.0
        for i in (0..<Int(vertexCount)) {

            // * 2 to keep the sample <= 0.5 (`maxRadius)
            let radiusForPoint = CGFloat(gaussian.nextInt())
            / (maximumPossibleGaussianSample * 2)

            let x = center.x + radiusForPoint * cos(cumulativeAngle)
            let y = center.y + radiusForPoint * sin(cumulativeAngle)
            points.append(CGPoint(x: x, y: y))

            cumulativeAngle += raidanAngleSteps[i]
        }
        return points
    }
}

// MARK: Observable Polygon Model

/// A `PolygonModel` describes a collection of randomized ``Polygons`` that
/// can be laid out by `AnyLayout` type.
private class PolygonModel: ObservableObject {

    static let total = (maxSides - minSides + 1) * polygonsPerSideCount

    /// The minimum sides the randomly generated sides will have
    private static let minSides = 4

    /// The maximum sides the randomly generated sides will have
    private static let maxSides = 7

    /// The number of randomly generated polygons to make _per side length_.
    private static let polygonsPerSideCount = 32

    /// All `PolygonGeometry`s that are laid out with `scatteredLayout`
    @Published var polygonGeometries: [PolygonGeometry] = makeGeometries()

    /// If `true`, `self` is expressing a grid layout with rectangular tiles.
    var tiled: Bool { usesGridLayout && !drawAsRandomPolygons }

    /// If `true`, ignore `scatteredLayout` and instead use a `Grid` layout
    @Published var usesGridLayout: Bool = false

    /// If `true`, `polygonGeometries` draw themselves as randomized polygons.
    /// If false, a rectangle that fills all available space.
    @Published var drawAsRandomPolygons: Bool = true {
        didSet {
            polygonGeometries = polygonGeometries.map {
                $0.drawn(asRandomizedPolygon: drawAsRandomPolygons)
            }
        }
    }

    /// Tunable by clients to experiment with different values.
    let spikiness: CGFloat = 0.2
    /// Tunable by clients to experiment with different values.
    let irregularity: CGFloat = 0.2

    /// Creates many ``PolygonGeometry`` instances with the given parameters.
    /// - Parameters:
    ///   - irregularity: A subjective term for how "irregular" the polygon is.
    ///   A fully regular polygon has all equal sides, assuming 0 `spikinesss`.
    ///   - spikiness: A subjective term for how "spiky" the polygon is.
    ///   A polygon with high spikiness will have more vertices closer and
    ///   farther from where the vertex would be on a regular polygon.
    /// - Returns: An array of `n` polygons where `n` is defined by the
    ///  `PolygonModel` class.
    private static func makeGeometries(
        irregularity: CGFloat = 0.3,
        spikiness: CGFloat = 0.3) -> [PolygonGeometry] {
            var scales: Array<CGFloat> = polygonSizeRatios
                .reduce(into: []) { partialResult, sizeRatio in
                    let (size, percentage) = sizeRatio
                    let scalesToMake = Int(ceil(percentage * CGFloat(total)))
                    partialResult.append(contentsOf: (0..<scalesToMake)
                        .map { _ in CGFloat.random(in: size.sizeRange) })
                }.shuffled()

            return (minSides...maxSides).flatMap { vertexCount in
                return (0..<polygonsPerSideCount).map { _ in
                    let unitPolygon = UnitPolygonGeometryFactory
                        .random(vertexCount: vertexCount,
                                irregularity: irregularity,
                                spikiness: spikiness)
                    let polygonGeometry = PolygonGeometry(
                        pointsVector: unitPolygon,
                        sideLength: scales.removeFirst())
                    return polygonGeometry
                }
            }.shuffled()
        }

    /// Complete remove and regenerate all model data.
    func reset() {
        polygonGeometries.removeAll(keepingCapacity: true)
        polygonGeometries = PolygonModel.makeGeometries(
            irregularity: irregularity,
            spikiness: spikiness
        )
    }
}

private extension PolygonModel {

    /// Use a sampling of various sized polygons
    enum PieceSize: Hashable {
        case tiny
        case small
        case medium
        case large

        /// The range for the side length of the bounding rect of a polygon
        var sizeRange: ClosedRange<CGFloat> {
            switch self {
            case .tiny:
                return 16.0...25.0
            case .small:
                return 25.0...40.0
            case .medium:
                return 40.0...50.0
            case .large:
                return 50.0...65.0
            }
        }
    }

    /// This dictionary denotes the ratio of sizes to use.
    /// - warning: Should sum to 100.
    private static let polygonSizeRatios: [PieceSize: CGFloat] =
    [
        .large: 0.15,
        .medium: 0.25,
        .small: 0.25,
        .tiny: 0.35
    ]
}

// MARK: - Utility Extensions

extension FloatingPoint {

    /// - returns an instance of `Self` clamped to the ``ClosedRange``.
    func clamped(to limits: ClosedRange<Self>) -> Self {
        return min(max(self, limits.lowerBound), limits.upperBound)
    }

    /// - returns an instance of `Self` clamped to the ``Range``.
    /// - note the value returned will be less than the provided upper bound, as
    ///  is dictated by ``Range``.
    func clamped(to limits: Range<Self>) -> Self {
        return min(max(self, limits.lowerBound), limits.upperBound.nextDown)
    }
}

extension CGRect {

    /// Creates a rectangular sequence of `vertexCount `points denoting a
    /// rectangular path.
    /// - note This is helpful for animating a `Path` composed of `vertexCount`
    /// points into a ``Rectangle``.
    func pointSequence(of vertexCount: Int) -> [CGPoint] {
        // Start at a random corner. When many Polygons are using this
        // animation at once, if they all start at the same corner, an
        // unnatural uniformity of motion emerges.
        var startingPercent = [0, 0.25, 0.5, 0.75].randomElement()!
        var points: [CGPoint] = []

        let extraPoints = vertexCount - 4
        let (groups, remainder) = extraPoints
            .quotientAndRemainder(dividingBy: 3)

        for edge in 0...3 {
            points.append(pointAlongPerimeter(at: startingPercent))
            for i in (0..<(edge == 3 ? remainder : groups)) {
                points.append(pointAlongPerimeter(
                    at: startingPercent + 0.25
                    / CGFloat(groups + 1) * CGFloat(i)))
            }
            startingPercent += 0.25
            startingPercent.formTruncatingRemainder(dividingBy: 1)
        }
        assert(points.count == vertexCount)
        return points
    }

    /// Returns the ``CGPoint`` that is `percent` along the path of `self`,
    /// with 0% mapping to the top-left corner, progressing clockwise.
    /// E.g. 50% would map to the bottom right corner if and only if `self` is
    ///  a square.
    /// - Parameters:
    ///   - percent: A percentage between `0.0` and `1.0`
    private func pointAlongPerimeter(at percent: CGFloat) -> CGPoint {
        let perimeter = size.width * 2 + size.height * 2

        // Mark the four corners as percentages around the rect. For example,
        /// these values for a square would be 25%, 50%, 75%, 100%
        let topRight = size.width / perimeter
        let bottomRight = topRight + (size.height / perimeter)
        let bottomLeft = bottomRight + (size.width / perimeter)
        let topLeft = 1.0

        switch percent {
        case 0..<topRight:
            return CGPoint(
                x: percent / topRight * size.width,
                y: minY)
        case topRight..<bottomRight:
            return CGPoint(
                x: maxX,
                y: (percent - topRight)
                / (bottomRight - topRight) * size.height)
        case bottomRight..<bottomLeft:
            return CGPoint(
                x: maxX - ((percent - bottomRight) / (bottomLeft - bottomRight)
                           * size.width),
                y: maxY)
        case bottomLeft...topLeft:
            return CGPoint(
                x: minX,
                y: maxY - (percent - bottomLeft) / (topLeft - bottomLeft)
                * size.height
            )
        default:
            preconditionFailure("Invalid percentage requested")
        }
    }
}

/// Returns a new `CGRect` with the same size as `self`, but centered in `other`
/// vertically, and horizontally.
extension CGSize {
    func centered(in other: CGRect) -> CGRect {
        CGRect(x: other.midX - width / 2.0,
               y: other.midY - height / 2.0,
               width: width,
               height: height)
    }
}

extension Path {
    /// Convenience for initializing a `Path` from an array of `CGPoint`s given
    /// the first point element is the `Path`'s first point.
    init(from points: [CGPoint]) {
        self.init()
        self.addLines(points)
        self.closeSubpath()
    }
}
```

Even more APIs that we didn't have time to include.





* https://developer.apple.com/forums/tags/wwdc2022-10052
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=484030
* https://developer.apple.com/documentation/SwiftUI
