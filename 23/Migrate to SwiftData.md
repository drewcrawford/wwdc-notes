Discover how you can start using SwiftData in your apps. We'll show you how to use Xcode to generate model classes from your existing Core Data object models, use SwiftData alongside your previous implementation, or even completely replace your existing solution. Before watching this session, make sure you check out "Meet SwiftData."
#swiftdata 

Whether you are ready for a complete transition,... etc.

# Generate model classes
Core data model to swift data model?
Editor -> Create SwifData code.

Also an option in the case of creating a new swift app.  
Swift types should conform to Model.  Each one captures information for entities/variables.  Etc.


# Complete adoption
When fully migrating your app, you're replacing the CD stack with a SD stack.

## Considerations
* Core Data model designs must be supported in SwiftData.
* Corresponding SwiftData model type for each CD entity

## Set up persistent stack
* generate model classes
* Set up ModelContainer

ensures that all windows in the gruop are configured to act as the same persistent container.  I'm setting up both my container and the content.  modelContainer also creates/sets a default model context in the environment.

### Creating a ModelContainer in SwiftUI - 4:37
```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(
            for: [Trip.self, BucketListItem.self, LivingAccommodation.self]
        )
    }
}
```

### Object creation in Core Data - 4:57
```swift
@Environment(\.managedObjectContext) private var viewContext

let newTrip = Trip(context: viewContext)
newTrip.name = name
newTrip.destination = destination
newTrip.startDate = startDate
newTrip.endDate = endDate
```


how object creation works?

### Object creation in SwiftData - 5:30
```swift
@Environment(\.modelContext) private var modelContext

let trip = Trip(
    name: name, 
    destination: destination, 
    startDate: startDate, 
    endDate: endDate
)

modelContext.insert(object: trip)
```

In SwiftData, we create a new instance of trip.  Already comparatively more legible than CD.

Inserts into model context to ensure that it is persisted.

We implicitly set saves on UI gestures, and maybe a timer?

### Fetch with Query in SwiftData - 6:16
```swift
@Query(sort: \.startDate, order: .forward)

var trips: [Trip]
```



# Coexists with Core Data

A full migration may not be feasible or practical.  Maybe you have two different stacks?  Different stacks that talk to the same store.

Ensure both stacks are writing to the same URL.
Turn on persistent history tracking.  Not enabled on CD by default.

### Setting store path and enabling persistent history tracking in Core Data - 7:30
```swift
let url = URL(fileURLWithPath: "/path/to/Trips.store")

if let description = container.persistentStoreDescriptions.first {
    description.url = url
    description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
}
```

* backwards compatibility
* Future development with SwiftData

considerations:
* namespace, don't call classes the same thing.
	* class name only.  entity name can be distinct


### Ensuring Core Data and SwiftData class names are unique - 9:11
```swift
class CDTrip: NSManagedObject {
    // ...
}


@Model final class Trip {
    // ...
}
```

* keep schemas in sync
	* mismatched hashes can trigger a migration
* schema versioning

[[Model your schema with SwiftData]]

further considerations for UIKit/AppKit
* Bind to CD/coexistence
* Treat as Swift classes

# Wrap up
* flexibly migrate to SwiftData
* choose the best option for your use case
[[Meet SwiftData]]
[[Model your schema with SwiftData]]


# Resources
https://developer.apple.com/documentation/SwiftData