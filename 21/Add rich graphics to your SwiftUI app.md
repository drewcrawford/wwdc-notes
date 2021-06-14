#swiftui 
# Safe areas
Why is there still empty space at top/bottom even after removing padding?

By default, within the safe area.  Region inset from the outermost area.

Safe area helps you avoid drawing content under the keyboard.  

Multiple different safe areas.  Most common is the container safe area.  

Keyboard safe area - region within container safe area.

Possible to opt out of safe areas.  

```swift
// Ignore all safe areas
ContentView()
    .ignoresSafeArea()

// Ignore keyboard only
ContentView()
    .ignoresSafeArea(.keyboard)
```

 Behavior is new in iOS 15 and "aligned OS releases"

 1.  Full area
 2.  safe area
 3.  Content
 4.  If we add a background, naively it will be aligned as 3 (e.g., with safe area)

Can apply a modifier to the background to put the background beyond safe area.

Evidently this behavior is now the default??

When I use a custom shape, no longer extends beyond the safe area.

Can apply materials.  

* Ultra thin
* to ultra thick

Right design on every platform.

# Foreground styles
Foreground style of `.secondary`.  In swiftui, when you use secondary in a material context, you get vibrancy.

Whenever you use secondary or automatically applied (like a sidebar)

They do the right thing in non-blur context as well.

Change behavior with `.foregroundStyle` e.g. colors, gradients, etc.  

> Please use tastefully.

Any text can have a single foreground color applied, but multiple colors.

e.g., can use string interpolation to set a color on the inner string.

Embedded emoji has the right behavior as well.

When I scroll down, things are hidden behind the blur.  Why?

1.  Bar is zstacked on top of our content.  Now that we want to see all the view, it's not the right behavior
2.  Change to Vstack?  Without the list under the blur, we wouldn't see color shown blue
3.  We want background under the bar, but not the main content.

If the safe area is inset by the bar, then important content will be unobscured.

`safeAreaInset`.  Auxilary content, like th bar, over the main content.

Put our chrome controls into `safeAreaInset`.  


# Materials



# Drawing with canvas

1.  GeometryReader and ZStack.
2.  `drawingGroup` => combine all views in a single layer.

This works well for grahpical elements but shouldn't be used for UI controls like textfields and lists. 

Large number of graphical elements.  Even though these views are drawn differently, can use same functionality from swiftUI you use elsewhere.

If you have a high number of elements, even that might be too much.  New canvas view. (Evidently we don't demo this problem though)

If you're familiar with `drawRect`, canvas block works similarly.  Closure gets context - send drawing commands here.

When we draw an image at 0,0, we center at origin.  Since we have the size of the whole canvas, let's draw in the middle.

Adapts to dark mode.

Note that canvas code is imperative, not a viewbuilder.  So I can use a for loop.

Each time, the context needs to resolve it to evaluate based on the current environment.  Improve by resolving the image ourselves before drawing it.  Better performance by sharing the image. 

Create CGRect with samex/y and use image size as width and height.

Draw images with context.fill, which takes a path and a shading.  Construct a path with standard bezier curves.  Also use shapes like `Ellipse` and get their `.path(in: rect)`.

`context.opacity` behaves like you'd expect.  e.g. all operations that happen afterwards.

In the past, if I wanted to make a change to some, I'd have to bracket them in save/restore call.  But in SwiftUI, but all I have to do is make changes *on a copy*.  Drawing on the original context is unaffected.

Set a shading to control how symbols are drawn.  

Blend modes control how colors are combined, esp with partial opacity.  `.screen` combines colors so that they always get brigther.

To make this like a simulation, it has to actually move.

This year, we're introducing a new lower-level tool called `TimelineView`.

Configure with a schedule.  If you're familiar with displaylink, `.animation` works very similarly.

> I hope you remember your trigonometry.

Remember, that one of the tradeoffs of Canvas is that individual elements are combined.  So we can't attach a gesture to each image. 

However, we can add a gesture to the view.  Increase the number of sparkles shown.

Since canvas is a single graphic, there's no info about contents to accessibility.  We'll use `.accessibilityLabel("Foo")`

`.accessibilityChildren` lets you specify a swiftuI view hierarchy.

[[SwiftUI accessibility beyond the basics]]

Canvas works on
* watchOS
* tvOS
* macOS
* e.g. all swiftui platforms

# Wrap up
* SAfe areas
* Foreground styles
* Materials
* Drawing with canvas

