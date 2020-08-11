Just as you wouldn't want an app to crash in fg, it can be disruptive to crash in bg.

# Top causes of app terminations
* crashes
* CPU resource limit
* watchdog
* memory limit exceeded
* memory pressure exit
* background task timeout

New MericKit API to find out how often this happens.

`MXBAckgroundExitData` provides counts.  Includes "normal exits", where user terminates your app in the app switcher.

# Crashes
 * segmentation fault
 * illegal instruction
 * asserts and uncaught exceptions

Surfaced in xcode organizer

[[understanding crashes and crash logs - 18]]

## Crash reporting via MetricKit
* diagnostics on a per-device basis
* MXCrashDiagnostic
	* stack trace
	* signal
	* exception code
	* termination reason

# Watchdog
Timeout during key transitions
* `application(_:didFinishLaunchingWithOptions:)`
* `applicationDidEnterBackground(_:)`
* `applicationWillEnterForeground(_:)`

Disabled in simulator and in the debugger ("while the debugger is attached")
Eliminate deadlocks, infinite loops, synchronous work
Report available via `MXCrashDiagnostic`

"On the order of 20s".  

# CPU resource limit
* High sustained CPU load in background
* Energy exception report
	* Xcode organizer
	* MXCPUExceptionDiagnostic
* call stack points out hotspots in code
* Consider moving work into `BGProcessingTask`.  This gives you several minutes of runtime while the device charges overnight, without CPU resource limits.

# Memory footprint exceeded
* app using too much memory
* same limit for fg and bg
* Use Instruments and Memory Debugger
* Keep in mind older devices.

If your app targets devices before iPhone 6s, aim to stay under 200mb at all times.

# Memory pressure exit
aka *jetsam*.
Not a bug with your app
most common termination
System freeing up memory for active applications

Reducing jetsam rate
* aim for less than 50mb in background
* upon backgrounding
	* flush state to disk
	* clear out image views
	* drop caches

jetsam is inevitable.  When it happens is unpredictable.  

## Recovering from jetsam
* save state upon entering background
* view controller stack
* draft input in text fields
* media playback position
* Adopt UIKit state restoration
* Users should not realize app was terminated.

# BG task timeout
`UIApplication.beginBackgroundTask(expirationhandler:)`
`UIApplication.endBackgroundTask(_:)`
Failure to end the task explicitly result in termination
Counts exposed via `MXBackgroundExitData`
Preventable

~30s.  As long as you end all tasks within 30s, you will end gracefully.

We don't surface crash logs for this, but we do surface stats via `MXBackgroundExitData`.

## provide a task name
`beginBackgroundTask(withName:expirationhandler:)`.  This is preferred over the other deprecated variant.

Terminations **do not occur in Debugger**
New console message, even in the foreground.  Do an audit of your calls if you see this.
```
Background Task DatabaseTransaction was created over 30 seconds ago, in applications running in the background, this creates a risk of termination.  Rmember to call UIApplication.endBackgroundTask(_::) for your task in a timely manner to avoid this.
```


## expiration handler
Implement an `expirationHandler`.
Note that it's safe to call `endBackgroundTask(_:)` inside the handler
Do not beigin new work ("you only have a few seconds")
Safety net
do not rely on it exclusively

## expiration handler telemetry
Add telemetry at the start/end of each expiration handler.
```swift
let handle = MXMetricManager.makeLogHandle(category: "DatabaseExpirationHandler")

mxSignpost(.event, log: handle, name: "Entered")
cancelOperations()
closeDatabase()
mxSignpost(.event, log: handle, name: "Exited")

UIApplication.shared.endBackgroundTask(backgroundTaskIdentifier)
```
You want to watch for "entered" signpost count being greater than the "exited" signpost count.

## check `backgroundTimeRemaining`
"You should be extra careful in starting tasks while the app is already in backgrounded since expiration handler won't be called if you start a task with fewer than 5s remaining"

Only start work if plenty of time remains
Unsafe to begin tasks with <5s remaining

```swift
let minimumTimeRemaining = min(5,estimateProcessingTime(inputData))
if UIApplication.shared.backgroundTimeRemaining > minimumTimeRemaining {
  //enough time remains, call beginBackgroundTask
  return UIApplication.shared.beginBackgroundTask { ... }
} else {
	//not enough time remains, defer this work until later
	registerProcessingTask(inputData)
	return .invalid
}
```

## Avoid leaking `UIBackgroundTaskIdentifier`

Storing in an instance variable... trouble.
```swift
class Foo: UIViewController {
	var backgroundTaskidentifier: UIBackgroundTaskIdentifier = .invalid
	
	@IBAction func beginDataExport(sender: UIButton) {
		self.backgroundTaskidentifier = UIApplication.shared.beginBackgroundTask { .. }
		
		//end bg task after archiving, which takes several seconds
		ArchiveUtility.exportUserData(completion: () -> ()) {
			UIApplication.shared.endBackgroundTask(self.backgroundTaskIdentifier)
		}
	}
}
```

If I tap this button several times, I can end up with multiple outstanding bg tasks, which are clobbered since ivar only holds 1 id.

Consider using an lvar to hold the identifier instead.

# Reduce terminations!
* Identify and fix terminations
* Reduce memory usage
* Implement state restoration