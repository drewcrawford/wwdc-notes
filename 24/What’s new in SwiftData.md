SwiftData makes it easy to add persistence to your app with its expressive, declarative API. Learn about refinements to SwiftData, including compound uniqueness constraints, faster queries with #Index, queries in Xcode previews, and rich predicate expressions. Join us to explore how you can use all of these features to express richer models and improve performance in your app. To discover how to build a custom data store or use the history API in SwiftData, watch “Create a custom data store with SwiftData” and “Track model changes with SwiftData history”.

### SampleTrips models decorated with @Model - 1:32
```swift
// Trip Models decorated with @Model
import Foundation
import SwiftData

@Model
class Trip {
    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date
    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}

@Model
class BucketListItem {...}

@Model
class LivingAccommodation {...}
```

### SampleTrips using modelContainer scene modifier - 1:43
```swift
// Trip App using modelContainer Scene modifier
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView
        }
        .modelContainer(for: Trip.self)
    }
}
```

### SampleTrips using @Query - 1:53
```swift
// Trip App using @Query
import SwiftUI
import SwiftData

struct ContentView: View {
    @Query var trips: [Trip]

    var body: some View {
        NavigationSplitView {
            List(selection: $selection) {
                ForEach(trips) { trip in
                    TripListItem(trip: trip)
                }
            }
        }
    }
}
```

### SampleTrips models decorated with @Model - 2:16
```swift
// Trip Models decorated with @Model
import Foundation
import SwiftData

@Model
class Trip {
    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date
    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}

@Model
class BucketListItem {...}

@Model
class LivingAccommodation {...}
```

### Add unique constraints to avoid duplication - 3:08
```swift
// Add unique constraints to avoid duplication
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])
    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date
    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}
```

### Add .preserveValueOnDeletion to capture unique columns - 3:36
```swift
// Add .preserveValueOnDeletion to capture unique columns
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])
    @Attribute(.preserveValueOnDeletion)
    var name: String
    var destination: String
    @Attribute(.preserveValueOnDeletion)
    var startDate: Date
    @Attribute(.preserveValueOnDeletion)
    var endDate: Date
    var bucketList: [BucketListItem] = [BucketListItem]()
    var livingAccommodation: LivingAccommodation?
}
```

### SampleTrips using modelContainer scene modifier - 4:35
```swift
// Trip App using modelContainer Scene modifier
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self)
    }
}
```

### Customize a model container in the app - 4:52
```swift
// Customize a model container in the app
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, inMemory: true, isAutosaveEnabled: true, isUndoEnabled: true)
    }
}
```

### Add a model container to the app - 5:13
```swift
// Add a model container to the app
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var container: ModelContainer = {
        do {
            let configuration = ModelConfiguration(schema: Schema([Trip.self]), url: fileURL)
            return try ModelContainer(for: Trip.self, configurations: configuration)
        } catch {
            ...
        }
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

### Use your own custom data store - 5:59
```swift
// Use your own custom data store
import SwiftUI
import SwiftData

@main
struct TripsApp: App {
    var container: ModelContainer = {
        do {
            let configuration = JSONStoreConfiguration(schema: Schema([Trip.self]), url: fileURL)
            return try ModelContainer(for: Trip.self, configurations: configuration)
        } catch {
            ...
        }
    }()

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

### Make preview data using traits - 6:58
```swift
// Make preview data using traits
struct SampleData: PreviewModifier {
    static func makeSharedContext() throws -> ModelContainer {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try ModelContainer(for: Trip.self, configurations: config)
        Trip.makeSampleTrips(in: container)
        return container
    }

    func body(content: Content, context: ModelContainer) -> some View {
        content.modelContainer(context)
    }
}

extension PreviewTrait where T == Preview.ViewTraits {
    @MainActor
    static var sampleData: Self = .modifier(SampleData())
}
```

### Use sample data in a preview - 8:15
```swift
// Use sample data in a preview
import SwiftUI
import SwiftData

struct ContentView: View {
    @Query var trips: [Trip]

    var body: some View {
        ...
    }
}

#Preview(traits: .sampleData) {
    ContentView()
}
```

### Create a preview query using @Previewable - 8:50
```swift
// Insert code snippet.
```

### Create a predicate to find a Trip based on search text - 9:55
```swift
// Create a Predicate to find a Trip based on Search Text
let predicate = #Predicate<Trip> { searchText.isEmpty ? true : $0.name.localizedStandardContains(searchText) }
```

### Create a Compound Predicate to find a Trip based on Search Text - 10:06
```swift
// Create a Compound Predicate to find a Trip based on Search Text
let predicate = #Predicate<Trip> { searchText.isEmpty ? true : $0.name.localizedStandardContains(searchText) || $0.destination.localizedStandardContains(searchText) }
```

### Build a predicate to find Trips with BucketListItems that are not in the plan - 10:46
```swift
// Build a predicate to find Trips with BucketListItems that are not in the plan
let unplannedItemsExpression = #Expression<[BucketListItem], Int> { items in
    items.filter { !$0.isInPlan }.count
}

let today = Date.now

let tripsWithUnplannedItems = #Predicate<Trip> { trip
    // The current date falls within the trip
    (trip.startDate ..< trip.endDate).contains(today) &&
    // The trip has at least one BucketListItem where 'isInPlan' is false
    unplannedItemsExpression.evaluate(trip.bucketList) > 0
}
```

### Add Index for commonly used KeyPaths or combination of KeyPaths - 12:41
```swift
// Add Index for commonly used KeyPaths or combination of KeyPaths
import SwiftData

@Model
class Trip {
    #Unique<Trip>([\.name, \.startDate, \.endDate])
    #Index<Trip>([\.name], [\.startDate], [\.endDate], [\.name, \.startDate, \.endDate])
    var name: String
    var destination: String
    var startDate: Date
    var endDate: Date
    var bucketList: [BucketListItem] = [BucketListItem]
    var livingAccommodation: LivingAccommodation
}
```


# REsources
* https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app
* https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app
