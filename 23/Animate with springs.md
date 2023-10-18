Discover how you can bring life to your app with animation! We'll show you how to create amazing animations when you take advantage of springs and help you learn how to use them in your app.
#uikit 

UIs continue to grow more dynamic.  Change, motion, everywhere.  People love it, makes the interface more alive, easier to understand what's happening, etc.  Brings as ense of enjoyment to interacting with the UI.

# Why springs
Animations give us a better sense of continuity.  Can be jarring and confusing.  More natural if we see the object move from one place to the next.

Not just position.  Velocity changes can also feel unnatural.  One goal we ahve tis to make our animation have continuous position and velocity.

Ease in/out.  Curve.  It doesn't feel there there are any sudden jumps.  We can also examine a chart.

Be cautious of using linear animations.  

Spring has continuous position and velocity.  

With ease in/out, it does aniamte to the end, but its motion jerks to a halt as the gesture ends.  Just a prespecified curve, so now ay to represent an initial velocity.

A spring can start with any initial velocity.  So we get a natural feel.
SwiftUI - get this behavior for free.

 Spring doesn't only mea a bouncy animation.  Great tool.  But later on, we'll look at when it makes sense to use a spring with bounce, but springs with no bounce are great too.  Used all over iOS.

No single point where object abruptly ends.  Based on the behavior of an object in the physical world.  Feels more natural and believable.

Note that we don't want animations to start/stop at the same time.  They start/stop at their own time, as defined by the physical world.

# How springs work
We're modeling a spring with the motion of an object attached to a spring.
1.  Mass
2. stiffness
3. damping

Initial position -> object
Target position -> resting position of the spring.
release the object to start the animation.  The properties we use to define the spring system determine the type of motion that occurs.  Chaining them, changes the resulting animation.

There isn't a real object with mass and there isn't a spring with stiffness. 

Duration, bounce.  These do what you'd expect.  Increasing the duration makes it take longer, and increasing the bounce, adds bounce.

Adopting these universally across apple's design/engineering efforts.  All frameworks now support this.

Underdamped - bouncy (0-100%)
critically damped - smooth (0%)
overdamped - flattened 0 to -100%

bounce of 100% - is a cosine wave.  Oscillates back and forth.
Physical interpretation is that it's a spring with no friction.  Oscillates forever, never reaches its target position.

math is simple.  Just a cosine curve.  For this, the period corresopnds exactly to the curve.

Decreasing bounce - adds friction.  

Exponential decay curve.  The part that gives us the gradual feeling of coming to rest.  

Technically a spring keeps moving forever, just with smaller and smaller movement.  So we do need to choose a time to remove it.  

That amount of time, is called the settling duration.  This is different from the duration parameter for configuring a spring.  Depends on many factors.  Duration parameter is a perceptual duration that is chosen to be predictable, even as the other parameters change.

Because of its unpredictable nature, don't wait for the settling duration.  Use the new completion handler support in SwiftUI which uses a perceptual duration.

# How to use springs


###  Spring Preset - 18:00
```swift
withAnimation(.snappy) {
  // Changes
}
```

An important part is tuning for the exact context you need.  These can also be used as tunable starting points.  Change duration
###  Spring Preset with Custom Duration - 18:15
```swift
withAnimation(.snappy(duration: 0.4)) {
  // Changes
}
```
more/less bounce.
###  Spring Preset with Custom Bounce - 18:21
```swift
withAnimation(.snappy(extraBounce: 0.1)) {
  // Changes
}
```

###  Custom Spring - 18:37
```swift
withAnimation(.spring(duration: 0.6, bounce: 0.2)) {
  // Changes
}

// UIKit
UIView.animate(duration: 0.6, bounce: 0.2) {
  // Changes
}

// Core Animation
let animation = CASpringAnimation(perceptualDuration: 0.6, bounce: 0.2)
```

Programmatically convert parameters.  Can also create a model with a set of parameters like mass, etc.  Use it as a spring animation directly.
###  Spring Model - 18:57
```swift
let mySpring = Spring(duration: 0.5, bounce: 0.2)
let (mass, stiffness, damping) = (mySpring.mass, mySpring.stiffness, mySpring.damping)
```


###  Spring Model Animation - 19:16
```swift
let otherSpring = Spring(mass: 1, stiffness: 100, damping: 10)
withAnimation(.spring(otherSpring)) {
    // Changes
}
```
If you really want to do it yourself

###  Spring Parameter Conversion - 19:26
```swift
mass = 1

stiffness = (2π ÷ duration)^2

damping = 1 - 4π × bounce ÷ duration, bounce ≥ 0
          4π ÷ (duration + 4π × bounce), bounce < 0
```

Can use spring models to build your own advanced spring behaviors.  Call methods to get your builtin spring evaluation map for yourself. call value to get the position of ths pring.  Just pass in a target.  and the time to evaluate it iat.

Use the same inputs on a velocity method, to get that over time.
###  Evaluating Spring Model - 19:35
```swift
let mySpring = Spring(duration: 0.4, bounce: 0.2)
let value = mySpring.value(target: 1, time: time)
let velocity = mySpring.velocity(target: 1, time: time)
```

Build your own custom animations.  Just call into the spring model, and modify the inputs/outputs to apply customization.

[[Explore SwiftUI animation]] to learn more.

###  Custom Spring Animation - 20:15
```swift
func animate<V: VectorArithmetic>(
    value: V, time: Double, context: inout AnimationContext<V>
) -> V? {
    spring.value(
        target: value, initialVelocity: context.initialVelocity,
        time: effectiveTime(time: time, context: context))
}
```

###  Spring with No Bounce - 20:34
```swift
withAnimation(.spring(duration: 0.5)) {
    isActive.toggle()
}
```

###  Spring with Small Bounce - 21:07
```swift
withAnimation(.spring(duration: 0.5, bounce: 0.15)) {
    isActive.toggle()
}
```

###  Spring with Large Bounce - 21:14
```swift
withAnimation(.spring(duration: 0.5, bounce: 0.3)) {
    isActive.toggle()
}
```

Be cautious of using values higher than around 0.4  They may feel too exaggerated for a UI element.

When you're not sure, use a spring with bounce 0.  Gives you a great general-purpose spring that's the most versatile.  if you want your aniamtion to feel a little more playful, add bounce.  Can also make sense when you want an animation to feel more physical, such as the end of a gesture.

Consider consistency.  Is it serious or playful?  Relaxed?  Or fast-paced?  Choose spring-values that feel consistent with the UI around them.

# Spring animations
* a spring doesn't need to bounce
* start with a preset
* configure duration and bounce
