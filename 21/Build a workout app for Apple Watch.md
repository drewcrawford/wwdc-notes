#watchOS 

# Code-along
Workout app tracks fitness activty during a workout.  Metrics are displayed.  Summary.
# Workout views
Bike, run, walk.  

```swift
import HealthKit
var workoutTypes: [HKWorkoutActivityType] = [.cycling, .running, .walking]
extension HKWorkoutActivityType: Identifiable {
    public var id: UInt {
        rawValue
    }

    var name: String {
        switch self {
        case .running:
            return "Run"
        case .cycling:
            return "Bike"
        case .walking:
            return "Walk"
        default:
            return ""
        }
    }
}

```
```swift
List(workoutTypes) { workoutType in
    NavigationLink(
        workoutType.name,
        destination: Text(workoutType.name)
    ).padding(
        EdgeInsets(top: 15, leading: 5, bottom: 15, trailing: 5)
    )
}
.listStyle(.carousel)
.navigationBarTitle("Workouts")
```

Controls view has buttons that control the in-progress session such as end, pause, and resume.

Metrics view
Media playback.
Tab view switches between these child views when swiping left or right.

Tab enum has 3 cses.  Controls, metrics, nowPlaying.  State variable named `selection` to provide a binding.

```swift
TabView(selection: $selection) {
    Text("Controls").tag(Tab.controls)
    Text("Metrics").tag(Tab.metrics)
    Text("Now Playing").tag(Tab.nowPlaying)
}
```

Whiel workout is runnnig, live metrics are displaying.  App should use large font sizes and arrange text so the most important information is easy to read.

## MetricsView
```swift
VStack(alignment: .leading) {
    Text("03:15.23")
        .foregroundColor(Color.yellow)
        .fontWeight(.semibold)
    Text(
        Measurement(
            value: 47,
            unit: UnitEnergy.kilocalories
        ).formatted(
            .measurement(
                width: .abbreviated,
                usage: .workout,
                numberFormat: .numeric(precision: .fractionLength(0))
            )
        )
    )
    Text(
        153.formatted(
            .number.precision(.fractionLength(0))
        )
        + " bpm"
    )
    Text(
        Measurement(
            value: 515,
            unit: UnitLength.meters
        ).formatted(
            .measurement(
                width: .abbreviated,
                usage: .road
            )
        )
    )
}
.font(.system(.title, design: .rounded)
        .monospacedDigit()
        .lowercaseSmallCaps()
)
.frame(maxWidth: .infinity, alignment: .leading)
.ignoresSafeArea(edges: .bottom)
.scenePadding()
```

Adjust for always-on state to hide subsecond intervals

```swift
struct ElapsedTimeView: View {
    var elapsedTime: TimeInterval = 0
    var showSubseconds: Bool = true
    @State private var timeFormatter = ElapsedTimeFormatter()

    var body: some View {
        Text(NSNumber(value: elapsedTime), formatter: timeFormatter)
            .fontWeight(.semibold)
            .onChange(of: showSubseconds) {
                timeFormatter.showSubseconds = $0
            }
    }
}

class ElapsedTimeFormatter: Formatter {
    let componentsFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.minute, .second]
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()
    var showSubseconds = true

    override func string(for value: Any?) -> String? {
        guard let time = value as? TimeInterval else {
            return nil
        }

        guard let formattedString = componentsFormatter.string(from: time) else {
            return nil
        }

        if showSubseconds {
            let hundredths = Int((time.truncatingRemainder(dividingBy: 1)) * 100)
            let decimalSeparator = Locale.current.decimalSeparator ?? "."
            return String(format: "%@%@%0.2d", formattedString, decimalSeparator, hundredths)
        }

        return formattedString
    }
}
```
Integrate these two
## Controls view
When end button is tapped, workout summary will be displayed.  When paused, metrics view will be displayed.

