

Reveal the history of your model's changes with SwiftData! Use the history API to understand when data store changes occurred, and learn how to use this information to build features like remote server sync and out-of-process change handing in your app. We'll also cover how you can build support for the history API into a custom data store.

SwiftData history is a new technology that lets your app track modificatiosn to its data.  Use history to build features that need to process these changes, like syncing with your server or responding to changes from an app extension

# Fundamentals
Content changes over time.  ex when app launches, it may create some models, or insert models fetched from a remote server.  When model context is saved, all pending changes are saved into datastores.  Over time, some models may change or be deleted.  At any time, your app can query data in the store.  However, a query's results represent what's currently in the datastore.  Without history or manual diffing, there's no way to know from a query which models may have been added/deleted/updated.

Swift Data history provides an easy/efficient way to track changes to datastore over time.

* sync offline changes
* Discover out of process changes
* Reload data efficiently

SDH
* helps explore changes to a store
* Transaction format
* Transactions contain metadata

all changes that occur on a boundary such as ModelContext save.  Within a transaction, set of changes preserve the order in which each change occurred.  Each change represents a model that is inserted, updated, or deleted, and parameterized by persistent model.  Allows references to properties of the Persistent Model using keypaths.

Token acts as a bookmark for transactions and history.  Keep track of lack transaction it processed in the stream of history.  Only valid for a specific store.  
History can be deleted through the model context.  
Tokens become expired, and cannot be used for fetches.  Ops with expired token will throw `SwiftDataError.historyTokenExpired`.

When models are deleted, essential data like ids might be lost and don't provide enough info when processing history info.  SDH lets you preserve specific attributes on a model.  When deleted, they're preserved as tombstone values.

Modifier `@Attribute(.preserveValueOnDeletion)` are preserved into the tombstone.  Also parameterized by persistent model.

###  Preserve values in history on deletion - 4:57
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


# Transactions and changes

I want to have an unread indicator for trips with changes.

Using SDH, I can build feature by finding out when the data changed.

Build `HistoryDescriptor` and fetch history
Process changes
Update UI

DefaultHistoryToken.  App uses default store in SwiftData.
###  Fetch transactions from history - 6:26
```swift
private func findTransactions(after token: DefaultHistoryToken?, author: String) -> [DefaultHistoryTransaction] {
    var historyDescriptor = HistoryDescriptor<DefaultHistoryTransaction>() 
    if let token {
        historyDescriptor.predicate = #Predicate { transaction in
            (transaction.token > token) && (transaction.author == author)
        }
    }
    
    var transactions: [DefaultHistoryTransaction] = []
    let taskContext = ModelContext(modelContainer)
    do {
        transactions = try taskContext.fetchHistory(historyDescriptor)
    } catch let error {
        print(error)
    }

    return transactions
}
```

Configure constraints for my request.  Build a predicate that constrains transaction to have occurred after provided token.  Fn only surfaces changes from widget, so add constraint to fetch changes authored by a specified author.

Calling without a token, just fetch all available history.




###  Process history changes - 7:34
```swift
private func findTrips(in transactions: [DefaultHistoryTransaction]) -> (Set<Trip>, DefaultHistoryToken?) {
        let taskContext = ModelContext(modelContainer)
        var resultTrips: Set<Trip> = []
        for transaction in transactions {
            for change in transaction.changes {
                let modelID = change.changedPersistentIdentifier
                let fetchDescriptor = FetchDescriptor<Trip>(predicate: #Predicate { trip in
                    trip.livingAccommodation?.persistentModelID == modelID
                })
                let fetchResults = try? taskContext.fetch(fetchDescriptor)
                guard let matchedTrip = fetchResults?.first else {
                    continue
                }
                switch change {
                case .insert(_ as DefaultHistoryInsert<LivingAccommodation>):
                    resultTrips.insert(matchedTrip)
                case .update(_ as DefaultHistoryUpdate<LivingAccommodation>):
                    resultTrips.update(with: matchedTrip)
                case .delete(_ as DefaultHistoryDelete<LivingAccommodation>):
                    resultTrips.remove(matchedTrip)
                default: break
                }
            }
        }
        return (resultTrips, transactions.last?.token)
    }
```

for each transaction and change, history provides persistent model ID.  To get modle isntance for a trip, build a fetch descriptor using that persistent model ID.  Then, fetch from model context.

Type is `DefaultHistoryInsert` because I'm using the default store for this model.

To handle this, I add a case for `DefaultHistoryDelete` and remove from my set.

Return token so that future calls can start from here.

Define a third function that user UserDefaults to store the most recent token.  In my function findUnreadTrips, I'll fetch the token if available, and decode from JSON, before calling findTransactions function with that token.

Specify author: widget.  


