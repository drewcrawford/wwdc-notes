Learn how to incorporate mental health and wellbeing into your app using HealthKit. There are new APIs for State of Mind, as well as for Depression Risk and Anxiety Risk. We'll dive into principles of emotion science to cover how reflecting on feelings can be beneficial, and how State of Mind can be used to represent different types of mood and emotion.

### 5:37 - Request authorization to read and write State of Mind HealthKit samples
```swift
// Request authorization to read and write State of Mind HealthKit samples
import HealthKitUI

func healthDataAccessRequest(
    store: HKHealthStore,
    shareTypes: Set<HKSampleType>,
    readTypes: Set<HKObjectType>? = nil,
    trigger: some Equatable,
    completion: @escaping (Result<Bool, any Error>) -> Void
) -> some View {
    // code goes here
}
```

### 6:26 - EmojiType
```swift
// EmojiType enum
enum EmojiType: CaseIterable {
    case angry
    case sad
    case indifferent
    case satisfied
    case happy
    
    var emoji: String {
        switch self {
        case .angry:
            return "😡"
        case .sad:
            return "😢"
        case .indifferent:
            return "😐"
        case .satisfied:
            return "😌"
        case .happy:
            return "😊"
        }
    }
}
```

### 6:32 - Create State of Mind sample for an event and emoji selection
```swift
/// Create State of Mind sample for an event and emoji selection
func createSample(for event: EventModel, emojiType: EmojiType) -> HKStateOfMind {
    let kind: HKStateOfMind.Kind = .momentaryEmotion
    let valence: Double = emojiType.valence
    let label = emojiType.label
    let association = event.association
    
    return HKStateOfMind(
        date: event.endDate,
        kind: kind,
        valence: valence,
        labels: [label],
        associations: [association]
    )
}
```

### 7:21 - Save State of Mind sample from emoji choice
```swift
// Save State of Mind sample from emoji choice
func save(sample: HKSample, healthStore: HKHealthStore) async {
    do {
        try await healthStore.save(sample)
    } catch {
        // Handle error here.
    }
}
```

### 10:34 - Query State of Mind samples
```swift
// Query State of Mind samples
let datePredicate: NSPredicate = { ... }
let associationsPredicate = NSCompoundPredicate(
    orPredicateWithSubpredicates: associations.map { HKQuery.predicateForStatesOfMind(with: $0) }
)
let compoundPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [datePredicate, associationsPredicate]
)
let state0fMindPredicate = HKSamplePredicate.stateOfMind(compoundPredicate)
```

### 10:49 - Query State of Mind samples
```swift
// Query State of Mind samples
let datePredicate: NSPredicate = { ... }
let associationsPredicate = NSCompoundPredicate(
    orPredicateWithSubpredicates: associations.map { HKQuery.predicateForStatesOfMind(with: $0) }
)
let compoundPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [datePredicate, associationsPredicate]
)
let stateOfMindPredicate = HKSamplePredicate.stateOfMind(compoundPredicate)
let descriptor = HKSampleQueryDescriptor(
    predicates: [stateOfMindPredicate],
    sortDescriptors: []
)
var results: [HKStateOfMind] = []
do {
    // Launch the query and wait for the results.
    results = try await descriptor.result(for: healthStore)
} catch {
    // Handle error here.
}
```

### 10:54 - Query State of Mind samples (continued)
```swift
// Adjust each valence value to be from a range of 0.0 to 2.0.
let adjustedValenceResults = results.map { $0.valence + 1.0 }

// Calculate average valence.
let totalAdjustedValence = adjustedValenceResults.reduce (0.0, +)
let averageAdjustedValence = totalAdjustedValence / Double(results.count)

// Convert valence to percentage.
let adjustedValenceAsPercent = Int(100.0 * (averageAdjustedValence / 2.0))
```

### 11:33 - Query for relevant State of Mind samples with a specific label
```swift
// Query for relevant State of Mind samples with a specific label
let label: HKStateOfMind.Label = .happy

// Configure the query
let datePredicate = HKQuery.predicateForSamples(
    withStart: dateInterval.start,
    end: dateInterval.end
)
let associationPredicate = HKQuery.predicateForStatesOfMind(with: association)
let labelPredicate = HKQuery.predicateForStates0fMind(with: label)
let compoundPredicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [datePredicate, associationPredicate, labelPredicate]
)
let stateOfMindPredicate = HKSamplePredicate.stateOfMind(compoundPredicate)
let descriptor = HKAnchoredObjectQueryDescriptor(
    predicates: [state0fMindPredicate],
    anchor: nil
)

// Fetch the results
let results = descriptor.results(for: healthStore)
let samples: [HKStateOfMind] = try await results.reduce([]) { $1.addedSamples }
```

### 11:45 - Process State of Mind sample data
```swift
// Process State of Mind sample data
let happiestSample = samples.max { $0.valence < $1.valence }
let happiestEvent: EventModel? = findClosestEvent(startDate: happiestSample?.startDate, endDate: happiestSample?.endDate)
```

# Resources
https://developer.apple.com/documentation/healthkit
