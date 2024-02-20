Meet Inspectors — a structural API that can help bring a new level of detail to your apps. We'll take you through the fundamentals of the API and show you how to adopt it. Learn about the latest updates to sheet presentation customizations and find out how you can combine the two to create perfect presentation experiences.

# Inspector
exciting new element in swiftui.  I'll go over what this is, how to use, etc.

Show further detail of selected content.  Probably seen this before.

Keynote uses an inspector to show formatting details.  

May also show content that supplements main content.  see, shortcuts.  

available on macos, iPadOS, iOS!  Includes programmatic control over column width, allowing you to tune the width of the trailing column.  Programmatic control over persented state, hiding/showing as needed.

Higher-level abstraction than just the trailing sidebar.  In compact classes, it adapts to a sheet.  

Overlaps on splitscreen on (larger?) iPads.


Inspector fits inside these APIs with characteristics of both navigation components as well as presentations.  



### 3:35 - Sample models and views
```swift
// Copy+Paste the below into an Xcode project to support building and running the session's code snippets

import SwiftUI

@main
struct SwiftUIInspectors: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(AnimalStore())
        }
    }
}

struct AnimalInspectorForm: View {
    var animal: Binding<Animal>?
    @EnvironmentObject private var animalStore: AnimalStore

    var body: some View {
        Form {
            if let animal = animal {
                SelectedAnimalInspector(animal: animal, animalStore: animalStore)
            } else {
                ContentUnavailableView {
                    Image(systemName: "magnifyingglass.circle")
                } description: {
                    Text("Select a suspect to inspect")
                } actions: {
                    Text("Fill out details from the interview")
                }
            }
        }
        #if os(iOS)
        .navigationBarTitleDisplayMode(.inline)
        #endif
    }
}

struct SelectedAnimalInspector: View {
    @Binding var animal: Animal
    @ObservedObject var animalStore: AnimalStore

    var body: some View {
            Section("Identity") {
                TextField("Name", text: $animal.name)
                Picker("Paw Size", selection: $animal.pawSize) {
                    Text("Small").tag(PawSize.small)
                    Text("Medium").tag(PawSize.medium)
                    Text("Large").tag(PawSize.large)
                }
                FruitList(selectedFruits: $animal.favoriteFruits, fruits: allFruits)
            }

            Section {
                TextField(text: animalStore(\.alibi, for: animal), prompt: Text("What was \(animal.name) doing at the time of nibbling?"), axis: .vertical) {
                    Text("Alibi")
                }
                .lineLimit(4, reservesSpace: true)
                if let schedule = Binding(animalStore(\.sleepSchedule, for: animal)) {
                    SleepScheduleView(schedule: schedule)
                } else {
                    Button("Add Sleep Schedule") {
                        animalStore.write(\.sleepSchedule, value: Animal.Storage.newSleepSchedule, for: animal)
                    }
                }
                Slider(
                    value: animalStore(\.suspiciousLevel, for: animal), in: 0...1) {
                        Text("Suspicion Level")
                    } minimumValueLabel: {
                        Image(systemName: "questionmark")
                    } maximumValueLabel: {
                        Image(systemName: "exclamationmark.3")
                    }
            } header: {
                Text("Interview")
            }
            .presentationDetents([.medium, .large])
    }
}

private struct FruitList: View {
    @Binding var selectedFruits: [Fruit]
    var fruits: [Fruit]

    var body: some View {
        Section("Favorite Fruits") {
                ForEach(allFruits) { fruit in
                    Toggle(isOn: .init(get: {
                        selectedFruits.contains(fruit)
                    }, set: { newValue in
                        if newValue && !selectedFruits.contains(fruit) {
                            selectedFruits.append(fruit)
                        } else {
                            _ = selectedFruits.firstIndex(of: fruit).map {
                                selectedFruits.remove(at: $0)
                            }
                        }
                    })) {
                        HStack {
                            FruitImage(fruit: fruit, size: .init(width: 40, height: 40), bordered: true)
                            Text(fruit.name).font(.body)
                        }
                    }
                }
        }
    }

    @ViewBuilder
    private func selectionBackground(isSelected: Bool) -> some View {
        if isSelected {
            RoundedRectangle(cornerRadius: 2).inset(by: -2)
                .fill(.selection)
        }
    }
}

private struct SleepScheduleView: View {
    @Binding var schedule: Animal.Storage.SleepSchedule
    var body: some View {
        DatePicker(selection: .init(get: {
            Calendar.current.date(from: schedule.sleepTime) ?? Date()
        }, set: { newDate in
            schedule.sleepTime = Calendar.current.dateComponents([.hour, .minute], from: newDate)
        }), displayedComponents: [.hourAndMinute]) {
            Text("Sleep at: ")
        }

        DatePicker(selection: .init(get: {
            Calendar.current.date(from: schedule.wakeTime) ?? Date()
        }, set: { newDate in
            schedule.wakeTime = Calendar.current.dateComponents([.hour, .minute], from: newDate)
        }), displayedComponents: [.hourAndMinute]) {
            Text("Awake at: ")
        }
    }
}

struct AppState {
    var selection: String? = "Snail"
    var animals: [Animal] = allAnimals
    var inspectorPresented: Bool = true
    var inspectorWidth: CGFloat = 270
    var cornerRadius: CGFloat? = nil
}

extension Binding where Value == AppState {
    func binding() -> Binding<Animal>? {
        self.projectedValue.animals.first {
            $0.wrappedValue.id == self.selection.wrappedValue
        }
    }
}

extension Animal {
    struct Storage: Codable {
        var alibi: String = ""
        var sleepSchedule: SleepSchedule? = nil

        /// Value between 0 and 1 representing how suspicious the animal is.
        /// 1 is guilty.
        var suspiciousLevel: Double = 0.0

        struct SleepSchedule: Codable {
            var sleepTime: DateComponents
            var wakeTime: DateComponents
        }

        static let newSleepSchedule: SleepSchedule = {
            // Asleep at 10:30, awake at 6:30
            .init(
                sleepTime: DateComponents(hour: 22, minute: 30),
                wakeTime: DateComponents(hour: 6, minute: 30))
        }()
    }

}

final class AnimalStore: ObservableObject {

    var storage: [Animal.ID: Animal.Storage] = [:]

    /// Getter for properties of an animal stored in self
    func callAsFunction<Result>(_ keyPath: WritableKeyPath<Animal.Storage, Result>, for animal: Animal) -> Binding<Result> {
        Binding { [self] in
            storage[animal.id, default: .init()][keyPath: keyPath]
        } set: { [self] newValue in
            self.objectWillChange.send()
            var animalStore = storage[animal.id, default: .init()]
            animalStore[keyPath: keyPath] = newValue
            storage[animal.id] = animalStore
        }
    }

    func write<Value>(_ keyPath: WritableKeyPath<Animal.Storage, Value>, value: Value, for animal: Animal) {
        objectWillChange.send()
        var animalStore = storage[animal.id, default: .init()]
        animalStore[keyPath: keyPath] = value
        storage[animal.id] = animalStore
    }

    func read<Value>(_ keyPath: WritableKeyPath<Animal.Storage, Value>, for animal: Animal) -> Value {
        storage[animal.id, default: .init()][keyPath: keyPath]
    }
}

struct AnimalTable: View {
    @Binding var state: AppState
    @EnvironmentObject private var animalStore: AnimalStore
    @Environment(\.horizontalSizeClass) private var sizeClass: UserInterfaceSizeClass?


    var fruitWidth: CGFloat {
        #if os(iOS)
        40.0
        #else
        25.0
        #endif
    }

    var body: some View {
        Table(state.animals, selection: $state.selection) {
            TableColumn("Name") { animal in
                HStack {
                    Text(animal.emoji).font(.title)
                        .padding(2)
                        .background(.thickMaterial, in: RoundedRectangle(cornerRadius: 3))
                    Text(animal.name + " " + animal.species).font(.title3)
                }
            }
            TableColumn("Favorite Fruits") { animal in
                HStack {
                    ForEach(animal.favoriteFruits.prefix(3)) { fruit in
                        FruitImage(fruit: fruit, size: .init(width: fruitWidth, height: fruitWidth), scale: 2.0, bordered: state.selection == animal.id)
                    }
                }
                .padding(3.5)
            }
            TableColumn("Suspicion Level") { animal in
                SuspicionTableCell(animal: animal)
            }
        }
        #if os(macOS)
        .alternatingRowBackgrounds(.disabled)
        #endif
        .tableStyle(.inset)
    }
}

private struct SuspicionTableCell: View {
    var animal: Animal
    @Environment(\.backgroundProminence) private var backgroundProminence
    @EnvironmentObject private var animalStore: AnimalStore

    var body: some View {
        let color = SuspiciousText.model(for: animalStore.read(\.suspiciousLevel, for: animal)).1
        HStack {
            Image(
                systemName: "cellularbars",
                variableValue: animalStore.read(\.suspiciousLevel, for: animal)
            )
            .symbolRenderingMode(.hierarchical)
            SuspiciousText(
                suspiciousLevel:
                    animalStore.read(\.suspiciousLevel, for: animal),
                selected: backgroundProminence == .increased)
        }
        .foregroundStyle(backgroundProminence == .increased ? AnyShapeStyle(.white) : AnyShapeStyle(color))
    }
}

private struct SuspiciousText: View {
    var suspiciousLevel: Double
    var selected: Bool

    static fileprivate func model(for level: Double) -> (String, Color) {
        switch level {
        case 0..<0.2:
            return ("Unlikely", .green)
        case 0.2..<0.5:
            return ("Fishy", .mint)
        case 0.5..<0.9:
            return ("Very suspicious", .orange)
        case 0.9...1:
            return ("Extremely suspicious!", .red)
        default:
            return ("Suspiciously Unsuspicious", .blue)
        }
    }

    var body: some View {
        let model = Self.model(for: suspiciousLevel)
        Text(model.0)
            .font(.callout)
    }
}

struct Animal: Identifiable {
    var name: String
    var species: String
    var pawSize: PawSize
    var favoriteFruits: [Fruit]
    var emoji: String

    var id: String { species }
}

var allAnimals: [Animal] = [
    .init(name: "Fabrizio", species: "Fish", pawSize: .small, favoriteFruits: [.arbutusUnedo, .bigBerry, .elstar], emoji: "🐟"),
    .init(name: "Soloman", species: "Snail", pawSize: .small, favoriteFruits: [.elstar, .flavorKing], emoji: "🐌"),
    .init(name: "Ding", species: "Dove", pawSize: .small, favoriteFruits: [.quercusTomentella, .pinkPearlApple, .lapins], emoji: "🕊️"),
    .init(name: "Catie", species: "Crow", pawSize: .small, favoriteFruits: [.pinkPearlApple, .goldenNectar, .hauerPippin], emoji: "🐦‍⬛"),
    .init(name: "Miko", species: "Cat", pawSize: .small, favoriteFruits: [.belleDeBoskoop, .tompkinsKing, .lapins], emoji: "🐈"),
    .init(name: "Ricardo", species: "Rabbit", pawSize: .small, favoriteFruits: [.mariposa, .elephantHeart], emoji: "🐰"),
    .init(name: "Cornelius", species: "Duck", pawSize: .medium, favoriteFruits: [.greenGage, .goldenNectar], emoji: "🦆"),
    .init(name: "Maria", species: "Mouse", pawSize: .small, favoriteFruits: [.arbutusUnedo, .elephantHeart], emoji: "🐹"),
    .init(name: "Haku", species: "Hedgehog", pawSize: .small, favoriteFruits: [.christmasBerry, .creepingSnowberry, .goldenGem], emoji: "🦔"),
    .init(name: "Rénard", species: "Raccoon", pawSize: .medium, favoriteFruits: [.belleDeBoskoop, .bigBerry, .christmasBerry, .kakiFuyu], emoji: "🦝")
]

enum PawSize: Hashable {
    case small
    case medium
    case large
}

struct Fruit: Identifiable, Hashable {
    var name: String
    var color: Color
    var id: String { name }
}

struct FruitImage: View {
    var fruit: Fruit
    var size: CGSize? = .init(width: 50, height: 50)
    var scale: CGFloat = 1.0
    var bordered = false

    var body: some View {
        fruit.color // Actual assets replaced with Color
            .scaleEffect(scale)
            .scaledToFill()
            .frame(width: size?.width, height: size?.height)
            .mask { RoundedRectangle(cornerRadius: 4) }
            .overlay {
                if bordered {
                    RoundedRectangle(cornerRadius: 4)
                        .stroke(fruit.color, lineWidth: 2)
                }
            }
    }
}

extension Fruit {
    static let goldenGem = Fruit(name: "Golden Gem Apple", color: .yellow)
    static let flavorKing = Fruit(name: "Flavor King Plum", color: .purple)
    static let mariposa = Fruit(name: "Mariposa Plum", color: .red)
    static let tompkinsKing = Fruit(name: "Tompkins King Apple", color: .yellow)
    static let greenGage = Fruit(name: "Green Gage Plum", color: .green)
    static let lapins = Fruit(name: "Lapins Sweet Cherry", color: .purple)
    static let hauerPippin = Fruit(name: "Hauer Pippin Apple", color: .red)
    static let belleDeBoskoop = Fruit(name: "Belle De Boskoop Apple", color: .red)
    static let elstar = Fruit(name: "Elstar Apple", color: .yellow)
    static let goldenDeliciousApple = Fruit(name: "Golden Delicious Apple", color: .yellow)
    static let creepingSnowberry = Fruit(name: "Creeping Snowberry", color: .white)
    static let quercusTomentella = Fruit(name: "Channel Island Oak Acorn", color: .brown)
    static let elephantHeart = Fruit(name: "Elephant Heart Plum", color: .red)
    static let goldenNectar = Fruit(name: "Golden Nectar Plum", color: .yellow)
    static let pinkPearlApple = Fruit(name: "Pink Pearl Apple", color: .pink)
    static let christmasBerry = Fruit(name: "Christmas Berry", color: .red)
    static let kakiFuyu = Fruit(name: "Kaki Fuyu Persimmon", color: .orange)
    static let bigBerry = Fruit(name: "Big Berry Manzanita", color: .red)
    static let arbutusUnedo = Fruit(name: "Strawberry Tree", color: .red)
}

extension Array where Element == Fruit {
    var groupID: Fruit.ID {
        reduce("") { result, next in
            result.appending(next.id)
        }
    }
}

var allFruits: [Fruit] = [
    .goldenGem,
    .flavorKing,
    .mariposa,
    .tompkinsKing,
    .greenGage,
    .lapins,
    .hauerPippin,
    .belleDeBoskoop,
    .elstar,
    .goldenDeliciousApple,
    .creepingSnowberry,
    .quercusTomentella,
    .elephantHeart,
    .goldenNectar,
    .kakiFuyu,
    .bigBerry,
    .arbutusUnedo,
    .pinkPearlApple,
]
```