```swift
HStack {
    VStack {
        Button {
        } label: {
            Image(systemName: "xmark")
        }
        .tint(Color.red)
        .font(.title2)
        Text("End")
    }
    VStack {
        Button {
        } label: {
            Image(systemName: "pause")
        }
        .tint(Color.yellow)
        .font(.title2)
        Text("Pause")
    }
}
```

```swift
ControlsView().tag(Tab.controls)
MetricsView().tag(Tab.metrics)
NowPlayingView().tag(Tab.nowPlaying)
```

Now playing view is provided by watchkit.

## Summary view

```swift
struct SummaryMetricView: View {
    var title: String
    var value: String

    var body: some View {
        Text(title)
        Text(value)
            .font(.system(.title2, design: .rounded)
                    .lowercaseSmallCaps()
            )
            .foregroundColor(.accentColor)
        Divider()
    }
}
```
```swift
@State private var durationFormatter: DateComponentsFormatter = {
    let formatter = DateComponentsFormatter()
    formatter.allowedUnits = [.hour, .minute, .second]
    formatter.zeroFormattingBehavior = .pad
    return formatter
}()
```

```swift
ScrollView(.vertical) {
    VStack(alignment: .leading) {
        SummaryMetricView(
            title: "Total Time",
            value: durationFormatter.string(from: 30 * 60 + 15) ?? ""
        ).accentColor(Color.yellow)
        SummaryMetricView(
            title: "Total Distance",
            value: Measurement(
                value: 1625,
                unit: UnitLength.meters
            ).formatted(
                .measurement(
                    width: .abbreviated,
                    usage: .road
                )
            )
        ).accentColor(Color.green)
        SummaryMetricView(
            title: "Total Energy",
            value: Measurement(
                value: 96,
                unit: UnitEnergy.kilocalories
            ).formatted(
                .measurement(
                    width: .abbreviated,
                    usage: .workout,
                    numberFormat: .numeric(precision: .fractionLength(0))
                )
            )
        ).accentColor(Color.pink)
        SummaryMetricView(
            title: "Avg. Heart Rate",
            value: 143
                .formatted(
                    .number.precision(.fractionLength(0))
                )
            + " bpm"
        ).accentColor(Color.red)
        Button("Done") {
        }
    }
    .scenePadding()
}
.navigationTitle("Summary")
.navigationBarTitleDisplayMode(.inline)
```

Activity rings
```swift
import HealthKit
import SwiftUI

struct ActivityRingsView: WKInterfaceObjectRepresentable {
    let healthStore: HKHealthStore

    func makeWKInterfaceObject(context: Context) -> some WKInterfaceObject {
        let activityRingsObject = WKInterfaceActivityRing()

        let calendar = Calendar.current
        var components = calendar.dateComponents([.era, .year, .month, .day], from: Date())
        components.calendar = calendar

        let predicate = HKQuery.predicateForActivitySummary(with: components)

        let query = HKActivitySummaryQuery(predicate: predicate) { query, summaries, error in
            DispatchQueue.main.async {
                activityRingsObject.setActivitySummary(summaries?.first, animated: true)
            }
        }

        healthStore.execute(query)

        return activityRingsObject
    }

    func updateWKInterfaceObject(_ wkInterfaceObject: WKInterfaceObjectType, context: Context) {

    }
}
```
```swift
Text("Activity Rings")
ActivityRingsView(
    healthStore: HKHealthStore()
).frame(width: 50, height: 50)
```

# HealthKit integration
* HKWorkoutSession => Prepares device sensors for data collection so you can accurately collect data relevant to the workout.
* Allows app to run in bg
* HKLiveWorkoutBuilder => Create and save workout object

[[New ways to work with workouts - 18]]

## Data flow
Views will then declare an environment object to gain access to the workout manager.

## WorkoutManager
```swift
import HealthKit

class WorkoutManager: NSObject, ObservableObject {

}
```

