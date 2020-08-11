#carekit

# CareKit UI
## New Views
### `SimpleTaskView`
SwiftUI is cool now I guess.

### `LabeledValueTaskView`
### `NumericProgressTaskView`
### `FeaturedContentView`
This one is UIKit-based?
Also has a detail view.
### `LinkView`
SwiftUI again.

## Synchronized Views
View basically accepts a query, and then the view is updated and stuff.

Another method that takes a closure where we can return our own view.

## CareKit on #watchOS
Intentionally blank

# CareKitStore
## HealthKit Driven Tasks
previously
`OCKStore`
`OCKSynchronized-StoreManager`
Synchronized View

While `OCKStore` uses CoreData as a source of truth, the other one uses healthkit

`OCKStoreCoordinator` is a megazord store

## FHIR compatability
[[Handling FHIR without getting burned]]

## Remote Synchronization

`OCKRemoteSynchronizable?`

This seems like quite a complex API.  There's a clock, a device revision, etc.

IBM is one partner for server sync.

[[ResearchKit and CareKit Reimagined]]

## Demo
```swift
private struct StoreManagerKey: EnvironmentKey {
	static var defaultValue: OCKSynchronizedStoreManager { ... }
}

extension EnvironmentValues {
	var storeManager: OCKSynchronizedStoreManager {
		get { self[StoreManagerKey.self]}
		set {self[StoreManagerKey.self] = newValue}
	}
}

struct SomeView: View {
	@Environment(\.storeManager) private var storeManager

}
```

# Community updates

