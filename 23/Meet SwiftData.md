Swift's new macro system to create a semaless API experience.

Relies on macros.

# Using the model macro
* Powerful new Swift macro
* define your schema with code
* Add SwiftData functionality to model types

* attributes inferred from peropeties
* support for basic value type
* struct, enum ,codable, etc.
Relationships are inferred from reference types
* other model types
* collections of model types

`@Model` modifies all stored properties
Control inference with `@Attribute` and `@Relationship`.

Exclude properties with `@Transient`.

```swift
@Model
class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var endDate: Date
    var startDate: Date
 
    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

[[Model your schema with SwiftData]]
# Working with your data

Model container
* persistence backend
* customized with configurations
* provides schema migration options

```swift
// Initialize with only a schema
let container = try ModelContainer([Trip.self, LivingAccommodation.self])

// Initialize with configurations
let container = try ModelContainer(
    for: [Trip.self, LivingAccommodation.self],
    configurations: ModelConfiguration(url: URL("path"))
)
```

```swift
import SwiftUI

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(
            for: [Trip.self, LivingAccommodation.self]
        )
    }
}
```

tracking updates
fetching models
saving changes
undoing changes

```swift
import SwiftUI

struct ContextView : View {
    @Environment(\.modelContext) private var context
}
```

## fetching
swift native types
* predicate
* fetchdescriptor
* sortdescriptor

## predicate
* fully type checked
* `#Predicate` construction instead of text parsing
* autocompleted keypaths
```swift
let today = Date()
let tripPredicate = #Predicate<Trip> { 
    $0.destination == "New York" &&
    $0.name.contains("birthday") &&
    $0.startDate > today
}
```

```swift
let descriptor = FetchDescriptor<Trip>(predicate: tripPredicate)

let trips = try context.fetch(descriptor)
```

## sortdescriptor
```swift
let descriptor = FetchDescriptor<Trip>(
    sortBy: SortDescriptor(\Trip.name),
    predicate: tripPredicate
)

let trips = try context.fetch(descriptor)
```

## modifying
insert, delete, save, change


```swift
var myTrip = Trip(name: "Birthday Trip", destination: "New York")

// Insert a new trip
context.insert(myTrip)

// Delete an existing trip
context.delete(myTrip)

// Manually save changes to the context
try context.save()
```

model macro modifies setters for change tracking and observation
updates automatically by the `ModelContext`

[[Dive deeper into SwiftData]]


# Use SwiftData with SwiftUI

* seamless integration with swiftui
* easy configuration
* automatically fetch data and update views

* leverage scene and view modifiers
* configure data store with `.modelcontainer`
* propagated throughout swiftui environment

no need for `@published`
swiftui automatically refreshes

[[Build an app with swiftData]]
```swift
import SwiftUI

struct ContentView: View  {
    @Query(sort: \.startDate, order: .reverse) var trips: [Trip]
    @Environment(\.modelContext) var modelContext
    
    var body: some View {
       NavigationStack() {
          List {
             ForEach(trips) { trip in 
                 // ...
             }
          }
       }
    }
}
```

* deifne your schema using `@Model`
* configure your modelcontainer
* leverage swiftui and swift data
[[Model your schema with SwiftData]]
[[Migrate to SwiftData]]