```swift
.environmentObject(workoutManager)
```
```swift
var selectedWorkout: HKWorkoutActivityType?
```
```swift
@EnvironmentObject var workoutManager: WorkoutManager
```
```swift
func startWorkout(workoutType: HKWorkoutActivityType) {
    let configuration = HKWorkoutConfiguration()
    configuration.activityType = workoutType
    configuration.locationType = .outdoor

    do {
        session = try HKWorkoutSession(healthStore: healthStore, configuration: configuration)
        builder = session?.associatedWorkoutBuilder()
    } catch {
        // Handle any exceptions.
        return
    }

    builder?.dataSource = HKLiveWorkoutDataSource(
        healthStore: healthStore,
        workoutConfiguration: configuration
    )

    // Start the workout session and begin data collection.
    let startDate = Date()
    session?.startActivity(with: startDate)
    builder?.beginCollection(withStart: startDate) { (success, error) in
        // The workout has started.
    }
}
```
```swift
{
    didSet {
        guard let selectedWorkout = selectedWorkout else { return }
        startWorkout(workoutType: selectedWorkout)
    }
}
```

Need to setup healthkit and request authorization.

```swift
// Request authorization to access HealthKit.
func requestAuthorization() {
    // The quantity type to write to the health store.
    let typesToShare: Set = [
        HKQuantityType.workoutType()
    ]

    // The quantity types to read from the health store.
    let typesToRead: Set = [
        HKQuantityType.quantityType(forIdentifier: .heartRate)!,
        HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned)!,
        HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning)!,
        HKQuantityType.quantityType(forIdentifier: .distanceCycling)!,
        HKObjectType.activitySummaryType()
    ]

    // Request authorization for those quantity types.
    healthStore.requestAuthorization(toShare: typesToShare, read: typesToRead) { (success, error) in
        // Handle error.
    }
}
```

Request authorization when view appears

```swift
.onAppear {
    workoutManager.requestAuthorization()
}
```

Add capability, add usage descriptions.  `NSHealthShareUsageDescription`: read

`NSHealthUpdateUsageDescription`: write

## Session state

```swift
// MARK: - State Control

// The workout session state.
@Published var running = false

func pause() {
    session?.pause()
}

func resume() {
    session?.resume()
}

func togglePause() {
    if running == true {
        pause()
    } else {
        resume()
    }
}

func endWorkout() {
    session?.end()
}
```

Listen for session changes.

```swift
// MARK: - HKWorkoutSessionDelegate
extension WorkoutManager: HKWorkoutSessionDelegate {
    func workoutSession(_ workoutSession: HKWorkoutSession,
                        didChangeTo toState: HKWorkoutSessionState,
                        from fromState: HKWorkoutSessionState,
                        date: Date) {
        DispatchQueue.main.async {
            self.running = toState == .running
        }

        // Wait for the session to transition states before ending the builder.
        if toState == .ended {
            builder?.endCollection(withEnd: date) { (success, error) in
                self.builder?.finishWorkout { (workout, error) in
                }
            }
        }
    }

    func workoutSession(_ workoutSession: HKWorkoutSession, didFailWithError error: Error) {

    }
}
```

Update sesison paging view for state

```swift
@EnvironmentObject var workoutManager: WorkoutManager
```

```swift
.navigationTitle(workoutManager.selectedWorkout?.name ?? "")
.navigationBarBackButtonHidden(true)
.navigationBarHidden(selection == .nowPlaying)
```

```swift
.onChange(of: workoutManager.running) { _ in
        displayMetricsView()
    }
}

private func displayMetricsView() {
    withAnimation {
        selection = .metrics
    }
}
```

Show/dismiss summary view

```swift
@Published var showingSummaryView: Bool = false {
    didSet {
        // Sheet dismissed
        if showingSummaryView == false {
            selectedWorkout = nil
        }
    }
}
```

```swift
.sheet(isPresented: $workoutManager.showingSummaryView) {
    SummaryView()
}
```

```swift
@Environment(\.dismiss) var dismiss
```

