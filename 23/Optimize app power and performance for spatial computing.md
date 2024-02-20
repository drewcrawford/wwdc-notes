Learn how you can create powerful apps and games for visionOS by optimizing for performance and efficiency. We'll cover the unique power characteristics of the platform, explore building a performance plan, and share some of the tools and strategies to test and optimize your apps.

How to optimize your app for spatial computing

# Spatial computing

Continuously updating content

Renders every frame
Computes spatial algorithms continuously
Runs multiple apps

Responsiveness, Immersion.

# Profile

You may already be familiar with metrics from other apple platforms.  People want apps that launch quickly, avoid disk work, and don't use too much battery.  Avoid terminations, inefficient memory use, etc.

Take power.  users want apps optimized for power.  Hangs are more critical on this platform.

rendering.  You may be optimizing for render performance for smooth UI.  But here render performance is essential for static content as well since the system is always rendering.


Check out [[Ultimate Application Performance Survival Guide]]

Profile during development with instruments and xcode gauges.  Once relased, gather more data with metricKit and xcode organizer

RK trace is a new instruments template.  

To learn more and epxlore this new template in action, watch [[Meet RealityKit Trace]]

Profile many scenarios.  

* proifle on device
* use different user input methods
* play audio and video
* use FaceTime
* run multiple apps



# Optimize

## Rendering
The top priorities for a great UX.

1.  App. (RK, SwiftUI, UIKit, etc.)  Updated on the MT.  Must provide updates promptly.
2. Updates sent to system render server.  Continuously running.
3. Compositor, matches display refresh rate.  
4. Display

Usually 90fps, but can be higher.  

While compositor is good, still need updates from you.  Render Server can miss frame deadline if your app takes too long.  So your app visuals are delayed.

### SwiftUI & UIKit

System renders static UI content
Even with no app updates.  This rendering work can increase from overdraw.
Translucency and overlap generate overdraw.
If the translucent content is fully opaque, we don't have to generate content behind them.

Minimize translucency and overlap with Z offsets.
Lower default window sizes.

UI redraws in the render server are usually triggered by app updates.  On this platform, triggered by dynamic content scaling.  Resolution of text or vector-based UI content changes based on where the user is looking to allow for sharper visuals.  Can also lead to
more frequent redrawing at different scales. - even with no app updates
SwiftUI and UIKit enable this by default
Apps doing custom graphics rendering can opt into this behavior.

[[Explore rendering for spatial computing]]

Reduce offscreen render passes.  (Shadows, blur, masking).
Eliminate unnecessary UI view updates.  ex, use `@Observable` in SwiftUI for more granular change tracking.

### RealityKit

optimize 3d rendering with RealityKit.

Reality Composer Pro provides good statistics.  

* simpler assets have lower statistics
	* [[Create 3D models for Quick Look spatial experiences]]

Optimize mesh rendering.  Reduce mesh parts, triangles, and vertex counts
Combine parts that share a material.
Minimize overdraw from transparency (use transparency sparingly)
Use PB materials vs 'custom' materials in RC pro.
Consider custom with an unlit surface.  Bake lighting or make visuals cheaper.

[[Explore materials in Reality Composer Pro]]
[[Explore rendering for spatial computing]]

Even more you can do to optimize your app with realitykit.  When your app updates its RK content, updates get sent to the render server which applies and renders them.  Too many updates in a short period of time can become a bottleneck for the render server.

Frequent entity creation and destruction
complex animations
Frequent SwiftUI redraws
Loading too many assets
* create entities in advance and show/hide as needed
* Flatten entity hierarchies
* Minimize updates from code-based animations
* Avoid excessive SwiftUI redraws from RK entity updates

Use asynchronous loading APIs to avoid blocking main thread
load assets in advance
reuse assets between entities
Export with reality composer pro as these are optimized for loading times and memory cost.  Texture compression for free.
Reduce asset sizes.

When your app is in a full space, it's the only thing running.  Your app can create an environment to fill the space.

* reduce GPU work per pixel
* optimize for GPU power use
* Use unlit 'custom' materials in reality composer pro
	* bake lighting, time-based animations, etc.

Can also create fully immersive experiences with Metal.

### Metal

Metal + CompositorServices bypasses the render server and go directly to compositor

[[Discover Metal for immersive apps]]

* Pace rendering for compositor render rate
* Query new input data each frame
* Query input data right before GPU encoding
* Meet render deadlines

Profile with Metal System Trace template
Heavy fragment and vertex work impacts system rendering
Reduce ALU instructions and texture accesses by shaders
Use Metal compute shaders wherever possible

[[Optimize Metal apps and games with GPU counters]]
[[Delivering optimized metal apps and games]]
[[Metal Game Performance Optimization]]

## User input

App updates are processed on the MT.  if they take too long, your app is slow and unresponsive. 

Input response time depends on display refresh rate.
At 90hz, aim to keep updates below 8ms.


Use static colliders over dyanmic colliders whenever possible
minimize overlapping interactive content


## ARKit
Always-on ARKit algorithms
Anchors contribute to computational needs
Consider if anchors need to be tracked continuously
Use `TrackingMOde.once` on `AnchorComponent` to avoid continuous costs
Minimize persistent and transient anchor counts

* query latest ARKit data
* Pose prediction queries are not free
* Toggle collision data generation for scene understanding meshes

## audio and video
Spatial audio is used by default.
Requires real-time computation work

* concurrent playback
* moving sources
* sound stage size

Consider a video in a shared space.  Multiple videos at once.  EAch video, system has to decode and render.  Each new frame has to make it at consistent fps.

Your app should minimize UI and 3d render updates during playback.
Use different video frame rates.  consider 24/30.  
Reduce concurrent playback.

presentation methods.

[[Create a great spatial playback experience]]

## SharePlay
Great performance over long periods of time.

STart with great sustained performance offline.
Profile rendering and input on both sides.
Profile for power consumption
Turn off unneeded features during SharePlay

## system pressure
As per usual, we manage system resources.  When under thermal pressure,

* do less work
* `ProcessInfo.thermalStateDidChangeNotification`.
* Use thermal inducers to simulate higher thermal states.
* [[Designing for Adverse Network and Temperature Conditions]]

## Memory pressure

Limited amount of memory shared between system and all apps.  When device is close to limit, system terminates apps.  Starting with apps not actively used.

Reduce memory usage.
If your app has UI content, reduce UI rendering memory allocations by minimizing offscreen render passes, total windows, and media content
For 3d memory,reduce RK's texture and geometry memory.
Evaluate total audio and video memory use

[[Profile and optimize your game's memory]]
[[Detect and diagnose memory issues]]
[[iOS memory deep dive - 18]]

# Next steps
* profile apps during development
* Build a performance plan for spatial computing
* Optimize for rendering and power
* Collect performance field data


# Resources
* https://developer.apple.com/documentation/visionOS/analyzing-the-performance-of-your-visionOS-app
* https://developer.apple.com/documentation/visionOS/creating-a-performance-plan-for-visionos-app
