Discover how to use HealthKit to create experiences that take full advantage of the spatial canvas. Learn the capabilities of HealthKit on the platform, find out how to bring an existing iPadOS app to visionOS, and explore the special considerations governing HealthKit during a Guest User session. You'll also learn ways to use SwiftUI, Swift Charts, and Swift concurrency to craft innovative experiences with HealthKit.

### Check whether health data is available - 2:43
```swift
import HealthKit
if HKHealthStore.isHealthDataAvailable() {
    // Configure HealthKit-powered experiences
} else {
    // Omit HealthKit experiences
}
```

### Request authorization to read or write data - 3:03
```swift
import HealthKitUI
import SwiftUI

func healthDataAccessRequest(
    store: HKHealthStore,
    shareTypes: Set<HKSampleType>,
    readTypes: Set<HKObjectType>? = nil,
    trigger: some Equatable,
    completion: @escaping (Result<Bool, any Error>) -> Void
) -> some View {
    // Function body goes here
}
```

### Update number of chart points based on chart’s size - 5:59
```swift
// Update number of chart points based on chart’s size
import SwiftUI
import HealthKit
import Charts

struct ChartView: View {
    @State var chartBinCount: Int
    
    var body: some View {
        Chart {
            // Chart body goes here
        }
        .onGeometryChange(for: Int.self) { proxy in
            // Observe for changes to the chart’s size
            Int(proxy.size.width / 80) // 80 points per chart point
        }
        action: { newValue in
            // Update the number of chart points
            chartBinCount = newValue
        }
    }
}
```

### Open chart as a new window - 6:33
```swift
// Opens chart as a new window
struct NewChartViewerButton: View {
    @Environment(\.openWindow) private var openWindow
    
    var body: some View {
        Button("Open In New Window", systemImage: "plus.rectangle.on.rectangle") {
            openWindow(id: "chart-viewer-window")
        }
    }
}
```

### HealthKit returns a new error if a write is attempted during a Guest User session - 9:00
```swift
let sample = HKStateOfMind(date: date, kind: .momentaryEmotion, valence: valence, labels: [label], associations: [association])
do {
    try await healthStore.save(sample)
} catch {
    switch error {
    case HKError.errorNotPermissibleForGuestUserMode:
        // Drop data generated in a Guest User session
    default:
        // Existing error handling
    }
}
```

### Request authorization to State of Mind datatype - 9:26
```swift
// Request authorization to State of Mind datatype
@main struct HKStateOfMindDataSampleApp: App {
    @State var toggleHealthDataAuthorization = false
    @State var healthDataAuthorized: Bool?
    
    var body: some Scene {
        WindowGroup {
            TabView {
                // Tab view contents go here
            }
            .healthDataAccessRequest(
                store: healthStore,
                shareTypes: [.stateOfMindType()],
                readTypes: [.stateOfMindType()],
                trigger: toggleHealthDataAuthorization
            ) { result in
                switch result {
                case .success:
                    healthDataAuthorized = true
                case .failure(let error as HKError):
                    switch (error.code) {
                    case .errorNotPermissibleForGuestUserMode:
                        // Defer requests for a later time
                    default:
                        // Existing error handling
                    }
                }
                // Other code goes here
            }
        }
    }
}
```

### Save a State of Mind sample from an emoji type - 9:42
```swift
// Saves a State of Mind sample from an emoji type
public func saveSample(
    date: Date,
    association: HKStateOfMind.Association,
    healthStore: HKHealthStore,
    didError: Binding<Bool>
) async -> SaveDetails? {
    do {
        let sample = createSample(date: date, association: association)
        try await healthStore.save(sample)
    } catch {
        switch error {
        case HKError.errorNotPermissibleForGuestUserMode:
            // Drop data you generate in a Guest User session.
            didError.wrappedValue = true
            return SaveDetails(errorString: "Health data is not saved for Guest Users.")
        default:
            // Existing error handling.
            didError.wrappedValue = true
            return SaveDetails(errorString: "Health data could not be saved: \(error)")
        }
    }
    // Other code goes here
}
```

### Present an alert with a message using the given details - 9:58
```swift
// Present an alert with a message using the given details
struct EventView: View {
    @State private var showAlert: Bool = false
    @State private var saveDetails: EmojiType.SaveDetails? = nil
    
    var body: some View {
        EmojiPicker()
            .alert("Unable to Save Health Data", isPresented: $showAlert, presenting: saveDetails, actions: { _ in }, // default OK button
            message: { details in
                Text(details.errorString)
            })
    }
}
```

# Resources
* https://developer.apple.com/documentation/swiftui/bringing_multiple_windows_to_your_swiftui_app
* https://developer.apple.com/documentation/healthkit
* https://support.apple.com/guide/apple-vision-pro/let-others-use-your-apple-vision-pro-dev57f3c667e/visionos
* https://developer.apple.com/documentation/healthkit/visualizing_healthkit_state_of_mind_in_visionos
