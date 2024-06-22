Experience Swift 6 migration in action as we update an existing sample app. Learn how to migrate incrementally, module by module, and how the compiler helps you identify code that's at risk of data races. Discover different techniques for ensuring clear isolation boundaries and eliminating concurrent access to shared mutable state.

By adopting swift concurrency we went from an ad-hoc concurrency architecture to something actually organized.

Reference types allow shared mutable state to be passed around.

Useful if: You're experiencing hard-to-reproduce crashes.
You're integrating more concurrency into your app
You're maintaining a library that is used from concurrent code

swiftpackageindex.com

1.  Enable complete concurrency checking
2. Enable Swift 6
3. Next target
4. Audit unsafe opt-outs

Resist the temptation to blend both refactoring and enabling data race safety.


###  9:08 - Recaffeinater and CaffeineThresholdDelegate
```swift
//Define Recaffeinator class
class Recaffeinater: ObservableObject {
    @Published var recaffeinate: Bool = false
    var minimumCaffeine: Double = 0.0
}

//Add protocol to notify if caffeine level is dangerously low
extension Recaffeinater: CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

###  9:26 - Add @MainActor to isolate the Recaffeinator
```swift
//Isolate the Recaffeinater class to the main actor
@MainActor class Recaffeinater: ObservableObject {
    @Published var recaffeinate: Bool = false
    var minimumCaffeine: Double = 0.0
}
```

###  9:38 - Warning in the protocol implementation
```swift
//warning: Main actor-isolated instance method 'caffeineLevel(at:)' cannot be used to satisfy nonisolated protocol requirement
public func caffeineLevel(at level: Double) {
    if level < minimumCaffeine {
        // TODO: alert user to drink more coffee!
    }
}
```

###  9:59 - Understanding why the warning is there
```swift
//This class is guaranteed on the main actor...
@MainActor class Recaffeinater: ObservableObject {
    @Published var recaffeinate: Bool = false
    var minimumCaffeine: Double = 0.0
}

//...but this protocol is not
extension Recaffeinater: CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

###  12:59 - A warning on the logger variable
```swift
//var 'logger' is not concurrency-safe because it is non-isolated global shared mutable state; this is an error in the Swift 6 language mode
var logger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.ContentView",
    category: "Root View"
)
```

###  13:38 - Option 1: Convert 'logger' to a 'let' constant
```swift
//Option 1: Convert 'logger' to a 'let' constant to make 'Sendable' shared state immutable
let logger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.ContentView",
    category: "Root View"
)
```

###  14:20 - Option 2: Isolate 'logger' it to the main actor
```swift
//Option 2: Annotate 'logger' with '@MainActor' if property should only be accessed from the main actor
@MainActor var logger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.ContentView",
    category: "Root View"
)
```

###  14:58 - Option 3: Mark it nonisolated(unsafe)
```swift
//Option 3: Disable concurrency-safety checks if accesses are protected by an external synchronization mechanism
nonisolated(unsafe) var logger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.ContentView",
    category: "Root View"
)
```

###  15:43 - The right answer
```swift
//Option 1: Convert 'logger' to a 'let' constant to make 'Sendable' shared state immutable
let logger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.ContentView",
    category: "Root View"
)
```

When does the initializer get run?  They're initialized lazily.  The value is initialized on first use.  Important difference vs C/ObjC.  They're initialized on startup, which is bad for launch time.  Swift's lazy initialization avoids these problems.  But this can introduce data races.  What if two threads try to log at the same time?

In Swift, global variables are guaranteed to be created atomically.  Only one will initialize while the other will block.


