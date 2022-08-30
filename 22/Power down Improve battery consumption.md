Use can come at a cost: battery drain.  Improtant to take special attentiont oimprove app's battery life so that users can use their devices and apps longer.  

Four key actions to improve battery life

# Adopt dark mode
Users can switch between light and dark presentation
Apps can match device presentation
Enabling dark mode can reduce battery consumption

Pixels on OLED displays consume varying levels of power
Less luminous pixels consume lessp ower
Dark mode reuslts in less pixel luminance

**Among all of the components in the system, display is one of the major sources of power consumption**.  In typical use cases, display is the leading contributor to battery drain.  You have a way to influence power consumption, by adopting dark mode.

For cases like this, we expect up to 70% display power savings as a result.  This is a huge savings!  When screen brightness is high, battery savings are even higher.  For users who prefer dark mode, this is a tremendous opportunity to save them some battery drain and it can reduce thermal load too.

Start by reviewing.  Xcode makes this easy by using your appearahnce feature.

Your app may have hardcoded colors.  Use dynamic colors in xcode to support background color.

system will update colors based on mode.

Use alternate images for different modes.

[[Implementing dark mode on iOS]]

Safari does not auto-darken any web content, amke sure you adopt DM for your web content too.
`color-scheme` style property
stylesheet vafriables for text and background colors
`prefers-color-scheme` for picture and media query
choose other image variants for web content in Dark Mode

[[Supporting Dark Mode in your Web Content - 19]]

# Audit frame rates
* Multiple elements compose frame
* Each element has an independent frame rate
* Screens adapt to the element with highest frame rate
* High frame rate elements causes high screen refresh rate
* Higher screen refresh rate consumes more battery

60fps->30fps, up to 20% of battery drain.  To debug, use Instruments.  CoreAnimation FPS instrument to see a timeline showing the frame rate of your app over time.  Start by auditing main user scenarios.

Determine if secondary elements have a higher frame rate than primary content.

* CADisplayLink is synchronized with display refresh rate
* Give hints to the system about desired refresh rates.
* Set `preferredFrameRateRange` with minimum, maximum, preferred rates
* System determines the rate it can handle based on your hint
	* Will try to stay within range if it can't hit the preferred rate.

```swift
// Create a display link

func createDisplayLink() {
   let displayLink = CADisplayLink(target: self, selector: #selector(step))

    // Configure your desired refresh rate by calling preferredFrameRateRange
    displayLink.preferredFrameRateRange = CAFrameRateRange(minimum: 10,
                                                           maximum: 60,
                                                           preferred: 30)

// then activate your CADisplayLink by adding it to the main runloop.
    displayLink.add(to: .current, forMode: .defaultRunLoopMode)
}
```

* battery consumption increases with refresh rate
* Audit your app to avoid unexpected frame rates
* Inform the system of your content's frame rate requirements

[[Optimize for variable-refresh displays]]

# Limit background time
When someone switches to a different app, your app may rely on background execution to ocntinue to run.  Continue to use common servicesl ike location and audio.  Long durations will cause drain. When you are using these services in the bg, be careful.

* location services drain significant battery
* Ensure your app stops background services
	* For location, call `stopUpdatingLocation()`
* Tools to detect background location sessions
	* Xcode gauges
	* MetricKit
	* Control center (location usage)

## Audio session
* stop audio engine when palyback stops
* Ensure hardware is stopped when idle to save power
* Use autoshutdownEnabled property of AVAudioEngine
	* Enforced behavior on watchOS

Key to limiting bg runtime, is to remember to tell the system when you are done.

# Defer work
Over the course of a day, you may process different tasks and data.  Some needs to occur immediately.  Other work like ML tasks, uploading analytics, or backups, is not as time sensitive.  If we defer time-insensitive work, when the device is charging, we can save battery and avoid contending with interactive work.

* BGProcessingTask => long durations
* discretionary URLSession => networking
* Push priority => help servers deliver pushes at the right time.

## BGProcessingTask
Defer long-running tasks to a better time
Database cleanup, backups, ML training
Use BGProcessingTaskRequest
Set `requiresExternalPower` and `requiresNetworkConnectivity` properties.  Helps the system schedule the task at a better window.

> Several minutes of runtime

## Discretionary URLSession
* allows the system to defer the download until better time
* discretionary flag for smart scheduling
* Non-user-initiated, long-running, networking.  Downloading next episode, etc.
Because networking is offloaded, your app does not need to be runnign while network transaction completes.

Set a bg URL session and set `isDiscretionary` to true.  You can provide additional information to hepl the system schedule the download at the right time.  Pass in timeout intervals so that the system does not attempt to download forever.

`earliestBeginDate` to defer until the future.  Pass in an expected workload size sot hat the system can load balance between various download tasks.

```swift
// Set up background URL session 
let config = URLSessionConfiguration.background(withIdentifier: "com.app.attachments") 
let session = URLSession(configuration: config, delegate: ..., delegateQueue: ...) 

// Set discretionary 
config.isDiscretionary = true

// Set timeout intervals
config.timeoutIntervalForResource = 24 * 60 * 60 
config.timeoutIntervalForRequest = 60 

// Create request and task 
var request = URLRequest(url: url) 
request.addValue("...", forHTTPHeaderField: "...") 
let task = session.downloadTask(with: request) 

// Set time window of two hours
task.earliestBeginDate = Date(timeIntervalSinceNow: 2 * 60 * 60) 

// Set workload size 
task.countOfBytesClientExpectsToSend = 160 
task.countOfBytesClientExpectsToReceive = 4096 

task.resume()
```

## Push priority
High priority pushes wake up the system
increased battery drain
only use high priority pushes for urgent notifications

Low prioroity pushes utilize system resources better
Set `apns-priority` to 5
Coalesces multiple pushes together

# Next steps
* respect user intent with dark mode
* Review your animations for frame rate
* Be aware of background sessions
* Defer work to better time


* https://developer.apple.com/documentation/backgroundtasks/refreshing_and_maintaining_your_app_using_background_tasks
* https://developer.apple.com/documentation/uikit/appearance_customization/adopting_ios_dark_mode
* https://developer.apple.com/documentation/backgroundtasks

