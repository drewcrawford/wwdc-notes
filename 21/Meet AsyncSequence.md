#async #swift 
# What is AsyncSequence?
```swift
@main
struct QuakesTool {
    static func main() async throws {
        let endpointURL = URL(string: "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_month.csv")!

        // skip the header line and iterate each one 
        // to extract the magnitude, time, latitude and longitude
        for try await event in endpointURL.lines.dropFirst() {
            let values = event.split(separator: ",")
            let time = values[0]
            let latitude = values[1]
            let longitude = values[2]
            let magnitude = values[4]
            print("Magnitude \(magnitude) on \(time) at \(latitude) \(longitude)")
        }
    }
}
```

Lets you write concurrent code without the need for callbacks using `await` keyword
Suspend and resume

AsyncSequence suspends on each element and suspends when the iterator produces a value

* Like `Sequence` except `async`.
* May throw
* Terminates at end or an error

## Regular iteration
```swift
for quake in quakes {
    if quake.magnitude > 3 {
        displaySignificantEarthquake(quake)
    }
}
```

becomes
```swift
var iterator = quakes.makeIterator()
while let quake = iterator.next() {
    if quake.magnitude > 3 {
        displaySignificantEarthquake(quake)
    }
}
```

## async version
```swift
var iterator = quakes.makeAsyncIterator()
while let quake = await iterator.next() {
    if quake.magnitude > 3 {
        displaySignificantEarthquake(quake)
    }
}
```
e.g.
```swift
for await quake in quakes {
    if quake.magnitude > 3 {
        displaySignificantEarthquake(quake)
    }
}
```


# Usage and APIs
Given a source, you can await with `for await i in list`.

Breaking is a good way to terminate iteration early from inside the loop.  Can also use `continue`.

Keep in mind that it might throw.
```swift
do {
    for try await quake in quakeDownload {
        ...
    }
} catch {
    ...
}
```

Just like throwing functions, `try` is required to process each element.  Also the compiler will detect when you're missing `try`.  When an `AsyncSequence` can produce errors, compiler knows.

Running code sequentially isn't always what's desired.  Maybe you want to run the iteration concurrently?  Consider creating an `async` task.

This is useful when the AsyncSequence might be indefinite.  
Also helpful when cancelling iteration externally:
```swift
let iteration1 = async {
    for await quake in quakes {
        ...
    }
}

let iteration2 = async {
    do {
        for try await quake in quakeDownload {
            ...
        }
    } catch {
        ...
    }
}

//... later on  
iteration1.cancel()
iteration2.cancel()
```

## Bytes from a `FileHandle`

```swift
for try await line in FileHandle.standardInput.bytes.lines {
    ...
}
```

## Read lines from a URL

```swift
let url = URL(fileURLWithPath: "/tmp/somefile.txt")
for try await line in url.lines {
    ...
}
```

Sometimes, getting things from the network requires more control.  

```swift
let (bytes, response) = try await URLSession.shared.bytes(from: url)

guard let httpResponse = response as? HTTPURLResponse,
      httpResponse.statusCode == 200 /* OK */
else {
    throw MyNetworkingError.invalidServerResponse
}

for try await byte in bytes {
    ...
}
```

Can now fetch via URLRequest.  [[Use asyncawait with URLSession]]

## Notifications

```swift
let center = NotificationCenter.default
let notification = await center.notifications(named: .NSPersistentStoreRemoteChange).first {
    $0.userInfo[NSStoreUUIDKey] == storeUUID
}
```

Using methods like `first(where:)` allows for new design patterns.

New APIs for async manipulating values.   `map`, `allSatisfy`, etc.  Anything you can think of for `Sequence`.


# Adopting AsyncSequence

There are a few ways of implementing a sequence.  But how to adapt your existing code?

* Callbacks
* Some delegates

Pretty much anything that provides a new value, and is not accepting a response back, can be a prime candidate for AsyncSequence.

```
// Support for AsyncStream will arrive in a later seed of Xcode 13.
//specify the element type
let quakes = AsyncStream(Quake.self) { continuation in
	//monitor can be created in construction closure
    let monitor = QuakeMonitor()
	monitor.quakeHandler = { quake in
	    continuation.yield(quake)
	}
	continuation.onTermination = { _ in
		monitor.stopMonitoring()
	}
	monitor.startMonitoring()
}
```

Now encapsulate the same monitor code in a new construction.  Then you can use `for await in`.

Focus on the intent of your code instead of worrying about replicating the bookkeeping.

Handles buffering
Handles cancellation

Solid way of building your own Async Sequences and is a suitable return type from your APIs.


If you need to represent errors, use `AsyncThrowingStream` which also handles errors.

[[Meet asyncawait in Swift]]
[[Explore Structured Concurrency in Swift]]

