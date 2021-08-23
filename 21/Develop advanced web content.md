#webkit #safari 

# Javascript enhancements
* Class field syntax
* Weak references
* Top-level await
* Module in workers
* Internationalization improvements

Stopwatch example.  One button, begins counting.  Click again, it will stop and give you the duration passed.

## Class field syntax
* Private instance fields and methods
* Language-enforced access protection
* Public and private static fields

`#startTime` can be private to the class.

## Weak references
* Weakly refer to an object without preventing garbage collection
* Get underlying object from weak reference if it exists
* Register notification about garbage collection of object

`new WeakRef(item)`.  

## FinalizationRegistery
Perform action on garbage collection of some object.  

**Be careful when using weak references**
1.  Lifetime may be extended
2.  Finalizer may run on eventloop

## Top-level await
* available in modules
* Use `await` keyword out of async function at the top level
* Async modules block execution of modules importing them

`import` function returns a promise, so can use `.then(())`

Now you can use `await import` instead.

If stopwatch module runs async operations and waits for results, variable will be initialized after execution.

## Modules in workers
Efficient resource utilization
Ergonomic and performance benefits of modules
Easy to move heavy work to background thread

Use modules in ServiceWorker
and in Worklet

## Internationalization improvements

### Number format
language, options
`Intl.Numberformat(language,options)`

Many options to create formats you need, such as style, unit, etc.

## DateTimeFormat
similar

## Segmenter
Segmented.  Enables language string splitting, etc.  Use it to find keywords in an event sentence.

## ListFormat
## DisplayNames

# WebAssembly updates
* Bulk memory operations
* Non-trapping float-int conversions
* Sign-extenson operators
* JavaScript BigInt integration
* Reference types
* Streaming download and compilation

# New web APIs
* WebGL 2.0
* Storage access
* WebM
* VP9
* Media Recorder
* Audio Worklet
* Web Share
* Media Session
* Speech Recognition

## WebGL 2
* 3D textures
* Sampler objects
* Transform feedback

**all Apple devices**

Because WebGL is a low-level API it can be very verbose.  But there are many great libraries and frameworks
* A-frame
* babylon.js
* playcanvas
* three.js

Now use **metal** backend.

xcode frame debugger, etc.

## Video
Sometimes it might be tricky for you to decide which format to use.  

### WebM
* streaming only
* Available on macOS and iPadOS

Check availability with MediaCapabilities API

### VP9
Also supported now.  We expect more web content to be available in safari adn webkit apps
* streaming and WebRTC
* macOS and iPadOS
* all apple silicon macs
* check availability with MediaCapabilities API

**We would recommend H264 or HEVC**.

Both come with hardware acceleration which can provide smoother playback and longer battery life.

## storage access
For securit yreasons, third-party domains don't have access to cookies by default.  This can be a problem when iframes only want to serve content to authenticated users.

Now you can request permissions to access first-party cookies.  If user grants permission, third-party will be able to access these cookies.

Available for over 3 years.  To improve interop we're adding 2 new features

* Per-page access scope.  Once permission is granted, it's extended to all its subresources on the same page.  Don't have to make a request for each iframe.
* Nested iframe request.

"Updates to the Storage Access API" at webkit.org

## Media Recorder
* Record audio or video from media elements
* Configure for input or output types
* Easy to use with single major interface

Just like that, create a voice recorder.  

## Audio worklet
* Part of Web Audio API
* Supply scripts to process audio on the rendering thread
* Low latency audio processing

## Web share
* Share links and text on the web to a destination of a user's choice
* Platform-specific sharing UI
* Support for sharing files: iamge, video, audio, text files, etc

## Speech recognition
Capture live audio and transcribe it to text with probabilities
Same speech engine as Siri (require to enable Siri or Dictation)
Privacy prompting for server-based recognition

Usage is a bit like MediaRecorder.  Must create `new webkitSpeechRecognition()`

* `continuous`
* `onresult`
* `onend`
* `.start()`
* `.stop()`

Alternatives sorted based on probabilities.

Add recognition capability to your web content.

Been a long journey and one less web API worth mentioning.

New web API can help you improve this situation.

## Media session
* Communicate media states between web page and platform components
* Control media without returning to browser or web page

[[Coordinate media playback on the web with GroupActivities]]

# Wrap up
* Try out the new features
* File bugs at bugs.webkit.org
* Use Safari Technology Preview
* Read the WebKit blog at webkit.org
* Follow @webkit on twitter

* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://webkit.org
* https://webkit.org/blog/8124/introducing-storage-access-api/

