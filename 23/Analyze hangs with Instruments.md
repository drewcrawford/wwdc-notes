#instruments 
User interface elements often mimic real-world interactions, including real-time responses. Apps with a noticeable delay in user interaction — a hang — can break that illusion and create frustration. We'll show you how to use Instruments to analyze, understand, and fix hangs in your apps on all Apple platforms. Discover how you can efficiently navigate an Instruments trace document, interpret trace data, and record additional profiling data to better understand your specific hang. If you aren't familiar with using Instruments, we recommend first watching "Getting Started with Instruments." And to learn about other tools that can help you discover hangs in your app, check out "Track down hangs with Xcode and on-device detection."
[[getting started with instruments]]
1.  Find
2. analyze
3. fix

[[Track down hangs with Xcode and on-device detection]]


# What is a hang?
When there's a significant delay, your brain says wait a moment, that's now how this works.

How fast is instant?  
To me, I noticed 100ms when turning on, but not when turning it off.
100ms is somewhat of a threshold.  Significantly smaller delays are not necessarily perceiveable.

250ms - doesn't feel instant anymore.  It's not slow, but the delay is definitely noticeable.

These kind of perceptual thresholds are also from hang reporting.  Below 100ms for discrete interaction (button) feels instant.
Special cases to go below taht, but it's a good goal to aim for.
100-250ms - circumstantial
\>250ms - substantial

most of our tools report at 250ms - "microhangs".  Often okay, but maybe not.
everything above 500ms we consider a hang.

conclusion:
Instant - <100ms
request-response - <500ms

Usually an interaction has both.  Button click - instant.  Email send -> request/response.  This was fine, because the button updated in realtime.  

For the actual UI elements in our interface, we often want the instant <100ms threshold.

# Event handling and rendering loop
At some point, someone interacts with the device.  No control.
Somebody detects this, create events, sends it to the OS.
OS forwards it to some process.
Main thread -> handles the events.  This is where most of your UI code runs.  Decisions on how to update the UI.
Sent to render server, separate process that composites UI layers.  Lastly the display driver does its thing.

see docs: improving app responsiveness

Often we are blocked on the MT either because it takes too long, or blocked on something else.

Ideally no work on MT should take longer than 100ms.

Can also cause hitches.  Lower thresholds apply.

[[UI animation hitches]]
see docs: app responsiveness.

# Busy main thread hang

Instruments, time profiler.

## two types of hangs
* unresponsive main thread
	* busy main thread -> CPU activity
	* blocked main thread -> little/no CPU activity.

Separate track for each thread.
Profile view - all functiosn which execute on the MT.
Option -> set inspection range and zoom.  

hide system libraries.  

Two cases of busy main thread, two cases:
* too long?
	* look at callees, further down
* too often?
	* Look at callers, further up.
Time profiler cannot tell us which case we are.


## Time profiler only samples
Cannot differentiate between long-running and often-called functions
use `os_signpost` or other instruments to measure precise runtime or how often specific work is performed.

[[getting started with instruments]]

One of which is the swiftui view body instrument.  List of all instruments your app ahs to offer.  There are a lot, you can write your own, etc.

swiftUI -> view body.

body should be called less often.  Have to look at callers to understand how to reduce it.  Main therad track again, secondary click, and hit view in xcode.  

Find->find selected symbol in workspace.  

