#screentime 

Last year, we introduced you to screentime API.  

Last year,
* Family controls
* managedSettings
* deviceActivity

brought new capabilities to parental control apps.

# Family controls
 * authorizes acces to screen time API
 * prevents removal and circumvention
 * Privacy presenving tokens for apps used by your fmaily
# managed settings
* set persistent restrictions on device
* provide web ocntent/filtering
* shield apps with custom ui
# Device activity
executes code on start and end of device activity schedules
executes code on usage threshold

Each framework has had updates.  These updates will make API easier and will enhance UX.

# Family controls
In iOs 15, only capable of authorizing child devices.  Now capable of autohrizing independent users from their own device.  No app removability or iCloud signout restrictions are available.

```swift
// APP: Request Authorization

import SwiftUI
import FamilyControls

@main
struct Worklog: App {
    let center = AuthorizationCenter.shared
    var body: some Scene {
        WindowGroup {
            VStack {…}
                .onAppear {
                    Task {
                        do {
                            try await center.requestAuthorization(for: .individual)
                        } catch {
                            print("Failed to enroll Aniyah with error: \(error)")
                        }
                    }
                }
        }
    }
```

Use shared authorization center to do this on first launch.  Either authorizes or errors.  Since app never has run, this asks for approval.  Tapping on allow prompts the user to authenticate with faceid, touchid, or device passcode.  Once authenticated, calling again will not prompt with another alert but will succeed.

Two switches are added in settings for the app.  Screen time, under apps with screen time access list, and another in per-app settings.

Choose to deauthorize the app via either switch.  Using the new indivdual authorization is as simple as using parental controls auth.

# managed settings
a datastore that applies settings to the curreent user or device.  In 15, you could onyl have one managed settings store per process, and device activity extensions had to ahve different stores.  Difficult to change settings and respond to acctivity.  in ios16, up to 50 stores per process.

* named stores
* apps and their extensions share same settings
* option to clear all settings per store


```swift
// MONITOR EXTENSION: Handle Social category at start/end of interval

import DeviceActivity
import ManagedSettings

class WorklogMonitor: DeviceActivityMonitor {
    let database = BarkDatabase()
    override func intervalDidStart(for activity: DeviceActivityName) {
        super.intervalDidStart(for: activity)
        let socialStore = ManagedSettingsStore(named: .social)
        socialStore.clearAllSettings()
    }
    
    override func intervalDidEnd(for activity: DeviceActivityName) {
        super.intervalDidEnd(for: activity)
        let socialStore = ManagedSettingsStore(named: .social)
        let socialCategory = database.socialCategoryToken
        socialStore.shield.applicationCategories = .specific([socialCategory])
        socialStore.shield.webDomainCategories = .specific([socialCategory])
    }
}
```

Didn't our gaming store restrict gaming websites?  Won't the stores conflict?  No, the most restrictive setting always wins.  

# Device activity

In iOS 15, device activity allowed your app to respond to the start and end of timing windows as wella s usage thresholds.  In 16, we're adding a new reporting service which enables you to create completely custom usage reports with swiftui.

Compeltely customize your users' experience
end-user privacy is assured.

```swift
// APP: Top-level view

import SwiftUI
import DeviceActivity

extension DeviceActivityReport.Context {
    static let pieChart = Self(“Pie Chart")
}

@main
struct Worklog: App {
    private let thisWeek = DateInterval(...)
    @State private var context: DeviceActivityReport.Context = .pieChart
    @State private var filter = DeviceActivityFilter(segment: .daily(during: thisWeek))

    var body: some Scene {
        WindowGroup {
            GeometryReader { geometry in
                VStack(alignment: .leading) {
                   DeviceActivityReport(context: context, filter: filter)
                        .frame(height: geometry.size.height * 0.75)
                    
       
    }
}
```

Context/filter: what type of report to draw.  

```swift
// REPORT EXTENSION: Configure Custom Device Activity Report

import SwiftUI
import DeviceActivity

struct PieChartReport: DeviceActivityReportScene {
    let context: DeviceActivityReport.Context = .pieChart
    let content: (PieChartView.Configuration) -> PieChartView
    
    func makeConfiguration(representing data: [DeviceActivityData]) 
        -> PieChartView.Configuration {
        var totalUsageByCategory: [ActivityCategory:TimeInterval]
        totalUsageByCategory = data.map(…)
        
        return PieChartView.Configuration(totalUsageByCategory: totalUsageByCategory)
    }
}
```


Framework invokes `makeConfiguration` when new usage dat ais available.

Here, we use `Configuration`, a view model.

```swift
// REPORT EXTENSION: Configure Custom Device Activity Report

import SwiftUI
import DeviceActivity

struct PieChartView: View {
    struct Configuration {
        let totalUsageByCategory: [ActivityCategory:TimeInterval]
    }
    
    let configuration: Configuration
    
    var body: some View {
        // A complex view that renders a bar graph based on Aniyah’s usage per category.
        PieChart(usage: configuration.totalUsageByCategory)
    }
}
```

Here, we produce the view.
```swift
// REPORT EXTENSION: Draw Custom Device Activity Report

import SwiftUI
import DeviceActivity

@main
struct WorklogReportExtension: DeviceActivityReportExtension {
    var body: some DeviceActivityReportScene {
        PieChartReport { configuration in
            PieChartView(configuration: configuration)
        }
    }
}
```

continues to support existing features, with new ones, etc.

# Recap
* family controls, individual authorizations
* Managed settings, Named Stores
* Device Activity, Device Activity Reporting

Broaden the amount of users capable of engaging with your application.

* https://developer.apple.com/documentation/ManagedSettings
* https://developer.apple.com/documentation/FamilyControls
* https://developer.apple.com/documentation/DeviceActivity
