#swiftdata 
Learn how you can harness the power of SwiftData in your app. Find out how ModelContext and ModelContainer work together to persist your app's data. We'll show you how to track and make your changes manually and use SwiftData at scale with FetchDescriptor, SortDescriptor, and enumerate. To get the most out of this session, we recommend first watching "Meet SwiftData" and "Model your schema with SwiftData" from WWDC23.

This builds on 

[[Meet SwiftData]]
[[Model your schema with SwiftData]]

Model
* types you already use
* `@Model` macro
* Inferred or explicit structure
* Deep customization

Model rules:
* describe the schema
* instances used in code

this duality of playing both parts, makes classes annotated with model the central point of contact in applications that use swift data.  Aligned API concept to support each of these roles.

1.  Schema is applied to a class called the ModelContainer, to describe how data should be persisted.  

ModelContainer is the bridge between schema and persistence
how objects are stored
evolution of modesl
* versioning
* migration
* graph seprataion.


### Trip model with cascading relationships - 1:45
```swift
@Model
final class Trip {
    var destination: String?
    var end_date: Date?
    var name: String?
    var start_date: Date?
  
    @Relationship(.cascade)
    var bucketListItem: [BucketListItem] = [BucketListItem]()
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?
}
```


# Configuring persistence
### Initializing a ModelContainer - 4:21
```swift
// ModelContainer initialized with just Trip
let container = try ModelContainer(for: Trip.self)

// SwiftData infers related model classes as well
let container = try ModelContainer(
    for: [
        Trip.self, 
        BucketListItem.self, 
        LivingAccommodation.self
    ]
)
```

ModelContainer actually infers a schema with the related types, even though I passed it only a single type.

## ModelConfiguration
* describes persistence of a schema
* In memory or on disk
* File location
* Read only
* CloudKid container identifier
### Using ModelConfiguration to customize ModelContainer - 5:41
```swift
let fullSchema = Schema([
    Trip.self,
    BucketListItem.self,
    LivingAccommodations.self,
    Person.self,
    Address.self
])

let trips = ModelConfiguration(
    schema: Schema([
        Trip.self,
        BucketListItem.self,
        LivingAccommodations.self
    ]),
    url: URL(filePath: "/path/to/trip.store"),
    cloudKitContainerIdentifier: "com.example.trips"
)

let people = ModelConfiguration(
    schema: Schema([Person.self, Address.self]),
    url: URL(filePath: "/path/to/people.store"),
    cloudKitContainerIdentifier: "com.example.people"
) 

let container = try ModelContainer(for: fullSchema, trips, people)
```

We can keep this data separate from the trips graph.

### Creating ModelContainer in SwiftUI - 6:49
```swift
@main
struct TripsApp: App {
    let fullSchema = Schema([
        Trip.self, 
        BucketListItem.self,
        LivingAccommodations.self,
        Person.self, 
        Address.self
    ])
  
    let trips = ModelConfiguration(
        schema: Schema([
            Trip.self,
            BucketListItem.self,
            LivingAccommodations.self
        ]),
        url: URL(filePath: "/path/to/trip.store"),
        cloudKitContainerIdentifier: "com.example.trips"
    )
  
    let people = ModelConfiguration(
        schema: Schema([
            Person.self, 
            Address.self
        ]),
        url: URL(filePath: "/path/to/people.store"),
        cloudKitContainerIdentifier: "com.example.people"
    )
  
    let container = try ModelContainer(for: fullSchema, trips, people)
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```


## configuring persistence
* `ModelContainer` combines `Schema` with persistence
* grows with an application



# Track and persist changes
### Using the modelContainer modifier - 7:40
```swift
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

We bind the modelContext in the environment.  the main context is a special actor-aligned model context.  By using this from the environment, view code has easy access to the context used by the query here.

Each trip pobject is fetched into the main context.  If the trip is edited, it's recorded by the context as a snapshot.  As other changes are made... the context tracks and maintains satates about these changes until you call context.save().

Even though the deleted trip is no longer visible in the list, it still exists in the model context, until save is called.

Once save is called, persisted in the model container, and clears its state.

* tracks objects in use
* propagates changes to `ModelContainer`
* Clear changes with rollback or reset
* Undo/redo support
* autosave etc

In SwiftUI 
### Referencing a ModelContext in SwiftUI views - 7:50
```swift
struct ContentView: View {
    @Query var trips: [Trip]
    @Environment(\.modelContext) var modelContext
  