This is a function you call from some action

## Live metrics
```swift
// MARK: - Workout Metrics
@Published var averageHeartRate: Double = 0
@Published var heartRate: Double = 0
@Published var activeEnergy: Double = 0
@Published var distance: Double = 0
```

Make this manager some delegate to find out about these emtrics

```swift
// MARK: - HKLiveWorkoutBuilderDelegate
extension WorkoutManager: HKLiveWorkoutBuilderDelegate {
    func workoutBuilderDidCollectEvent(_ workoutBuilder: HKLiveWorkoutBuilder) {
    }

    func workoutBuilder(_ workoutBuilder: HKLiveWorkoutBuilder, didCollectDataOf collectedTypes: Set<HKSampleType>) {
        for type in collectedTypes {
            guard let quantityType = type as? HKQuantityType else { return }

            let statistics = workoutBuilder.statistics(for: quantityType)

            // Update the published values.
            updateForStatistics(statistics)
        }
    }
}
```

updateforStatistics

```swift
func updateForStatistics(_ statistics: HKStatistics?) {
    guard let statistics = statistics else { return }

    DispatchQueue.main.async {
        switch statistics.quantityType {
        case HKQuantityType.quantityType(forIdentifier: .heartRate):
            let heartRateUnit = HKUnit.count().unitDivided(by: HKUnit.minute())
            self.heartRate = statistics.mostRecentQuantity()?.doubleValue(for: heartRateUnit) ?? 0
            self.averageHeartRate = statistics.averageQuantity()?.doubleValue(for: heartRateUnit) ?? 0
        case HKQuantityType.quantityType(forIdentifier: .activeEnergyBurned):
            let energyUnit = HKUnit.kilocalorie()
            self.activeEnergy = statistics.sumQuantity()?.doubleValue(for: energyUnit) ?? 0
        case HKQuantityType.quantityType(forIdentifier: .distanceWalkingRunning), HKQuantityType.quantityType(forIdentifier: .distanceCycling):
            let meterUnit = HKUnit.meter()
            self.distance = statistics.sumQuantity()?.doubleValue(for: meterUnit) ?? 0
        default:
            return
        }
    }
}
```

Use these

```swift
VStack(alignment: .leading) {
    ElapsedTimeView(
        elapsedTime: workoutManager.builder?.elapsedTime ?? 0,
        showSubseconds: true
    ).foregroundColor(Color.yellow)
    Text(
        Measurement(
            value: workoutManager.activeEnergy,
            unit: UnitEnergy.kilocalories
        ).formatted(
            .measurement(
                width: .abbreviated,
                usage: .workout,
                numberFormat: .numeric(precision: .fractionLength(0))
            )
        )
    )
    Text(
        workoutManager.heartRate
            .formatted(
                .number.precision(.fractionLength(0))
            )
        + " bpm"
    )
    Text(
        Measurement(
            value: workoutManager.distance,
            unit: UnitLength.meters
        ).formatted(
            .measurement(
                width: .abbreviated,
                usage: .road
            )
        )
    )
}
```

Builders elapsed time doesn't update.  What we can do is wrap the vstack in a timeline view.

