Discuss ways to better understand your swift concurrency code and make it go faster
# Swift concurrency recap
* async/await
* tasks
	* basic unit of work in concurrent code, manage staet and associated data.
* structured concurrency
	* easy to spawn tasks to run in parallel and wait for them to complete.  Language provides syntax for joins, etc.
* actors
	* coordinate multiple tasks that need to access shared data.  Isolate data from theoutside, only 1 task at a time can manipulate internal state.

New instrument to hepl you understand what your app is doing, locate problems and improve performance.

see related videos

# Concurrency optimization
Write correct, concurrent, and parallel code.  Still possible to write code that misuses concurrency concepts.  Also possible to use correctly but slowly.

* Main actor blocking
* actor contention
* thread poole exhaustion
* continuation misuse

## main actor blocking
long task runs on main actor.  Main actor is a speicalactor which executes on the main thread, UI work must be done on the main thread, and main actor lets you integrate UI work into swift concurrency.app locks up and becomes non-responsive.

main actor must finish quickly and either finish work or move off.  Put in a normal actor or a detached task.  Small units of work can be executed on the main actor or perform other tasks.

### ex
UI hung due to blocked by main thread.

Use the swift concurrency template in instruments.  Take a look at top levle statistics provided by swift tasks instrument.

Running tasks, alive tasks, etc.  Total tasks (all time).  For memory, take a close look at alive and total.  Combination give you a good indication of how resources are used.

**Task forest**.  Graphical represnentation of parent-child relationship between tasks in structured concurrency code.
Tasks summary view.  Shows how much time each task spent in different state.
Right click on a task and pin a task to timeine.  Quickly find and learn about tasks that may be running for a very long time or stuck waiting to get access to an actor.
In timeline:
* track shows you what state it's in
* creation backtrace in right panel
* narrative, e.g. what task are we waiting on
* can pin from narriative view.  So pin child task, a thread, or even a swift actor tot he timeline.
* instrumental in finding out how a task is related to everything else.

### demo
cmd-I.  From here, you can pick swift ocncurrency and start recording.  Reproduce.
Investigate trace.  Fullscreen to see all info.  Use option-drag to zoom in on area.

In process track, instruments shows us where this UI hang is occurring.  Used in cases where it's unclear when the hang occurs or how long it lasted.

Top-level statists.  For most of the time, only 1 task is running.  All of our work is forced to serialize.

narrative tells us it ran for a short amount of time, and then ran on the main thread for a long time.  We can pin the main thread to the timeline.

Being blocked by several long-running tasks.  
1.  What is this doing
2. where did it come from?

Right click on symbol to open in source.  

Now taht we know where the task was created and what it is doing, adapt our code.

Compressed file function is located within the compression state class.  Entire compression state is annotated to run on `@MainActor`.  That's why the task ran on the main thread.

We need the class to be on the main actor because `@Published` can only be updated on the MT.  So instead we could try to convert it into an actor.  However, the compiler will tell us we can't do this, because the shared mutable state needs to bep rotected with 2 actors.

## Islands in a sea of concurrency

* files isolated to main actor
* logs => concurrent access

```swift
@MainActor
class CompressionState: ObservableObject {
    @Published var files: [FileStatus] = []
    var logs: [String] = []
    
    func update(url: URL, progress: Double) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].progress = progress
        }
    }
    
    func update(url: URL, uncompressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].uncompressedSize = uncompressedSize
        }
    }
    
    func update(url: URL, compressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].compressedSize = compressedSize
        }
    }
    
    func compressAllFiles() {
        for file in files {
            Task {
                let compressedData = compressFile(url: file.url)
                await save(compressedData, to: file.url)
            }
        }
    }
    
    func compressFile(url: URL) -> Data {
        log(update: "Starting for \(url)")
        let compressedData = CompressionUtils.compressDataInFile(at: url) { uncompressedSize in
            update(url: url, uncompressedSize: uncompressedSize)
        } progressNotification: { progress in
            update(url: url, progress: progress)
            log(update: "Progress for \(url): \(progress)")
        } finalNotificaton: { compressedSize in
            update(url: url, compressedSize: compressedSize)
        }
        log(update: "Ending for \(url)")
        return compressedData
    }
    
    func log(update: String) {
        logs.append(update)
    }
```

Need to move `logs` to its own actor.  

```swift
actor ParallelCompressor {
    var logs: [String] = []
    unowned let status: CompressionState
    
    init(status: CompressionState) {
        self.status = status
    }
    
    func compressFile(url: URL) -> Data {
        log(update: "Starting for \(url)")
        let compressedData = CompressionUtils.compressDataInFile(at: url) { uncompressedSize in
            Task { @MainActor in
                status.update(url: url, uncompressedSize: uncompressedSize)
            }
        } progressNotification: { progress in
            Task { @MainActor in
                status.update(url: url, progress: progress)
                await log(update: "Progress for \(url): \(progress)")
            }
        } finalNotificaton: { compressedSize in
            Task { @MainActor in
                status.update(url: url, compressedSize: compressedSize)
            }
        }
        log(update: "Ending for \(url)")
        return compressedData
    }
    
    func log(update: String) {
        logs.append(update)
    }
}

@MainActor
class CompressionState: ObservableObject {
    @Published var files: [FileStatus] = []
    var compressor: ParallelCompressor!
    
    init() {
        self.compressor = ParallelCompressor(status: self)
    }
    
    func update(url: URL, progress: Double) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].progress = progress
        }
    }
    
    func update(url: URL, uncompressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].uncompressedSize = uncompressedSize
        }
    }
    
    func update(url: URL, compressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].compressedSize = compressedSize
        }
    }
    
    func compressAllFiles() {
        for file in files {
            Task {
                let compressedData = await compressor.compressFile(url: file.url)
                await save(compressedData, to: file.url)
            }
        }
    }
}
```