###  17:03 - scheduleBackgroundRefreshTasks() has two warnings
```swift
func scheduleBackgroundRefreshTasks() {
    scheduleLogger.debug("Scheduling a background task.")

    // Get the shared extension object.
    let watchExtension = WKApplication.shared() //warning: Call to main actor-isolated class method 'shared()' in a synchronous nonisolated context

    // If there is a complication on the watch face, the app should get at least four
    // updates an hour. So calculate a target date 15 minutes in the future.
    let targetDate = Date().addingTimeInterval(15.0 * 60.0)

    // Schedule the background refresh task.
    watchExtension.scheduleBackgroundRefresh(withPreferredDate: targetDate, userInfo: nil) {
        //warning: Call to main actor-isolated instance method 'scheduleBackgroundRefresh(withPreferredDate:userInfo:scheduledCompletion:)' in a synchronous nonisolated context error in

        // Check for errors.
        if let error {
            scheduleLogger.error(
                "An error occurred while scheduling a background refresh task: \(error.localizedDescription)"
            )
            return
        }

        scheduleLogger.debug("Task scheduled!")
    }
}
```

###  17:57 - Annotate function with @MainActor
```swift
@MainActor func scheduleBackgroundRefreshTasks() {
    scheduleLogger.debug("Scheduling a background task.")

    // Get the shared extension object.
    let watchExtension = WKApplication.shared()

    // If there is a complication on the watch face, the app should get at least four
    // updates an hour. So calculate a target date 15 minutes in the future.
    let targetDate = Date().addingTimeInterval(15.0 * 60.0)

    // Schedule the background refresh task.
    watchExtension.scheduleBackgroundRefresh(withPreferredDate: targetDate, userInfo: nil) { error in

        // Check for errors.
        if let error {
            scheduleLogger.error(
                "An error occurred while scheduling a background refresh task: \(error.localizedDescription)"
            )
            return
        }

        scheduleLogger.debug("Task scheduled!")
    }
}
```

Some callbacks have a guarantee.  They might say in their documentation.  Some delegates make no guarantee.  Puts a lot of burden on you the user to do the right thing.  Or can change over time.


###  22:15 - Revisiting the Recaffeinater
```swift
//This class is guaranteed on the main actor...
@MainActor class Recaffeinater: ObservableObject {
    @Published var recaffeinate: Bool = false
    var minimumCaffeine: Double = 0.0
}

//...but this protocol is not
//warning: Main actor-isolated instance method 'caffeineLevel(at:)' cannot be used to satisfy nonisolated protocol requirement
extension Recaffeinater: CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```



###  22:26 - Option 1: Mark function as nonisolated
```swift
//error: Main actor-isolated property 'minimumCaffeine' can not be referenced from a non-isolated context
nonisolated public func caffeineLevel(at level: Double) {
    if level < minimumCaffeine {
        // TODO: alert user to drink more coffee!
    }
}
```

###  23:07 - Option 1b: Wrap functionality in a Task
```swift
nonisolated public func caffeineLevel(at level: Double) {
    Task {
        @MainActor in
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```



###  23:34 - Option 1c: Explore options to update the protocol
```swift
public protocol CaffeineThresholdDelegate: AnyObject {
    func caffeineLevel(at level: Double)
}
```

###  24:15 - Option 1d: Instead of wrapping it in a Task, use `MainActor.assumeisolated`
```swift
nonisolated public func caffeineLevel(at level: Double) {
    MainActor.assumeIsolated {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

assumeIsolated asserts, and traps that we're on the main actor.  Better than a race condition that could corrupt the user's data.



###  25:21 - `@preconcurrency` as a shorthand for assumeIsolated
```swift
extension Recaffeinater: @preconcurrency CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

Assumes it's being called on the actor the view is isolated to, and will trap if it isn't.


###  26:42 - Add `@MainActor` to the delegate protocol in CoffeeKit
```swift
@MainActor public protocol CaffeineThresholdDelegate: AnyObject {
    func caffeineLevel(at level: Double)
}
```