### 3:54 - Xcode Previews
```swift
import SwiftUI

 #Preview("Meet Inspector", traits: .fixedLayout(width: 800, height: 500)) {
    ContentView()
        .navigationTitle("SwiftUI Inspectors")
        .environmentObject(AnimalStore())
}
 
public struct ContentView: View {
    @State private var state = AppState()
    @State private var presented = true

    public var body: some View {
        AnimalTable(state: $state)
            .inspector(isPresented: $presented) {
                AnimalInspectorForm(animal: $state.binding())
                    .inspectorColumnWidth(
                        min: 200, ideal: 300, max: 400)
                    .toolbar {
                        Spacer()
                        Button {
                            presented.toggle()
                        } label: {
                            Label("Toggle Inspector", systemImage: "info.circle")
                        }
                    }
            }
    }
}
 
import MapKit

struct FruitNibbleBulletin: View {
    var fruit: Fruit = .pinkPearlApple
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(alignment: .leading) {
                    Grid(horizontalSpacing: 12, verticalSpacing: 2) {
                        GridRow {
                            FruitImage(fruit: fruit, size: .init(width: 60, height: 60), bordered: false)

                            Text("""
                            A \(fruit.name.lowercased()) was nibbled! The bite \
                            happened at 9:41 AM. The nibbler left behind only \
                            a few seeds.
                            """
                            )
                        }
                        GridRow {
                            Text("""
                            The Fruit Inspectors were on \
                            the scene moments after it happened. \
                            Unfortunately, their efforts to catch the nibbler \
                            were fruitless.
                            """).gridCellColumns(2)
                        }
                    }
                    GroupBox("Clues") {
                        LabeledContent("Paw Size") {
                            Text("Large")
                        }
                        LabeledContent("Favorite Fruit") {
                            Text("\(fruit.name.capitalized(with: .current))")
                        }
                        LabeledContent("Alibi") {
                            Text("None")
                        }
                    }
                    HStack {
                        VStack {
                            fruit.color
                                .aspectRatio(contentMode: ContentMode.fit)
                                .shadow(radius: 2.5)
                            Text("The pink pearls left behind").font(.caption)
                                .frame(alignment: .leading)
                        }
                        AppleParkMap()
                            .mask(RoundedRectangle(cornerSize: CGSize(width: 20, height: 10)))

                    }
                    Text("The Fruit Inspection team was on the scene minutes after the incident. However, their attempts to discover any meaningful clues around the identity of the nibbler were fruitless.")
                }
                .scenePadding(.horizontal)
                .toolbar {
                    ToolbarItem {
                        Button(role: .cancel) {
                            dismiss()
                        } label: {
                            Label("Close", systemImage: "xmark.circle.fill")
                        }
                        .symbolRenderingMode(.monochrome)
                        .tint(.secondary)
                    }
                }
            }
#if os(iOS)
            .navigationBarTitleDisplayMode(.inline)
#endif
            .navigationTitle("Fruit Nibble Bulletin")
        }
    }
}

struct AppleParkMap: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 37.334_371,
                                       longitude: -122.009_558),
        latitudinalMeters: 100,
        longitudinalMeters: 100
    )

    var body: some View {
        GeometryReader { geometry in
            Map(position: .constant(.automatic), bounds: .init(centerCoordinateBounds: region, minimumDistance: 100, maximumDistance: 100), interactionModes: [], scope: .none) { }
        }
        .frame(height: 180, alignment: .center)
    }
}
```