### 19:38 - BackgroundThumbnailView
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    
    var body: some View {
        Image(uiImage: background.thumbnail)
    }
}
```

### 19:58 - BackgroundSelectionView with Grid
```swift
var body: some View {
    ScrollView {
        Grid {
            ForEach(backgroundsGrid) { row in
                GridRow {
                    ForEach(row.items) { background in
                        BackgroundThumbnailView(background: background)
                            .onTapGesture {
                                selectedBackground = background
                            }
                    }
                }
            }
        }
    }
}
```

Instead, consider LazyVGrid. Only includes as many views as necessary.  consider using lazy variants.

### 20:03 - BackgroundSelectionView with Grid (simplified)
```swift
var body: some View {
    ScrollView {
        Grid {
            ForEach(backgroundsGrid) { row in
                GridRow {
                    ForEach(row.items) { background in
                        BackgroundThumbnailView(background: background)
                    }
                }
            }
        }
    }
}
```

### 20:26 - LazyVGrid variant
```swift
var body: some View {
    ScrollView {
        LazyVGrid(columns: [.init(.adaptive(minimum: BackgroundThumbnailView.thumbnailSize.width))]) {
            ForEach(BackyardBackground.allBackgrounds) { background in
                BackgroundThumbnailView(background: background)
            }
        }
    }
}
```

eager variants use much less memory when you need to render the same contents.  So they're the default, switch to lazy when you find issues.

[[Stacks, Grids, and Outlines in SwiftUI]]

Note that it's still slow on iPad, it has a larger screen.  A microhang on your desk might be a major hang for your users under different conditions.

We would like to do the thumbnail preparation in the background here.  "Open in source viewer".  Annotates the lines of the implementation with the samples to see where our code spent how much time.

### 24:05 - BackgroundThumbnailView
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    
    var body: some View {
        Image(uiImage: background.thumbnail)
    }
}
```

### 24:59 - BackgroundThumbnailView with progress (but without loading)
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    @State private var image: UIImage?
    
    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .frame(width: Self.thumbnailSize.width, height: Self.thumbnailSize.height, alignment: .center)
        }
    }
}
```

on appear, swiftui will start a task for us.
### 25:26 - BackgroundThumbnailView with async loading on main thread
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    @State private var image: UIImage?
    
    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .frame(width: Self.thumbnailSize.width, height: Self.thumbnailSize.height, alignment: .center)
                .task {
                    image = background.thumbnail
                }
        }
    }
}
```

what happens here is that the hang happens slightly later.

types of hang: interaction

synchronous/direct hang: event takes a long time to process.
asynchronous/delayed: we delayed work to happen later.  Then an event comes in during that work, and that event has to wait for previous work to be done.

the way our thing works is we look at all work items to see if they're >100ms.  We mark those as hangs.  Because user input could happen at any time, you might have an actual time.  hang detection detects these delayed cases, but it only detects potential hang, not the experienced hang.

SwiftUI creates 40 views with progress indicators.  35 are only displayed, for those 35 we stat loading the image, view updates, and body is called again, so 75 body executions.  



### 29:59 - BackgroundThumbnailView with async loading on main thread
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    @State private var image: UIImage?
    
    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .frame(width: Self.thumbnailSize.width, height: Self.thumbnailSize.height, alignment: .center)
                .task {
                    image = background.thumbnail
                }
        }
    }
}
```

### 31:41 - BackgroundThumbnailView with async loading on main thread (simplified)
```swift
struct BackgroundThumbnailView: View {
    // [...]
    
    var body: some View {
        // [...]
        ProgressView()
            .task {
                image = background.thumbnail
            }
        // [...]
    }
}
```

# Async hang
try the swift concurrency tasks instrument.

Task lane shows task execution on the main thread.  Setting inspection range confirms that this task is doing the computation work.

1.  `body` inherits `MainActor` from SwiftUI's `View`
2. `task` inherits actor isolation from surrounding context, so it is also `@MainActor`
3. `thumbnail` is synchronous, so it executes on the main actor.

options:
* await a non-main-actor-isolated async function.  Many cases whre this is not feasible.
* `Task.detached` to remove from surrounding tasks.  Creating a separate task is more expensive than suspending an existing one.
* Note that cancellation does not propagate to unstructured task like detached

[[Visualize and optimize Swift concurrency]]
docs: improving app responsiveness

### 33:40 - BackgroundThumbnailView with async loading on main thread
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    @State private var image: UIImage?
    
    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .frame(width: Self.thumbnailSize.width, height: Self.thumbnailSize.height, alignment: .center)
                .task {
                    image = background.thumbnail
                }
        }
    }
}
```

### 33:59 - synchronous thumbnail property
```swift
public var thumbnail: UIImage {
    get {
        // compute and cache thumbnail
    }
}
```

