#xctest 

# Hitch

Any time that a frame appears on the screen later than expected.

# Frames
iPHone: 60hz, frame duration: 16.67ms
iPad pro: 120hz, 8.33ms
A vsync indicates when the screen swaps the frame onto the display.

# Units

**Hitch time**: Time in ms that a frame is late to display
**Hitch ratio**: Hitch time in ms per second for a given duration

* Test
* scroll event
* transition

why not fps?

* 60 or 120fps may not always be the desired target
* The display may not be updated intentionally
* Target frame rate may intentionally be lower than possible.  e.g. clock icon only runs at 10fps

However, hitch time is not easily comparable.  Different durations, target refresh rates, or number of expected frames.

Normalize hitch ratio in ms per second

# Hitch ratio
<5ms/s is good
5-10ms/s is warning
\>=10ms, ,critical

We're going to talk about development workflow.  There's also a production workflow, see

[[What's new in MetricKit]]
[[Diagnose Performance Issues with the Xcode Organizer]]

# Metrics
* Clock
* CPU
* memory
* OSSignpost
* Storage
* ApplicationLaunch (XC12)
* Template for custom metrics

In xc12, when using an animation `XCTOSSignPostMetric` we get additional options.  total count of hitches, duration of hitches, ratio, frame rate, frame count.

# How to create `os_signpost` intervals.

Non-animation intervals: custom `os_signpost` with `.begin`
Animation interval: custom `os_signpost` with `.animationBegin`
also UIKit instrumented animation interval

# Testing

We recommend you use one of the predefined metrics

```swift
measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric]) {
	foodCollection.swipeUp(velocity: .fast)
}
```

We can call `stopMeasuring()` to manually stop?  And then reset our block.

# Configuration

Recommended config for performance testing

1.  New scheme with Release
2.  No debugger
3.  disable screenshot collection
4.  disable code coverage
5.  Turn off all diagnostic option (asan, etc.)