Inspectors are not resizable by default, need `.inspectorColumnWidth`.
* ideal - size on first launch
* system persists user resize across launches

Inspector has different behaviors depending on context.

* full height
* under toolbar -> inspector nested under toolbar.  Title separator has window width.
* similarly in iOS we can show at top of screen or top of inspector (half width)

two dimensions
inspector inside/outside navigation structure
toolbar content inside/outside inspector

when inspector is contained in nav stack, inspector is underneath nav bar's toolbar.  
(compact horizontal) presents as sheet, toolbar items stays in the main contents toolbar.



### 7:14 - Inspector content inside a navigation structure
```swift
struct Example1: View {
    @State private var state = AppState()

    var body: some View {
        NavigationStack {
            AnimalTable(state: $state)
                .inspector(isPresented: $state.inspectorPresented) {
                    AnimalInspectorForm(animal: $state.binding())
                }
                .toolbar {
                    Button {
                        state.inspectorPresented.toggle()
                    } label: {
                        Label("Toggle Inspector", systemImage: "info.circle")
                    }
                }
        }
    }
}
```


### 7:55 - Inspector content outside a navigation structure
```swift
struct Example2: View {
    @State private var state = AppState()
    
    var body: some View {
        NavigationStack {
            AnimalTable(state: $state)
        }
        .inspector(isPresented: $state.inspectorPresented) {
            AnimalInspectorForm(animal: $state.binding())
                .toolbar {
                    ToolbarItem(placement: .principal) {
                        HStack {
                            Button {
                            } label: {
                                Image(systemName: "rectangle.and.pencil.and.ellipsis")
                            }
                            Button {
                            } label: {
                                Image(systemName: "pawprint.circle")
                            }
                        }
                    }
                }
        }
    }
}
```

