Adaptive Sync

# Displays on Apple platforms

* Fixed-rate displays.  iPhone, iPad, Mac built-in displays
* Variable-rate displays iPad promotion, or adaptive sync
# Adaptive-Sync displays on #macOS 
## What is a fixed rate display?
Each frame is presented 16ms.


## What is an Adaptive sync display?

Instead of a static duration, each frame has a window of time that it can be onscreen.  This display oeprates between 40-120hz.  Frame 8-25ms.

Once maximum time has elapsed, system will refresh the panel and system will be unavailable for a short time (8ms?)

## Benefits of Adaptive-Sync
For apps that mostly run at max refresh rate, adaptive-sync has a free benefit.

Due to a momentary increas in scene complexity, frame runs in 9ms instead.  On fixed display, we double the frame time.  Perceptable hitch.

On adaptive-sync display, presented in 9ms.  So app incurs only 1ms penalty.  Not perceptible.

For workloads that can't reach max framerate, can provide smooth even frames.  Consider a game rnning a complex scene at 90hz.  Intermittent defect causes spikes down to 66hz.  By monitoring your app's GPU work, can respond by presenting your frames later until your complexity is lower.

## Best practices

On fixed-rate display, when you exceed internval, we recommended you slow down your rendering.  Typically, that means lowering fps to 30.

On adaptive sync, we recommend you present frames at the highest rate you can do evenly.  Recall that if you go under minimum, display might go unavailable for new frames

* Present your frames evennly
* Present your frames faster than the display's minimum rate
* As fast as makes sense for your app

## Adaptive-sync support on apple devices
Supported mac platforms
* Apple silicon macs
* Recent intel-based macs

Enabled in display preferences on supported adaptive-sync displays

App must be full screen.  

```objc
// Detecting an Adaptive-Sync display

- (BOOL) isAdaptiveSyncSupported:(NSScreen *)screen {
    NSTimeInterval minInterval = screen.minimumRefreshInterval;  
    NSTimeInterval maxInterval = screen.maximumRefreshInterval;  
    return minInterval != maxInterval;
}

// Detecting full-screen

- (BOOL) isWindowFullscreen:(NSWindow *)window {
    return ([window styleMask] &= NSFullScreenWindowMask) == NSFullScreenWindowMask;
}

// Tying it all together

- (BOOL) isAdaptiveSyncSchedulingEnabled:(NSScreen *)window {
    NSScreen* windowScreen = [window screen];
    return [self isWindowFullscreen:window] && [self isAdaptiveSyncSupported:windowScreen];
}
```

## Adaptive sync in action

```objc
// Drawable present APIs with frame-pacing

[commandBuffer presentDrawable:drawable afterMinimumDuration:interval];
[commandBuffer presentDrawable:drawable atTime:t];

// Drawable present API without frame-pacing
//roll your own solution
[commandBuffer presentDrawable:drawable];
```


```objc
id<CAMetalDrawable> currentDrawable = [metalLayer nextDrawable];

// Your encoder and command buffers here

[commandBuffer presentDrawable:currentDrawable];
```

Here we're relying on drawable backpressure.  On a fixed-rate display this isn't the best idea since there's no guarantee your GPU work will align to the refresh rate of the display.

But as you can see in instruments, when our scene is consistent this works out ok.  The problem is that this scene is running into periodic hitches.  This creates stutter.

Let's fix that by presenting at a fixed rate.

```objc
id<CAMetalDrawable> currentDrawable = [metalLayer nextDrawable];

NSTimeInterval userFramerateCap = 78.0;
NSTimeInterval userInterval     =  1.0 / userFramerateCap;

// Your encoders and command buffers are still here
[commandBuffer presentDrawable:currentDrawable afterMinimumDuration:userInterval];
```

Users don't encounter stutters.  

How do we do even frames without a single rate?

Here we usea  rolling average.

```objc
id<CAMetalDrawable> currentDrawable = [metalLayer nextDrawable];

// Your encoders and command buffers are still available!

//any guess is ok.
NSTimeInterval averageGPUTime = screen.minimumRefreshInterval;

[commandBuffer presentDrawable:currentDrawable afterMinimumDuration:averageGPUTime];

//measure the amount of time spent rendering the frame
[commandBuffer addCompletedHandler:^(id<MTLCommandBuffer> buffer) {
  const NSTimeInterval GPUTime = buffer.GPUEndTime - buffer.GPUStartTime;

  // Use an exponential moving average
  const double alpha = .25;

  averageGPUTime = (GPUTime * alpha) + (averageGPUTime * (1.0 - alpha));
}];
```

As you can see, we're similar to the previous example but this limit is determined by the previous rate.