### 34:03 - asynchronous thumbnail property
```swift
public var thumbnail: UIImage {
    get async {
        // compute and cache thumbnail
    }
}
```



### 34:08 - BackgroundThumbnailView with async loading in background
```swift
struct BackgroundThumbnailView: View {
    static let thumbnailSize = CGSize(width:128, height:128)
    
    var background: BackyardBackground
    @State private var image: UIImage?
    
    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .frame(width: Self.thumbnailSize.width, height: Self.thumbnailSize.height, alignment: .center)
                .task {
                    image = await background.thumbnail
                }
        }
    }
}
```

# Blocked main thread hang

need other instruments to analyze.

Traceback from another hang.  Long one, several seconds.  Main thread track.

Time profiler doesn't collect samples here?  So we need thread states tool.

we see the backtrace of the syscall shown in the narrative.  Allocating an MLModel which in turn happened due to colorizingservice, , closure in the body getter, etc.



### 38:52 - shared property causes blocked main thread
```swift
var body: some View {
    mainContent
        .task(id: imageMode) {
            defer {
                loading = false
            }
            do {
                var image = await background.thumbnail
                if imageMode == .colorized {
                    let colorizer = ColorizingService.shared
                    image = try await colorizer.colorize(image)
                }
                self.image = image
            } catch {
                self.error = error
            }
        }
}
```
 note inserting an await here is deceptive.  `shared` property is not async, and that is the thing that is blocking here.

### 39:00 - shared property causes blocked main thread (simplified)
```swift
struct ImageTile: View {
    // [...]
    
    // implicit @MainActor
    var body: some View {
        mainContent
            .task() { // inherits @MainActor isolation
                // [...]
                let colorizer = ColorizingService.shared
                result = try await colorizer.colorize(image)
            }
    }
}
```

### 39:10 - shared property causes blocked main thread + ColorizingService (simplified)
```swift
class ColorizingService {
    static let shared = ColorizingService()
   

    // [...]
}

struct ImageTile: View {
    // [...]
    
    // implicit @MainActor
    var body: some View {
        mainContent
            .task() { // inherits @MainActor isolation
                // [...]
                let colorizer = ColorizingService.shared
                result = try await colorizer.colorize(image)
            }
    }
}
```

### 39:25 - shared synchronous property after await keyword still causes blocked main thread
```swift
class ColorizingService {
    static let shared = ColorizingService()
   

    // [...]
}

struct ImageTile: View {
    // [...]
    
    // implicit @MainActor
    var body: some View {
        mainContent
            .task() { // inherits @MainActor isolation
                // [...]
                result = try await ColorizingService.shared.colorize(image)
            }
    }
}
```

### 39:39 - shared synchronous property after await keyword still causes blocked main thread (+colorize function)
```swift
class ColorizingService {
    static let shared = ColorizingService()
   
    func colorize(_ grayscaleImage: CGImage) async throws -> CGImage
    // [...]
}

struct ImageTile: View {
    // [...]
    
    // implicit @MainActor
    var body: some View {
        mainContent
            .task() { // inherits @MainActor isolation
                // [...]
                result = try await ColorizingService.shared.colorize(image)
            }
    }
}
```

Fix
* make `shared` property `async`
* be careful with locks/semaphores in combination with swift concurrency

[[Swift concurrency Behind the scenes]]

sometimes, the main thread is just asleep and waiting for input.  But it is shown as blocked.  Not a hang through.  So to determine whether a blocked thread is a response to inputs, or not, look to hangs instrument, not to threads instrument.

Note that
* blocked thread does not imply unresponsive main thread
* high cpu usage does not imply unresponsive main thread
* unresponsive main thread implies either blocked main thread or busy main thread
* our hangs instrument figures this out

# Wrap up
* Keep main thread <100ms
* Determine whether the main thread is busy or blocked
* Hangs can surface synchronously or asynchronously
* do less work
* move work to the background
* Measure first, then optimize
* 

# Resources
https://developer.apple.com/documentation/Xcode/improving-app-responsiveness