given full height of trailing content to layout.  Inspector's toolbar content belongs to inspector.  Ex, centered above the inspector.

as a sheet, toolbar content is in the sheet.

These principles extend to macOS, except inspector does not present as sheet on macOS.  So on macOS, we only worry about inside/outside navigation structure.

### 8:56 - Inspector with NavigationSplitView: detail column
```swift
NavigationSplitView {
  Sidebar()
} detail: {
  AnimalTable()
    .inspector(presented: $isPresented) {
      AnimalInspectorForm()
    }
}
```

note that here, inspector should be placed in detail columns viewbuilder.  Or it can be placed outside the navigation structure.



### 9:06 - Inspector with NavigationSplitView: Outside
```swift
NavigationSplitView {
  Sidebar()
} detail: {
  AnimalTable()
}
.inspector(presented: $isPresented) {
  AnimalInspectorForm()
}
```

# Presentation customization
After that, I'll review modifiers for presentation customizations.

SwiftUI released with iOS 16.4.

suppose we have a sheet (not inspector).  Let's try some customizations.

presentationBackground -> shows stuff behind the sheet.
presentationBackgroundInteraction -> interact with bg content
upThrough -> provide partial dimming and partial interaction.

many more customizations available.  Radius, compactAdaptation, etc.
### 9:49 - Presentation customizations
```swift
.sheet(item: $nibbledFruit) { fruit in
  FruitNibbleBulletin(fruit: fruit)
    .presentationBackground(.thinMaterial)
    .presentationDetents([.height(200), .medium, .large])
    .presentationBackgroundInteraction(.enabled(upThrough: .height(200)))
}
```

can also use on inspector (when presenting as sheet).
### 11:58 - Presentation customizations on Inspector
```swift
.inspector(presented: $state.inspectorPresented) {
  AnimalInspectorForm(animal: $state.binding())
    .presentationDetents([.height(200), .medium, .large])
    .presentationBackgroundInteraction(.enabled(upThrough: .height(200)))
}
```


# next steps
• add .inspector to your app
customize your presentations
fruit nibbler -a  cold case!