[[What's new in watchOS 8]]
[[21/What's new in SwiftUI]]

Updates over time in line with its schedule.  Makes the view aware of changes to the always-on context.

With workout sessions, can update *at most* once per seconds.  So hide subseconds in always-on state.

Need a custom schedule to change interval based on timeline schedule mode.

```swift
private struct MetricsTimelineSchedule: TimelineSchedule {
    var startDate: Date

    init(from startDate: Date) {
        self.startDate = startDate
    }

    func entries(from startDate: Date, mode: TimelineScheduleMode) -> PeriodicTimelineSchedule.Entries {
        PeriodicTimelineSchedule(
            from: self.startDate,
            by: (mode == .lowFrequency ? 1.0 : 1.0 / 30.0)
        ).entries(
            from: startDate,
            mode: mode
        )
    }
}
```

```swift
TimelineView(
    MetricsTimelineSchedule(
        from: workoutManager.builder?.startDate ?? Date()
    )
) { context in
    VStack(alignment: .leading) {
        ElapsedTimeView(
            elapsedTime: workoutManager.builder?.elapsedTime ?? 0,
            showSubseconds: context.cadence == .live
        ).foregroundColor(Color.yellow)
        Text(
            Measurement(
                value: workoutManager.activeEnergy,
                unit: UnitEnergy.kilocalories
            ).formatted(
                .measurement(
                    width: .abbreviated,
                    usage: .workout,
                    numberFormat: .numeric(precision: .fractionLength(0))
                )
            )
        )
        Text(
            workoutManager.heartRate
                .formatted(
                    .number.precision(.fractionLength(0))
                )
            + " bpm"
        )
        Text(
            Measurement(
                value: workoutManager.distance,
                unit: UnitLength.meters
            ).formatted(
                .measurement(
                    width: .abbreviated,
                    usage: .road
                )
            )
        )
    }
    .font(.system(.title, design: .rounded)
            .monospacedDigit()
            .lowercaseSmallCaps()
    )
    .frame(maxWidth: .infinity, alignment: .leading)
    .ignoresSafeArea(edges: .bottom)
    .scenePadding()
}
```

## Summary view
```swift
@Published var workout: HKWorkout?
```

```swift
DispatchQueue.main.async {
    self.workout = workout
}
```

```swift
func resetWorkout() {
    selectedWorkout = nil
    builder = nil
    session = nil
    workout = nil
    activeEnergy = 0
    averageHeartRate = 0
    heartRate = 0
    distance = 0
}
```

Call when summary dismisses.

```swift
resetWorkout()
```

## Progress view
While workout is saving.

```swift
@EnvironmentObject var workoutManager: WorkoutManager
```

Display the progress view until workout manager has HKWorkout assigned (indicating saved)

```swift
if workoutManager.workout == nil {
    ProgressView("Saving workout")
        .navigationBarHidden(true)
} else {
    ScrollView(.vertical) {
        VStack(alignment: .leading) {
            SummaryMetricView(
                title: "Total Time",
                value: durationFormatter.string(from: 30 * 60 + 15) ?? ""
            ).accentColor(Color.yellow)
            SummaryMetricView(
                title: "Total Distance",
                value: Measurement(
                    value: 1625,
                    unit: UnitLength.meters
                ).formatted(
                    .measurement(
                        width: .abbreviated,
                        usage: .road
                    )
                )
            ).accentColor(Color.green)
            SummaryMetricView(
                title: "Total Calories",
                value: Measurement(
                    value: 96,
                    unit: UnitEnergy.kilocalories
                ).formatted(
                    .measurement(
                        width: .abbreviated,
                        usage: .workout,
                        numberFormat: .numeric(precision: .fractionLength(0))
                    )
                )
            ).accentColor(Color.pink)
            SummaryMetricView(
                title: "Avg. Heart Rate",
                value: 143.formatted(
                    .number.precision(.fractionLength(0))
                )
                + " bpm"
            )
            Text("Activity Rings")
            ActivityRingsView(healthStore: workoutManager.healthStore)
                .frame(width: 50, height: 50)
            Button("Done") {
                dismiss()
            }
        }
        .scenePadding()
    }
    .navigationTitle("Summary")
    .navigationBarTitleDisplayMode(.inline)
}
```

Luminance reduced?

```swift
.tabViewStyle(
    PageTabViewStyle(indexDisplayMode: isLuminanceReduced ? .never : .automatic)
)
.onChange(of: isLuminanceReduced) { _ in
    displayMetricsView()
}
```

Hide the dots I guess?

# Wrap up
* SwiftUI
* HealthKit integration
* Always On state

https://developer.apple.com/documentation/healthkit/workouts_and_activity_rings/build_a_workout_app_for_apple_watch

