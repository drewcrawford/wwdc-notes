#coredata #swift  #async 

# Persistence everywhere
Works across all platforms, etc.

We enhanced CoreData APIs to be as expressive as possible in Swift.  

CoreData has always cared about running code concurrently.  Persisting data requires reading/writing to an external storage medium.  Means supporting new model is a natural fit.

Last year we focused on batch operations.  This year, let's focus on currency.

## Importing USGS in earthquakes
1.  Download feed of quake data
2.  Convert to local representation
3.  Populate persistent store

## Perfomring in a managed object context
`.performAndWait` ties up execution.

 ```swift
 //BEFORE
 managedObjectContext.performAndWait {
 	//DURING
 }
 //AFTER
 ```
 
 1.  t1: BEFORE
 2.  t1: blocked
 3.  t2: DURING
 4.  t1: AFTER

We've always offered an async variant.  But this year we have a better concurrency model.

```swift
//BEFORE
await managedObjectContext.perform {
	//DURING
}
//AFTER
```

Seamlessly works with structured error handling.

## Managed object context's async support
Now `perform` decorated with async keyword.
Provided closure now lets you throw an error or return a value.

Needed to do a lot of plubming by passing completion handlers around.

## Most recent earthquakes
Number of earthquakes in the last 5 hours

Not safe to return managed objects that are already registered.  Only valid to refer to them within the closure of a call to `.perform`.

Instead, if you need to refer to a MO between execution contexts, wither make use of the objectID, or make use of the dictionar representation option.

## ScheduledTaskType
So far every async perform has been `.immediate`.  There's a second option called `.enqueued`.

For immediate:
1.  If you're on a different context, work is enqueued.  Existing queue items are ahead of you
2.  If you're on the same context, work is optimistically put at the front of the line

For enqueued:
1.  Always appends work to the end regardless of the originating callsite

| When I used to call... | Now call                   |
|------------------------|----------------------------|
| `.performAndWait`      | `await perform`            |
| `.perform`             | `await perform(.enqueued)` |          |

Further core data support for swift async
`NSManagedObjectcontext`
`NSPersistentContainer`
`NSPersistentStoreCoordinator`

Debugging concurrency in CoreData
* Address sanitizer
* Thread sanitizer
* Core Data concurrency debug


# Swift concurrency
# Improved Swift API
New APIs
* CloudKit sharing
* Core Data Spotlight integration
* Many enumeration improvements

[[Build apps that share data through CloudKit and Core Data]]
[[Showcase app data in Spotlight]]

## NSPersistentStore
NSXMLStoreType, NSBinaryStoreType, etc.  => `.xml`, `.binary` etc.

Of course, persistent stores are not the only thing that concerns itself with types.  Framework is all about storing type data.
## Attribute types
`NSAttributeDescription.AttributeType`

Basically you can introspect on CD entity types and see if their fields match what you expect

# SwiftUI
## Lazy entity resolution
Relaxes the requirements that apps have CD setup before views

We have some ivar on `App` to enforce that CD was setup before views are loaded.

This trick isn't necessary anymore for this year's SDKs.

## Dynamic configuration
* Predicates
* Sort descriptors
	* New value type!
* Bindings

```swift
private let sorts = [(
    name: "Time",
    descriptors: [SortDescriptor(\Quake.time, order: .reverse)]
), (
    name: "Time",
    descriptors: [SortDescriptor(\Quake.time, order: .forward)]
), (
    name: "Magnitude",
    descriptors: [SortDescriptor(\Quake.magnitude, order: .reverse)]
), (
    name: "Magnitude",
    descriptors: [SortDescriptor(\Quake.magnitude, order: .forward)]
)]

struct ContentView: View {
    @FetchRequest(sortDescriptors: [SortDescriptor(\Quake.time, order: .reverse)])
    private var quakes: FetchedResults<Quake>

    @State private var selectedSort = SelectedSort()

    var body: some View {
        List(quakes) { quake in
            QuakeRow(quake: quake)
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                SortMenu(selection: $selectedSort)
                .onChange(of: selectedSort) { _ in
                    let sortBy = sorts[selectedSort.index]
                    quakes.sortDescriptors = sortBy.descriptors
                }
            }
        }
    }

    struct SelectedSort: Equatable {
        var by = 0
        var order = 0
        var index: Int { by + order }
    }

    struct SortMenu: View {
        @Binding private var selectedSort: SelectedSort

        init(selection: Binding<SelectedSort>) {
            _selectedSort = selection
        }

        var body: some View {
            Menu {
                Picker("Sort By", selection: $selectedSort.by) {
                    ForEach(Array(stride(from: 0, to: sorts.count, by: 2)), id: \.self) { index in
                        Text(sorts[index].name).tag(index)
                    }
                }
                Picker("Sort Order", selection: $selectedSort.order) {
                    let sortBy = sorts[selectedSort.by + selectedSort.order]
                    let sortOrders = sortOrders(for: sortBy.name)
                    ForEach(0..<sortOrders.count, id: \.self) { index in
                        Text(sortOrders[index]).tag(index)
                    }
                }
            } label: {
                Label("More", systemImage: "ellipsis.circle")
            }
            .pickerStyle(InlinePickerStyle())
        }
        
        private func sortOrders(for name: String) -> [String] {
            switch name {
            case "Magnitude":
                return ["Highest to Lowest", "Lowest to Highest"]
            case "Time":
                return ["Newest on Top", "Oldest on Top"]
            default:
                return []
            }
        }
    }
}
```