Warns that preconcurrency is no longer needed.
###  26:50 - A new warning
```swift
//warning: @preconcurrency attribute on conformance to 'CaffeineThresholdDelegate' has no effect
extension Recaffeinater: @preconcurrency CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

###  27:09 - Remove @preconcurrency
```swift
extension Recaffeinater: CaffeineThresholdDelegate {
    public func caffeineLevel(at level: Double) {
        if level < minimumCaffeine {
            // TODO: alert user to drink more coffee!
        }
    }
}
```

**Don't panic!!**

1.  Resolve simple issues first
2. Look for root causes to large numbers of issues
3. Try with latest SDK

Take your time!


###  29:56 - Global variables in CoffeeKit are marked as `var`
```swift
//warning: Var 'hkLogger' is not concurrency-safe because it is non-isolated global shared mutable state
private var hkLogger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.HealthKitController",
    category: "HealthKit"
)

// The key used to save and load anchor objects from user defaults.
//warning: Var 'anchorKey' is not concurrency-safe because it is non-isolated global shared mutable state
private var anchorKey = "anchorKey"

// The HealthKit store.
// warning: Var 'store' is not concurrency-safe because it is non-isolated global shared mutable state
private var store = HKHealthStore()

// warning: Var 'isAvailable' is not concurrency-safe because it is non-isolated global shared mutable state
private var isAvailable = HKHealthStore.isHealthDataAvailable()

// Caffeine types, used to read and write caffeine samples.
// warning: Var 'caffeineType' is not concurrency-safe because it is non-isolated global shared mutable state
private var caffeineType = HKObjectType.quantityType(forIdentifier: .dietaryCaffeine)!

// warning: Var 'types' is not concurrency-safe because it is non-isolated global shared mutable state
private var types: Set<HKSampleType> = [caffeineType]

// Milligram units.
// warning: Var 'miligrams' is not concurrency-safe because it is non-isolated global shared mutable state
internal var miligrams = HKUnit.gramUnit(with: .milli)
```

###  30:19 - Change all global variables to `let`
```swift
private let hkLogger = Logger(
    subsystem: "com.example.apple-samplecode.Coffee-Tracker.watchkitapp.watchkitextension.HealthKitController",
    category: "HealthKit"
)

// The key used to save and load anchor objects from user defaults.
private let anchorKey = "anchorKey"

// The HealthKit store.
private let store = HKHealthStore()

private let isAvailable = HKHealthStore.isHealthDataAvailable()

// Caffeine types, used to read and write caffeine samples.
private let caffeineType = HKObjectType.quantityType(forIdentifier: .dietaryCaffeine)!

private let types: Set<HKSampleType> = [caffeineType]

// Milligram units.
internal let miligrams = HKUnit.gramUnit(with: .milli)
```

Lots of easy wins, a few hard issues to tackle.


###  30:38 - Warning 1: Sending arrays in `drinksUpdated()`
```swift
// warning: Sending 'self.currentDrinks' risks causing data races
// Sending main actor-isolated 'self.currentDrinks' to actor-isolated instance method 'save' risks causing data races between actor-isolated and main actor-isolated uses
await store.save(currentDrinks)
```

###  32:04 - Looking at Drink struct
```swift
// The record of a single drink.
public struct Drink: Hashable, Codable {
    // The amount of caffeine in the drink.
    public let mgCaffeine: Double
    
    // The date when the drink was consumed.
    public let date: Date
    
    // A globally unique identifier for the drink.
    public let uuid: UUID
    
    public let type: DrinkType?
    
    public var latitude, longitude: Double?
    
    // The drink initializer.
    public init(type: DrinkType, onDate date: Date, uuid: UUID = UUID()) {
        self.mgCaffeine = type.mgCaffeinePerServing
        self.date = date
        self.uuid = uuid
        self.type = type
    }
    
    internal init(from sample: HKQuantitySample) {
        self.mgCaffeine = sample.quantity.doubleValue(for: miligrams)
        self.date = sample.startDate
        self.uuid = sample.uuid
        self.type = nil
    }
    