    var body: some View {
        NavigationStack (path: $path) {
            List(selection: $selection) {
                ForEach(trips) { trip in
                    TripListItem(trip: trip)
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                modelContext.delete(trip)
                            } label: {
                                Label("Delete", systemImage: "trash")
                            }
                        }
                }
                .onDelete(perform: deleteTrips(at:))
            }
        }
    }
}
```

### Enabling undo on a ModelContainer - 9:57
```swift
@main
struct TripsApp: App {
    @Environment(\.undoManager) var undoManager
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isUndoEnabled: true)
    }
}
```

as changes are made in the main context, system gestures like 3-finger swipe and shape can be used to undo or redo changes with no additional code.

* automatically registers actions as undo/redo
* `modelContainer()` the environment's `undoManager` usually proivded as part of window/group
* supports standard system gestures

autosave.  the model context will save in response to system events, such as fg/bg.
main context will also periodically save as an application is used.

### Enabling autosave on a ModelContainer - 11:05
```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isAutosaveEnabled: false)
    }
}
```



# Modeling at scale

In [[Meet SwiftData]] we learned about how to work with model context. But UI is not the only place where we work with model objects.  I'll examine how swiftdata makes writing powerful, scalable code easier and safer than ever.

* background operatons
* sync
* batch processing


### Fetching objects with FetchDescriptor - 11:54
```swift
let context = self.newSwiftContext(from: Trip.self)
var trips = try context.fetch(FetchDescriptor<Trip>())
```

ex, what are all the trips that involve staying in a specific hotel?
### Fetching objects with #Predicate and FetchDescriptor - 12:14
```swift
let context = self.newSwiftContext(from: Trip.self)
let hotelNames = ["First", "Second", "Third"]

var predicate = #Predicate<Trip> { trip in
    trip.livingAccommodations.filter {
        hotelNames.contains($0.placeName)
    }.count > 0
}

var descriptor = FetchDescriptor(predicate: predicate)
var trips = try context.fetch(descriptor)
```

complicated queries that support subqueries and joins can be written in pure swift.

### Fetching objects with #Predicate and FetchDescriptor - 12:27
```swift
let context = self.newSwiftContext(from: Trip.self)

predicate = #Predicate<Trip> { trip in
    trip.livingAccommodations.filter {
        $0.hasReservation == false
    }.count > 0
}

descriptor = FetchDescriptor(predicate: predicate)
var trips = try context.fetch(descriptor)
```

### Enumerating objects with FetchDescriptor - 13:18
```swift
context.enumerate(FetchDescriptor<Trip>()) { trip in
    // Operate on trip
}
```
* compiler validated queries
* typed by model
* additional parameters
	* offset/limit
	* faulting/prefetching


### Enumerating with FetchDescriptor and SortDescriptor - 13:36
```swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

context.enumerate(descriptor) { trip in
    // Remind me to make reservations for trip
}
```

reduce I/O operations during the traversal.  Enumerate also includes mutation guards by default.  Frequent causes of performance issues with large traversals.


### Fine tuning enumerate with batchSize - 14:01
```swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

context.enumerate(
    descriptor,
    batchSize: 10000
) { trip in
    // Remind me to make reservations for trip
}
```

### Fine tuning enumerate with batchSize and allowEscapingMutations - 14:28

tells enumerate that we intend to do mutations.  This is a performance issue.  This prevents it from freeing objects that were already traversed.
```swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

context.enumerate(
    descriptor,
    batchSize: 500,
    allowEscapingMutations: true
) { trip in
    // Remind me to make reservations for trip
}
```

# Wrap up
* adopt `schema` and `ModelConfiguration`
* adopt undo/redo
* adopt `FetchDescriptor` and enumerate
# Resources
* https://developer.apple.com/documentation/coredata/adopting_swiftdata_for_a_core_data_app
* https://developer.apple.com/documentation/SwiftData