## Sectioned fetching
New property wrapper type.
* Takes a `sectionIdentifier` keypath parameter
* Additional SectionIdentifier generic
	* `: Hashable`
* `SectionedFetchResults` contains sections, each having identifier and results

```swift
extension Quake {
    lazy var dateFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "MMMM d, yyyy"
        return formatter
    }()

    @objc var day: String {
        return dateFormatter.string(from: time)
    }
  
    @objc var magnitude_str: String {
        return "\(magnitude)"
    }
}

private let sorts = [(
    name: "Time",
    descriptors: [SortDescriptor(\Quake.time, order: .reverse)],
    section: \Quake.day
), (
    name: "Time",
    descriptors: [SortDescriptor(\Quake.time, order: .forward)],
    section: \Quake.day
), (
    name: "Magnitude",
    descriptors: [SortDescriptor(\Quake.magnitude, order: .reverse)],
    section: \Quake.magnitude_str
), (
    name: "Magnitude",
    descriptors: [SortDescriptor(\Quake.magnitude, order: .forward)],
    section: \Quake.magnitude_str
)]

struct ContentView: View {
    @SectionedFetchRequest(
        sectionIdentifier: \.day,
        sortDescriptors: [SortDescriptor(\Quake.time, order: .reverse)])
    private var quakes: SectionedFetchResults<String, Quake>

    @State private var selectedSort = SelectedSort()

    var body: some View {
        List {
            ForEach(quakes) { section in
                Section(header: Text(section.id)) {
                    ForEach(section) { quake in
                        QuakeRow(quake: quake)
                    }
                }
            }
        }
        .toolbar {
            ToolbarItem(placement: .primaryAction) {
                SortMenu(selection: $selectedSort)
                .onChange(of: selectedSort) { _ in
                    let sortBy = sorts[selectedSort.index]
                    let config = quakes
                    config.sectionIdentifier = sortBy.section
                    config.sortDescriptors = sortBy.descriptors
                }
            }
        }
    }

    struct SelectedSort: Equatable {
        var by = 0
        var order = 0
        var index: Int { by + order }
    }

    struct SortMenu: View {
        @Binding private var selectedSort: SelectedSort

        init(selection: Binding<SelectedSort>) {
            _selectedSort = selection
        }

        var body: some View {
            Menu {
                Picker("Sort By", selection: $selectedSort.by) {
                    ForEach(Array(stride(from: 0, to: sorts.count, by: 2)), id: \.self) { index in
                        Text(sorts[index].name).tag(index)
                    }
                }
                Picker("Sort Order", selection: $selectedSort.order) {
                    let sortBy = sorts[selectedSort.by + selectedSort.order]
                    let sortOrders = sortOrders(for: sortBy.name)
                    ForEach(0..<sortOrders.count, id: \.self) { index in
                        Text(sortOrders[index]).tag(index)
                    }
                }
            } label: {
                Label("More", systemImage: "ellipsis.circle")
            }
            .pickerStyle(InlinePickerStyle())
        }
        
        private func sortOrders(for name: String) -> [String] {
            switch name {
            case "Magnitude":
                return ["Highest to Lowest", "Lowest to Highest"]
            case "Time":
                return ["Newest on Top", "Oldest on Top"]
            default:
                return []
            }
        }
    }
}
```

Changes to requests are commited whenever .. is called

# Wrap up
* Persistence for Apple platforms
* Powerful new concurrency support
* New Swift API
* Dynamic SwiftUI integration
[[Simplify with SwiftUI]]
[[Meet Swift Concurrency]]

* https://developer.apple.com/documentation/coredata/loading_and_displaying_a_large_data_feed