    // Calculate the amount of caffeine remaining at the provided time,
    // based on a 5-hour half life.
    public func caffeineRemaining(at targetDate: Date) -> Double {
        // Calculate the number of half-life time periods (5-hour increments)
        let intervals = targetDate.timeIntervalSince(date) / (60.0 * 60.0 * 5.0)
        return mgCaffeine * pow(0.5, intervals)
    }
}
```

Swift doesn't infer sendable for public types.  Must explicitly add conformances for public types.


###  33:29 - Mark `Drink` struct as Sendable
```swift
// The record of a single drink.
public struct Drink: Hashable, Codable, Sendable {
    //...
}
```

###  33:35 - Another type that isn't Sendable
```swift
// warning: Stored property 'type' of 'Sendable'-conforming struct 'Drink' has non-sendable type 'DrinkType?'
public let type: DrinkType?
```

could use nonisolated unsafe for this.
###  34:28 - Using nonisolated(unsafe)
```swift
nonisolated(unsafe) public let type: DrinkType?
```

###  34:45 - Undo that change
```swift
public let type: DrinkType?
```

###  35:04 - Change DrinkType to be Sendable
```swift
// Define the types of drinks supported by Coffee Tracker.
public enum DrinkType: Int, CaseIterable, Identifiable, Codable, Sendable {
    //...
}
```


###  36:35 - CoreLocation using AsyncSequence
```swift
//Create a new drink to add to the array.
var drink = Drink(type: type, onDate: date)

do {
    //error: 'CLLocationUpdate' is only available in watchOS 10.0 or newer for
    try await update in CLLocationUpdate.liveUpdates() {
        guard let coord = update.location else {
            logger.info("Update received but no location, \(update.location)")
            break
        }
        
        drink.latitude = coord.coordinate.latitude
        drink.longitude = coord.coordinate.longitude
    }
} catch {
    //...
}
```

we need the older APIs because we need to support older deployment target.


###  38:10 - Create a CoffeeLocationDelegate
```swift
class CoffeeLocationDelegate: NSObject, CLLocationManagerDelegate {
    
    var location: CLLocation?
    var manager: CLLocationManager!
    
    var latitude: CLLocationDegrees? {
        location?.coordinate.latitude
    }
    
    var longitude: CLLocationDegrees? {
        location?.coordinate.longitude
    }
    
    override init () {
        super.init()
        manager = CLLocationManager()
        manager.delegate = self
        manager.startUpdatingLocation()
    }
    
    func locationManager(
        _ manager: CLLocationManager,
        didUpdateLocations locations: [CLLocation]
    ) {
        self.location = locations.last
    }
}
```

Note that we guarantee that the thread we're called on is the thread created from.  ex this is a dynamic requirement.

Since we create it from the main actor, let's enforce that here.


###  39:32 - Put the CoffeeLocationDelegate on the main actor
```swift
@MainActor class CoffeeLocationDelegate: NSObject, CLLocationManagerDelegate {
    
    var location: CLLocation?
    var manager: CLLocationManager!
    
    var latitude: CLLocationDegrees? {
        location?.coordinate.latitude
    }
    
    var longitude: CLLocationDegrees? {
        location?.coordinate.longitude
    }
    
    override init () {
        super.init()
        // This CLLocationManager will be initialized on the main thread
        manager = CLLocationManager()
        manager.delegate = self
        manager.startUpdatingLocation()
    }
    
    // error: Main actor-isolated instance method 'locationManager_:didUpdateLocations:)' cannot be used to satisfy nonisolated protocol requirement
    func locationManager(
        _ manager: CLLocationManager,
        didUpdateLocations locations: [CLLocation]
    ) {
        self.location = locations.last
    }
}
```

###  40:06 - Update the locationManager function
```swift
nonisolated func locationManager(
    _ manager: CLLocationManager,
    didUpdateLocations locations: [CLLocation]
) {
    MainActor.assumeIsolated {
        self.location = locations.last
    }
}
```

# Wrap up
* take your time
* eliminate simple issues
* refactor t improve code *after* swift 6 is enabled
* see the migration guide for more examples
[[Swift concurrency update a sample app]]
# Resources
* https://www.swift.org/migration/documentation/migrationguide/
* https://developer.apple.com/documentation/Swift/updating-an-app-to-use-strict-concurrency