Same program running smoothly on a less powerful mac without additional code changes.

## Learn more
[[Introducing Metal 2]]
[[Delivering optimized metal apps and games]]

# ProMotion on iPad Pro

Since 2017, every iPad pro has equipped with ProMotion and 120hz.  120hz may not be available in some situations, such as low power mode.  

Proper frame pacing allows your app to present motion contents correctly regardless of display characteristics, system state etc.

## ProMotion vs fixed rate displays
Fixed 60hz display refreshes every 16ms.  
However, when a content is slower than refresh rate, the display itself still refreshes at the same cadence.  So every other frame is the same as the previous.

ProMotion has
* Up to 120hz
* great responsiveness
* adapts to onscreen contents
* reduces power consumption

At 120hz, every 8ms.
Also offers intermediate rates.  Can dynamically adjust refresh rate, so with 60hz content, can refresh without repeats.  Otherwise would be required on fixed 120hz.  All the way down to 24 hz.

## Frame rate availability on ProMotion
* Limit frame rate in accessibility settings
* Thermal state
* Low power mode

* Most apps will work without any changes
* Frame-by-frame custom drawing requires extra care

## Drive custom drawing using display link
A timer synchronized with display refresh rate
Drive custom animations and custom render loop
`CVDisplayLink` and `CADisplayLink`.

We'll only discuss `CADisplayLink`.

Wakes up at every vsync.  App has full 8ms to complete its work.  Regular timer such as `NSTimer` is unlikely to be in perfect sync.  Can drift, etc.

Some additional benefits:
* Desired rate can be set as a hint via `preferredFramesPerSecond`
* Automatically adjust frame rate based on the system state
* Current timing information available via `timestamp` and `targetTimestamp`


## Display link best practices
Query the display refresh rate at runtime 
Use the actual frame rate of the `CADisplayLink` itself
use `argetTimestamp` to prepare the drawing
Dynamically cmpute the time delta.

## Query the display refresh rate at runtime
```objc
// Maximum frame rate from UIKit

NSInteger maxRate = [[UIScreen mainScreen] maximumFramesPerSecond];

// Current maximum frame rate from CoreAnimation

NSInteger currentMaxRate = round(1 / link.duration);
```

Use the actual frame rate from CADisplayLink
* `CADisplayLInks` can run slower than the maximum display refres rate
* Available frame rates are based on hardware capabilities
* The actual frame rate may be changed by the system

## Use the actual frame rate of the CADisplayLink
```objc
CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self 
                                                  selector:@selector(displayLinkCallback:)];

[link setPreferredFramesPerSecond:40];
[link addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];

- (void)displayLinkCallback:(CADisplayLink *)link {
    CFTimeInterval interval = link.targetTimestamp - link.timestamp;

    //...
}
```

## Use `targetTimestamp` to prepare drawing

* `timestamp` is when the callback is scheduled to be invoked
* `targettimestamp` is when the next frame will be composited by CoreAnimation.

Why `targetTimestamp`?

This has to do with switching framerates by CA.  Some problem with the derivative around the transition.  I think this is like if you're doing a constant simulation per frame?

Instead, we present with `targetTimestamp`.  

Replace any `timestamp` usage with `targetTimestamp`.

## Time delta
Difference between `targetTimestamp` and `timestamp` is the expected amount of time between displaylink callbacks

...but the *actual* amount of time is not guaranteed.

Higher priority thread may be scheduled on CPU or runloop gets busy, etc.
Callbacks may be skipped due to busy runloop.

Critical to do the correct timing.

When CADisplayLink is invoked, app performs its work.

Usually the callback will be invoked right at the scheduled wakeup time, but not alwas the case, e.g. if the runloop is blocked.  You may not get the full 8ms.

You can query `CACurrentMediaTime()` to get a sense of how much time is available.

Runloops may also skip your callbacks.

Be mindful that the `previousTargetTimestamp` to `targettimestamp` might be more than one frame.

Therefore, to use time delta...

```objc
- (void)displayLinkCallback:(CADisplayLink *)link {
    progress += link.targetTimestamp - previousTargetTimestamp;
    previousTargetTimestamp = link.targetTimestamp;

    [self renderAnimationWithProgress:progress];
}
```

Keep track of the previous target timestamp yourself to advance the state correctly.

And if your drawing has high workload, can look at `targetTimestamp` to reduce workload to meet deadline

# Summary
* Don't guess the refresh rate, query at runtime
* Adapt to different `CADisplayLInk` frame rates
* Use `targettimestamp`
* be prepared for missed callback

# Wrap up
Adaptive-sync displays on macOS
ProMotion on iPad pro

As display tech continues to evolve, we hope you have insight and tools best practices to support dynamic timing of displays.