###  Save and use a history token - 10:19
```swift
private func findUnreadTrips() -> Set<Trip> {
    let tokenData = UserDefaults.standard.data(forKey: UserDefaultsKey.historyToken)
    
    var historyToken: DefaultHistoryToken? = nil
    if let tokenData {
        historyToken = try? JSONDecoder().decode(DefaultHistoryToken.self, from: tokenData)
    }
    let transactions = findTransactions(after: historyToken, author: TransactionAuthor.widget)
    let (unreadTrips, newToken) = findTrips(in: transactions)
    
    if let newToken {
        let newTokenData = try? JSONEncoder().encode(newToken)
        UserDefaults.standard.set(newTokenData, forKey: UserDefaultsKey.historyToken)
    }
    return unreadTrips
}
```



###  Update the user interface - 11:30
```swift
struct ContentView: View {
    @Environment(\.scenePhase) private var scenePhase
    @State private var showAddTrip = false
    @State private var selection: Trip?
    @State private var searchText: String = ""
    @State private var tripCount = 0
    @State private var unreadTripIdentifiers: [PersistentIdentifier] = []

    var body: some View {
        NavigationSplitView {
            TripListView(selection: $selection, tripCount: $tripCount,
                         unreadTripIdentifiers: $unreadTripIdentifiers,
                         searchText: searchText)
            .toolbar {
                ToolbarItem(placement: .topBarLeading) {
                    EditButton()
                        .disabled(tripCount == 0)
                }
                ToolbarItemGroup(placement: .topBarTrailing) {
                    Spacer()
                    Button {
                        showAddTrip = true
                    } label: {
                        Label("Add trip", systemImage: "plus")
                    }
                }
            }
        } detail: {
            if let selection = selection {
                NavigationStack {
                    TripDetailView(trip: selection)
                }
            }
        }
        .task {
            unreadTripIdentifiers = await DataModel.shared.unreadTripIdentifiersInUserDefaults
        }
        .searchable(text: $searchText, placement: .sidebar)
        .sheet(isPresented: $showAddTrip) {
            NavigationStack {
                AddTripView()
            }
            .presentationDetents([.medium, .large])
        }
        .onChange(of: selection) { _, newValue in
            if let newSelection = newValue {
                if let index = unreadTripIdentifiers.firstIndex(where: {
                    $0 == newSelection.persistentModelID
                }) {
                    unreadTripIdentifiers.remove(at: index)
                }
            }
        }
        .onChange(of: scenePhase) { _, newValue in
            Task {
                if newValue == .active {
                    unreadTripIdentifiers += await DataModel.shared.findUnreadTripIdentifiers()
                } else {
                    // Persist the unread trip names for the next launch session.
                    await DataModel.shared.setUnreadTripIdentifiersInUserDefaults(unreadTripIdentifiers)
                }
            }
        }
        #if os(macOS)
        .onReceive(NotificationCenter.default.publisher(for: NSApplication.didBecomeActiveNotification)) { _ in
            Task {
                unreadTripIdentifiers += await DataModel.shared.findUnreadTripIdentifiers()
            }
        }
        .onReceive(NotificationCenter.default.publisher(for: NSApplication.willTerminateNotification)) { _ in
            Task {
                await DataModel.shared.setUnreadTripIdentifiersInUserDefaults(unreadTripIdentifiers)
            }
        }
        #endif
    }
}
```


# Custom stores

If you're building custom datastores, custom store can also support history, if your underlying model supports it.

Implement your own types to represent the fundamental elements of the swift data history API.  
* HistoryTransaction
* HistoryChange
* HistoryToken
* conform to HistoryProviding

Transaction boundaries groups a set of changes

Write ops will need to be coalesced and ordered.  In default store, all changes to models are grouped as a single transaction. 

Transactions need unique identifiers.
Custom change represents a granular modification.  In default store, it's a model
Use unique identifiers
Consider change types that you need (all?  different?).  Ex if you only insert, you only need insert.
Preserve values on deletion?  

* Identify rows
* Build sets of transactions and changes
* Consider when to delete history
If you remove models from your app, there may be history data about those models that you never use going forward.  Consider deleting history from the datastore.

Custom token:
Unique identifiers
Multiple stores

# Next steps
* create experiences with history
* Migrate from Core Data
* Support history in custom stores







# Resources
* [Fetching and filtering time-based model changes](https://developer.apple.com/documentation/SwiftData/Fetching-and-filtering-time-based-model-changes)
* [Forum: Programming Languages](https://developer.apple.com/forums/topics/programming-languages-topic?cid=vf-a-0010)
* [SwiftData](https://developer.apple.com/documentation/SwiftData)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10075/4/0F3D64B6-B594-42E8-8B59-2088D1B251F8/downloads/wwdc2024-10075_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10075/4/0F3D64B6-B594-42E8-8B59-2088D1B251F8/downloads/wwdc2024-10075_sd.mp4?dl=1)