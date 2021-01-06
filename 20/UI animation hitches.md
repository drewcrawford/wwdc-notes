tech talk 2020, 3 talks
# Explore UI animation hitches and the render loop

## what is a hitch?
Any time a frame appears on screen later than expected.

Repeating frames.

The render loop fails to finish a frame ontime.
## The Render Loop
Continuous process by which touch events are sent, and frames output.
iPhone/iPad: 60hz: every 16.67ms
iPad Pro - 120hz, 8.33ms

Beginning of a frame, VSYNC happens.  Render loop is timed to vsyncs.  

1.  App
2.  Render server
3.  On the display

Double buffering.  But the system may switch to triple-buffering.  But this is a fallback mode.

Phases.
0.  Event phase
1.  Commit phase
2.  (render server) render prepare
3.  (render server) render execute
4.  display

Although render happens in a separate process, it happens on your behalf.  So it's on you to optimize.


### ex
0.  Events.  Touch, network, keyboard, timers.  App alters layer hierarchy.  bounds, etc.  
	1.  CA calls `setNeedsLayout`.  System batches layouts and reorders.
1.  Commit phase
	1.  Layout.  Common performance bottleneck.  Keep in mind you only have a few ms.
	2.  Custom drawing – drawRect.  If these require a visual update, they must call `setNeedsDisplay`.  Like layout, system will coalesce to perform after layout.  Views receive texture-backed or graphics context to draw into.  
2. Render prepare phase.  Process layers, effects, and animations for execution.  From parent to child, sibling to sibling.  Back-to-front.
3. Render execute, linear pipeline on GPU.  Some layers take longer to render, we will discuss shortly.
4. Display

Double-buffing parallel pipelining.  App renders new frame while system renders the previous frame.

## types of hitches

### commit hitches
App takes too long to process events or commit.  Here, render server has nothing to process because it doesn't get the commit in time.

(see other talk about this phase)
### render hitch
Render server can't prepare or execute in time.  Render phase is late.
(see other talk about this phase)
## Measuring hitch time
It's difficult to compare hitches unless each scroll/animation has the exact same number of frames.  iOS devices do not always update the screen, e.g. if no commits etc.  Makes it hard to compare hitch times across tests/devices.

Hitch time ratio - normalizes total hitch time for different intervals.  Since it's normalized to total time, we can compare across experiences.  hitch ms / S.

