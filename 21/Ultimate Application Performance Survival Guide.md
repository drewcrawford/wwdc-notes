Take your development to the next level.

# Introduction
* Battery usage
* launch time
* hang rate
* memory disk writes
* scrolling
* terminations
* MXSignposts


# Metrics and resolutions
## Battery usage
battery report.  Shows foreground/background activity, etc.  Users prioritize using apps that allow them to use their apps throughout the day without needing to recharge.

* CPU (>20% is high)
* Networking
* Location

Debug navigator (spraybottle).  
Energy gauge.  Common to see a spike in CPU when drawing UX, etc.  But once those tasks are complete and my app is idle, I should see ~0%.

Can press "Time profile" to go direct to instruments.  

## MetricKit
Can help me narrow down the root cause and valuable inisghts.

```swift
class AppMetrics: MXMetricManagerSubscriber {
	init() {
		let shared = MXMetricManager.shared
		shared.add(self)
	}

	deinit {
		let shared = MXMetricManager.shared
		shared.remove(self)
	}

	// Receive daily metrics
	func didReceive(_ payloads: [MXMetricPayload]) {
		// Process metrics
	}

	// Receive diagnostics
	func didReceive(_ payloads: [MXDiagnosticPayload]) {
		// Process metrics
	}
}
```

 Analytics pipeline at a glance.  Only gathered from devices with explicit consent.
 
 New in xcode 13, we have a regressions pane.  Metrics that have increased significantly in the most recent version so I can see new issues.
 
 Energy organizer under reports can view regions of high CPU and logs.
 
 ASC API can also return this data via JSON.
 
 [[Improving Battery Life and Performance - 19]]
 [[Analyze HTTP traffic in Instruments]]
 
 ## Hang rate  
 
 Hang - was unresponsive for user input for >=250ms.
 Customers may forcequit and are impediment to UX.
 
 Stuttering scrolls appear when new content isn't ready for next screen refresh.  Unenjoyable UX. Maximize user engagement, so a great place to start optimizing.  Aim for smooth scroll.
 
 Xcode organizer.  Watch for upward trends, or yellow/red bars instead of green (scrolling).
 
 Red bar - poor scroll experience.
 Also available through ASC API.
 
 Instrumenst can detect the cause of hang by Thread State or System Call Trace.
 
 ```swift
 func testScrollingAnimationPerformance() throws {
        
    app.launch()
    app.staticTexts["Meal Planner"].tap()
    let foodCollection = app.collectionViews.firstMatch

    let measureOptions = XCTMeasureOptions()
    measureOptions.invocationOptions = [.manuallyStop]
        
    measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric],
    options: measureOptions) {
        foodCollection.swipeUp(velocity: .fast)
        stopMeasuring()
        foodCollection.swipeDown(velocity: .fast)
    }
}
```

MetricKit can allow me to collect diagnostics with `MXDiagnosticPayload`.  MetricKit delivers hangs at 24-hour cadence.

In iOS15/macOS 12, I now receive *all* diagnostics (including hangs) immediately after the issue.  I can quickly root-cause and resolve the most pressing responsiveness problems.

```swift
func startAnimating() {
	// Mark the beginning of animations
	mxSignpostAnimationIntervalBegin(
		log: MXMetricManager.makeLogHandle(category: "animation_telemetry"), 
		name: "custom_animation”)
	}

	func animationDidComplete() {
	// Mark the end of the animation to receive the collected hitch rate telemetry
	mxSignpost(OSSignpostType.end, 
		log: MXMetricManager.makeLogHandle(category: "animation_telemetry"), 
		name: "custom_animation")
}
```

I can strategically mark custom animations and catch hit rate telemetry during the interval.  

[[Understand and eliminate hangs from your app]]

[[Explore UI animation hitches and the render loop]]
[[Eliminate animation hitches with XCTest]]

## Disk writes
Important to batch writes.  I can profile my app using the file activity template in instruments.  

* Batch write operations
* Use coredata
* avoid rapid file creation/deletion

Use XCTest to look for disk writes

```swift
// Example performance XCTest

func testSaveMeal() {
	let app = XCUIApplication()
	let options = XCTMeasureOptions()
	options.invocationOptions = [.manuallyStart]

	measure(metrics: [XCTStorageMetric(application: app)], options: options) {
		app.launch()
		startMeasuring()

		let firstCell = app.cells.firstMatch
		firstCell.buttons["Save meal"].firstMatch.tap()

		let savedButton = firstCell.buttons["Saved"].firstMatch
		XCTAssertTrue(savedButton.waitForExistence(timeout: 2))
	}
}
```

Test fails if the code in the block exceeds a limit.

I can use organizer to track performance over time.  Disk writes shows me the trend of current vs previous writes.

Spikes can indicate my app has bugs that cause a high amount of writes.  

Look for source of writes by looking at Disk Writes reports in Xcode organizer.  Generated when my app writes more than 1GB in a 24-hour period.  Has stacktrace.

Now in xc13, I can get additional details called "Insights" in the righthand pane.

Data is now available thorugh ASC.

Obtain through metrickit, similar to other bookend code.

[[Diagnose power and performance regressions in your app]]

## Launch time
User taps icon vs
first frame rendered

Terminations.  Goes through full launch next time.

If you're not restoring state, this can also add to frustration.

Since launch is already in a version of the app, I can start by looking in organizer in "Launch Time" and "New Terminations?"

Terminations => How frequently terminated by the system "because of how long it's taking to launch".

MetricKit again.

[[Why is my app getting killed]]

## Memory
Shared resource between apps, OS, and kernel.  Terminated if exceed the limit.

Keep an eye on memory and terminations metrics to make sure that isn't the case.  

Large spike in memory use.  Peak memory.

Profile the memory use of my app by leaks, allocations, and VM tracker templates in instruments.

MetricKit can get the same information etc.

[[Detect and diagnose memory issues]]

# Next steps
Developers have used the same tools we provide, to make significant perf optimizations.

99% reduction in terminations in snapchat.  You can accomplish this too.

[[Diagnose Performance Issues with the Xcode Organizer]]
[[What's new in MetricKit]]
[[Identify Trends with the Power and Performance API]]
[[getting started with instruments]]

* Use the organizer to track metrics like launch time and terminations
* Explore time profiler
* write XCTests
* Implement MetricKit in your app to supplement your analytics