## Actor contention
make it safe for multiple tasks to manipulate shared state, by serializing access.  Only one task at a time is allowed to occupy the actor.  Other tasks will wait.

Swift concurrency allows for parallel computation using unstructed tasks, task groups, and async let.  Ideally, many cores simulateneously.  beware of performing too much work on shared actors.  When multiple tasks use the same actor simultaneously we serialize those tasks.

Lose the performance benefit of serialized computation.  Each task must wait for the actor to become available.  Tasks only run on the actor when they realyl need exclusive access to the state.

Divide the task into chunks.  Soem chunks run ont he actor, some don't.  Non-siolated chunks can execute in parallel.

Our concurrency code is spending an alarming amount of time in the enqueued state.  Tasks waiting to get exclusive access to an actor.  Pull function out of actor isolation and into a detached task.

Move onto the main actor to access files.  Thenm moves into the sea of concurrency.  Until it needs logs.  Moves onto that actor.  Then leaves the actor.

We don't have just 1 task doing compressor work,w e ahve many.  And by not being constrained to an actor, they are executed concurrently.

Each actor can only execute 1 task at a time.  But most tasks don't need to be on an actor.  So... allows our copmression tasks to be executed in parallel and utilize all available CPU cores.

```swift
actor ParallelCompressor {
    var logs: [String] = []
    unowned let status: CompressionState
    
    init(status: CompressionState) {
        self.status = status
    }
    
    nonisolated func compressFile(url: URL) async -> Data {
        await log(update: "Starting for \(url)")
        let compressedData = CompressionUtils.compressDataInFile(at: url) { uncompressedSize in
            Task { @MainActor in
                status.update(url: url, uncompressedSize: uncompressedSize)
            }
        } progressNotification: { progress in
            Task { @MainActor in
                status.update(url: url, progress: progress)
                await log(update: "Progress for \(url): \(progress)")
            }
        } finalNotificaton: { compressedSize in
            Task { @MainActor in
                status.update(url: url, compressedSize: compressedSize)
            }
        }
        await log(update: "Ending for \(url)")
        return compressedData
    }
    
    func log(update: String) {
        logs.append(update)
    }
}

@MainActor
class CompressionState: ObservableObject {
    @Published var files: [FileStatus] = []
    var compressor: ParallelCompressor!
    
    init() {
        self.compressor = ParallelCompressor(status: self)
    }
    
    func update(url: URL, progress: Double) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].progress = progress
        }
    }
    
    func update(url: URL, uncompressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].uncompressedSize = uncompressedSize
        }
    }
    
    func update(url: URL, compressedSize: Int) {
        if let loc = files.firstIndex(where: {$0.url == url}) {
            files[loc].compressedSize = compressedSize
        }
    }
    
    func compressAllFiles() {
        for file in files {
            Task.detached {
                let compressedData = await self.compressor.compressFile(url: file.url)
                await save(compressedData, to: file.url)
            }
        }
    }
}
```

1.  nonisolated
2. promote to async
3. mark accesses with `await`
4. createa  `detached` task
	5. This ensures the task does not inherit the actor context that it was created in.
	6. For detached tasks, we need to explicitly capture `self`.

## Accomplishments
* isoalted cause of UI hang
* restructured code into islands of concurrency for better parallelism

# Thread pool exhaustion
hurt performance or deadlock an app.  Requires apps to make forward progress.  When a task waits for something, it does so by suspending.  Possible for code within a task to perform a blocking call.  

This breaks the requirement for tasks to make forward progress.  When this happens, the task continues to occupy the thread, but it isn't actually using a CPU core.  Because pool is limited, concurrency is unable to fully utilize CPU cores.

In extreme cases, when the entire threadpool is occupied, and they're waiting on something that requires a new task, concurrency runtime can deadlock.

* avoid blocking calls
* avoid condition variables and semaphores
	* fine-grained short locks are acceptable if necessary
* Use async APIs for blocking operations
	* Especially file/network IO
* Make blocking calls outside Swift Concurrency

# Continuation misuse
This can be used with callback state.  From the perspective of swift ocncurrency, the task suspends and then resumes when continuation resumes.

concurrency isntrument knows about ocntinuations and will mark time accordingly.

continuation callbacks have a special requirement => called exactly **once.** No more, no less.  
In other APIs, this may not be enforced by the runtime.  Btu swift makes this a **hard** requirement.

* more than once => crash, misbehave
* less than once => leak

Important to be careful when code is more complex.  On failure, the continuation will not be resumed, and the task will be suspended forever.

* checked
	* always use!
	* unless performance is absolutely critical.
	* automatically detects misuse and flags an error.
* unsafe

# more to explore
* source and disassembly integration
* time profiler integration
* graphic visualizations of structured concurrency
* task creation calltrees
[[Eliminate data races using Swift Concurrency]]
[[Swift concurrency Behind the scenes]]


* https://developer.apple.com/documentation/Swift/concurrency
* https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html