[[What's new in MetricKit]]
[[Eliminate animation hitches with XCTest]]

### no hitches
Duration: 30 frames at 60fps = 0.5s
Hitch time: 0ms
Hitch time ratio: 0/0.5 = 0

### many hitches
Duration: 30 frames = 0.5s
Hitch time: 100.02ms
Hitch time ratio: 100.02/0.5 = 200.04 ms/s

## targets
Critical >= 10ms/s
Warning: 5..10ms/s
Good: <5ms/s

## Wrap up
The render loop
* process of handling user events and displaying an updated UI

Hitches
* any time a frame appears later than expected
* commit and render hitches

Hitch time ratio
* metric to measure the amount of hitching within a duration

# Find and fix hitches in the commit phase
## Render loop
See prior talk

## What is a commit transaction?
Receive touch event
Records that layout or display are necessary
During the commit transaction, we do the display/layout.

Phases of commit
0.  Layout
1.  Display
2.  Prepare
3.  Commit

### Layout
`layoutSubviews` is called for every view that needs layout
This is required when
* positioning views (`frame` `bounds` `transform`)
* Adding/removing views
* Calling `setNeedsLayout()`.

### draw
`draw(rect:)` is called for every view that needs to update content
display is needed when 
* adding views that override `draw(rect:)`
* calls to `setNeedsDisplay()`

### prepare
Images that need to be decoded, will be decoded.
Can take time for large images.
Image conversion if the GPU can't use the color format.
[[Image and Graphics Best Practices – 18]]

### commit
Package up view layer tree recursively
send to render server

Note that deep view hierarchies take longer.

## Find hitches with Instruments
New template for hitches.
Demo.

Basically they accidentally disabled reuse, by overriding `prepareforResuse` and clearing all the views, which will have to be added again expensively.

## Recommendations
1.  Keep views lighteweight
	1. rely on CALayer properties over custom `draw(rect:)` code.
	2. Do not override `draw(rect:)` if not needed.
	3. Reuse views to avoid expensive add and remove (hierarchy) operations
	4. Take advantage of `hidden` rather than adding/removing.  This is a lot chaper.
	5. Reduce expensive or redundant layout
	6. Rely on `setNeedsLayout()`, not `layoutIfNeeded()`.
	7. Use the minimum number of contraints that you need
	8. Recursive layout is expensive, invalidate yourself (children if needed), not siblings or parent views.  

[[High performance AutoLayout – 18]]
[[Image and Graphics Best Practices – 18]]

## Wrap up
Understand the commit transaction pipeline to avoid expensive commits
Use the Animation Hitches template in Instruments to investigate hitches
Ensure `parepareForReuse()` does not incur additional work
View hierarchy lightweight.

Next talk!

# Demystify and eliminate hitches in the render phase
## The render loop
Previous talk.
## What are the render phases?
Render server renders the commits.  If renderserver takes longer than a frame duration, it hitches.
Even though rendering happens out of process, it's done on your behalf.

### Render prepare
* breaks down animations from the commit to be rendered over time
* Breaks down layers/effects into a step-by-step plan of simple operations

### render execute
* draws each step in the pipeline using GPU
* compiles the final image for display

### ex
prepare
* step layer-by-layer to compile a piepline of draw commands.
* appears to be depth-first.

execute
* draw each layer into a single composite texture.

shadows require offscreen rendering.  The core idea here is that the shadow is defined by some layer +child? contents, but we haven't yet drawn the layer.  But we can't draw the layer first because then the shadow will be on top.

Instead we draw the layer (with children) to an offscreen buffer.  I'm imagining this is a depth/stencil situation, although he doesn't say.  Then we apply various transformations to the buffer (such as blur) to make a shadow, which we can then composite.

### offscreen pass
Anytime the gpu must render a layer by first rendering it somewhere else, then copying it over.

Four main types of offscreen passes
* shadowing
	* as discussed earlier
* masking
	* Problem here is that we need to draw some subtree, but we only want to draw some of it.  Evidently, we have to draw the whole subtree offscreen and then copy the region we want.
* rounded rectangles
	* Similar to masking
* visual effects
	* UINavigationBars, UITabBars, etc.

## Find hitches with Instruments
New template we discussed in the prior talk.

Renders track - prepare phase
GPU track - render phase?

Buffer count - number of buffers used by the render server at the time of the hitch.  This seems to be related to double/triple buffering.  Use the hitch duration to follow the hitch, regardless of pipelining.

Hitch type 

Render count - number of offscreen passes that the GPU had to make.

View debugger now has a 'show layers' button.  Editor->Show Layers.  Inspector has the offscreen count on a per-layer basis.  Below this, there are the reasons for the offscreen.  

New runtime issue type called 'optimization opportunitites'.  

Can save view debugger state to send via email.  File->Export View Hierarchy

## Recommendations
Use provided APIs.
* set `shadowPath` to inform the renderer of a shadow's shape
* Use `cornerRadius` and `cornerCurve` to make rounded layers

For most layers, this is all you need
```swift
view.layer.shadowPath = UIBezierPath(roundedRect: view.layer.bounds, cornerRadius: view.layer.cornerRadius).cgPath
```

Use `maskToBounds` where possible.  This is much more performant than custom masking.

Avoid `maskToBounds` entirely if you know content will not exceed the bounds.

## Wrap up
* Instruments template
* Optimization opportunities
* View hierarchy file format

See prior talks.
