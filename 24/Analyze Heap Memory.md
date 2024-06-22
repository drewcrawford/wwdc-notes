

Dive into the basis for your app's dynamic memory: the heap! Explore how to use Instruments and Xcode to measure, analyze, and fix common heap issues. We'll also cover some techniques and best practices for diagnosing transient growth, persistent growth, and leaks in your app.


Memory allocated by malloc, directly or indirectly
Memory with the most developer control
Almost always dirty, counts against memory limit

[[Profile and optimize your game's memory]]
[[Detect and diagnose memory issues]]
[[iOS memory deep dive - 18]]

# Measurement

What is heap memory?  

* app exectuable
* heap
* stack
* resources
* gpu
* linked libraries, etc.

Virtual memory regions.  Each region is broken up into allocations.
16kb pages.  By allocations can be bigger or smaller
* clean - haven't written to
* dirty - written to recently
* swapped
only dirty/swapped.  In most applications, the heap is responsible for footprint.

malloc gives you
* lifetime beyond current scope until free
* 16-byte minimum and alignment
* zeroes smaller blocks on free
* debugging features (MallocStackLogging)

Xcode Memory Report.  Shows footprint over time.More than just its heap.
Memory Graph Debugger.  Focus on a specific allocation.
Powerful CLI tools for memory analysis.  Leaks, heap, vmmap.
Instruments.  Allocation.  Leaks.  

Demo.


# Transient growth

Effects of transient memory:
* sudden memory pressure
* out of memory crashes
* fragmentation

Finding: 
* created and still living (single allocation)
* created and destroyed (aggregate)

###  ThumbnailLoader.makeThumbnail(from:) implementation - 10:01
```swift
func makeThumbnail(from photoURL: URL) -> PhotoThumbnail {
  validate(url: photoURL)
  var coreImage = CIImage(contentsOf: photoURL)!

  let sepiaTone = CIFilter.sepiaTone()
  sepiaTone.inputImage = coreImage
  sepiaTone.intensity = 0.4
  coreImage = sepiaTone.outputImage!

  let squareSize = min(coreImage.extent.width, coreImage.extent.height)
  coreImage = coreImage.cropped(to: CGRect(x: 0, y: 0, width: squareSize, height: squareSize))

  let targetSize = CGSize(width:64, height:64)
  let scalingFilter = CIFilter.lanczosScaleTransform()

  scalingFilter.inputImage = coreImage
  scalingFilter.scale = Float(targetSize.height / coreImage.extent.height)
  scalingFilter.aspectRatio = Float(Double(coreImage.extent.width) / Double(coreImage.extent.height))
  coreImage = scalingFilter.outputImage!

  let imageData = context.generateImageData(of: coreImage)

  return PhotoThumbnail(size: targetSize, data: imageData, url: photoURL)
}
```

###  ThumbnailLoader.loadThumbnails(with:), with autorelease pool growth issues - 10:23
```swift
func loadThumbnails(with renderer: ThumbnailRenderer) {
  for photoURL in urls {
    renderer.faultThumbnail(from: photoURL)
  }
}
```

###  Simple autorelease example - 10:33
```swift
print("Now is \(Date.now)") // Produces autoreleased .description String
```
Even with ARC, objects can sit in autorelease pools
Swift can trigger autoreleasing code when calling into frameworks

###  Autorelease pool growth in loop - 11:08
```swift
autoreleasepool {
  // ...
  
  for _ in 1...1000 {
    // Autoreleases into single pool, causing growth as loop runs
    print("Now is \(Date.now)")
  }
  
  // ...
}
```

###  Autorelease pool growth in loop, managed by nested pool - 11:50
```swift
autoreleasepool {
  // ...
  
  for _ in 1...1000 {
    autoreleasepool {
      // Autoreleases into nested pool, preventing outer pool from bloating
      print("Now is \(Date.now)")
    }
  }
  
  // ...
}
```

###  ThumbnailLoader.loadThumbnails(with:), with nested autorelease pool growth issues fixed - 12:16
```swift
func loadThumbnails(with renderer: ThumbnailRenderer) {
    for photoURL in urls {
        autoreleasepool {
            renderer.faultThumbnail(from: photoURL)
        }
    }
}
```

Memory profiling is pretty accurate in simulator.

# Persistent growth
Memory that doesn't get deallocated, stairstep  pattern.

generation A -> before first marker.

We can put address from instruments into memory graph debugger, I guess you can open the graph debugger into instruments?

Four main types of references
* strong -> definitely a pointer, explicit ownership
* Weak/unowned -> definitely a pointer, explicit non-ownership
* Unmanaged -> probably a pointer, manually managed
* Conservative -> might be a pointer, tools must assume it is

When the tools scan your process heap, they use best type info available for each allocation.

For C/C++ there is no reference ownership, so you only see conservative references.  We look up type names for virtual methods.



###  C++ class with virtual method - 17:27
```cpp
class Coconut {
  Swallow *swallow;
  virtual void virtualMethod() {}
};
```
For types without virtual methods, stack traces can help provide names.  
###  C++ class without virtual method - 17:40
```cpp
class Coconut {
  Swallow *swallow;
};
```



###  ThumbnailRenderer.faultThumbnail(from:), caching thumbnails incorrectly - 18:41
```swift
func faultThumbnail(from photoURL: URL) {
  // Cache the thumbnail based on url + creationDate
  let timestamp = UInt64(Date.now.timeIntervalSince1970) // Bad - caching with wrong timestamp
  let cacheKey = CacheKey(url: photoURL, timestamp: timestamp)

  let thumbnail = cacheProvider.thumbnail(for: cacheKey) {
    return makeThumbnail(from: photoURL)
  }
  images.append(thumbnail.image)
}
```

###  ThumbnailRenderer.faultThumbnail(from:), caching thumbnails correctly - 19:28
```swift
func faultThumbnail(from photoURL: URL) {
  // Cache the thumbnail based on url + creationDate
  let timestamp = cacheKeyTimestamp(for: photoURL) // Fixed - caching with correct timestamp
  let cacheKey = CacheKey(url: photoURL, timestamp: timestamp)

  let thumbnail = cacheProvider.thumbnail(for: cacheKey) {
    return makeThumbnail(from: photoURL)
  }
  images.append(thumbnail.image)
}
```

# Leaked memory
Reachability - finding paths from roots to heap blocks
Useful memory: reachable allocations that will be used again
Abandoned memory: reachable allocations that will not ever be used again

Unreachable allocations that have no owning references to free.

Find and fix one reference in a cycle.  Memory graph debugger shows us leaks as warnings.

Closure context are paired 1:1 with active closures
Reference qualifiers, no capture names

Why isn't a specific allocation shown as leaked?
Leak scanning is conservative, meaning it allows uncertain references

Tools have to allow things that look like they might be pointers, but maybe aren't.  We look for pointers byte-by-byte.  We check references against list of allocations.

If they match, we report an uncertain reference to the block.  A real leak can be missed due to conservative references.  If I'm creating an intentional leak, consider running in a loop 100 times.

How can the number of leaks go down over time?  Heap is noisy and random.  Noise makes conservative references nondeterministic.  Tools might find five 'pointers' but only see four later.

Why is noreturn function leaking?  Calls to noreturn or returning `Never` don't cleanup local references.

If they're used to park a thread forever... they can leak.  Consider explicitly storing the object into a global.  By storing the reference outside the scope, it's stored somewhere tools can see. And since tools can see it, object is considered reachable instead of leaked, even though local variables aren't preserved by compiler.

# Performance
###  Code creating reference cycle with closure context - 22:19
```swift
let swallow = Swallow()
swallow.completion = {
  print("\(swallow) finished carrying a coconut")
}
```

weak vs unowned.  

Weak references are always valid to use, always an optional, nil when destination deinitialized.  

To implement the weak reference, swif tallocates a 'swift weak reference storage' the first time it's referenced.  This sits between and lazily nils out.

Unlike weak references, unowned directly hold destinations.  No extra memory, less time to access.  Can also be nonoptional or constant.

If it goes away, it will be deinitialized *but not deallocated*.  However, as long as the unowned reference exists, we can't deallocate the memory.  Therefore if you don't know how long things live, weak overhead may be worth it.

If you're not seeing weak/unowned references, may need to check 'reflection metadata' build setting.  All -> types, layouts, and property names
without names -> types and layouts

Be careful when using methods as closures.


###  PhotosView image loading code, with leak - 23:11
```swift
// ...
let renderer = ThumbnailRenderer(style: .vibrant)
let loader = ThumbnailLoader(bundle: .main, completionQueue: .main)
loader.completionHandler = {
  self.thumbnails = renderer.images // implicit strong capture of renderer causes strong reference cycle
}
loader.beginLoading(with: renderer)
// ...
```

###  PhotosView image loading code, with leak fixed - 23:40
```swift
// ...
let renderer = ThumbnailRenderer(style: .vibrant)
let loader = ThumbnailLoader(bundle: .main, completionQueue: .main)
loader.completionHandler = { [weak renderer] in
	guard let renderer else { return }

  self.thumbnails = renderer.images
}
loader.beginLoading(with: renderer)
// ...
```

###  Intentional leak of manually-managed allocation - 24:24
```swift
let oops = UnsafeMutablePointer<Int>.allocate(capacity: 16)
// intentional mistake: missing `oops.deallocate()`
```

###  Loop over intentional leak of manually-managed allocations - 25:12
```swift
for _ in 0..<100 {
  let oops = UnsafeMutablePointer<Int>.allocate(capacity: 16)
  // intentional mistake: missing `oops.deallocate()`
}
```

###  Nonreturning function which can see leaks of allocations owned by local variables - 26:11
```swift
func beginServer() {
  let singleton = Server(delegate: self)
  dispatchMain() // __attribute__((noreturn))
}
```

###  Fix for reported leak in nonreturning function - 26:22
```swift
static var singleton: Server?

func beginServer() {
  Self.singleton = Server(delegate: self)
  dispatchMain()
}
```

###  Weak reference example - 27:21
```swift
weak var holder: Swallow?
```

###  Unowned reference example - 27:43
```swift
unowned let holder: Swallow
```

###  Implicit use of self by method causes reference cycle - 29:07
```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = defaultAction // Implicitly uses `self`
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

###  Break reference cycle cause day implicit use of self by method, using weak - 29:25
```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = { [weak self] data in
      return self?.defaultAction(data)
    }
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

###  Break reference cycle cause day implicit use of self by method, using unowned - 29:41
```swift
class ByteProducer {
  let data: Data
  private var generator: ((Data) -> UInt8)? = nil

  init(data: Data) {
    self.data = data
    generator = { [unowned self] data in
      return self.defaultAction(data)
    }
  }

  func defaultAction(_ data: Data) -> UInt8 {
    // ...
  }
}
```

Safety and performance chart

| x                                   | weak                        | unowned                         |
| ----------------------------------- | --------------------------- | ------------------------------- |
| accessing deinitialized destination | nil                         | Crash                           |
| Memory cost                         | 32 bytes per destination    | None (when used correctly)      |
| Runtime cost                        | ~10x (swift_weakLoadStrong) | ~4x (swift_unownedRetainStrong) |
see docs.

Don't circumvent ARC.  
Ensure -whole-module-optimization is enabled. This may allow more inlining.
Profile and look for generics that may need specialization.

Few any boxes, retainable references in Swift struct.
###  Struct with non-trivial init/copy/deinit - 31:14
```swift
struct Nontrivial {
  var number: Int64
  var simple: CGPoint?
  var complex: String // Copy-on-write, requires non-trivial struct init/copy/destroy
}
```
[[Explore Swift performance]]
[[Consume noncopyable types in Swift]]

ObjC retain and release:
* don't circumvent ARC (`-fno-objc-arc`
* Inlining opportunities `__attribute__((objc_direct))`
* Parameters `__attribute__((objc_externally_retained))`

Cost of measurement.
malloc stack logging, allocations: live recording, performance overhead
leaks, virtual memory tracker, memory graph debugger: snapshot-based, hang

# Wrap up
* profile with instruments to find transient and persistent growth
* Use xcode's memory graph debugger to find why objects are still alive
* Be proactive!
* 
# Resources
* [Forum: Developer Tools & Services](https://developer.apple.com/forums/topics/developer-tools-and-services?cid=vf-a-0010)
* [The Swift Programming Language: Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10173/4/5ADD00F7-AAD5-4C66-A3ED-9FC7E27C7720/downloads/wwdc2024-10173_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10173/4/5ADD00F7-AAD5-4C66-A3ED-9FC7E27C7720/downloads/wwdc2024-10173_sd.mp4?dl=1)