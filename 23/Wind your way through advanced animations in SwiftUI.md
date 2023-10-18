Discover how you can take animation to the next level with the latest updates to SwiftUI. Join us as we wind our way through animation and build out multiple steps, use keyframes to add coordinated multi-track animated effects, and combine APIs in unique ways to make your app spring to life.

###  Scale Animation - 0:42
```swift
struct Avatar: View {
    var petImage: Image
    @State private var selected: Bool = false

    var body: some View {
        petImage
            .scaleEffect(selected ? 1.5 : 1.0)
            .onTapGesture {
                withAnimation {
                    selected.toggle()
                }
            }
    }
}
```

adding animation is as easy as using `withAnimation`.  Great behavior out of the box.

SwiftUI applies animation that interpolates from the previous state tot he new state.  But with an animation, sometimes the most rewarding experiences are found where you aren't so focused on where you came from or where you're going.

repeating animations that loop continuously.  Or event-driven animations.

# Animation phases
I've been building an app to plan upcoming events.  Nutrition issuper important.  Easy to forget to eat late in a race.

Adding a feature that reminds me to eat the right times.

```swift
OverdueReminderView()
    .phaseAnimator([false, true]) { content, value in
        content
            .foregroundStyle(value ? .red : .primary)
    } animation: { _ in
        .easeInOut(duration: 1.0)
    }
```

You provide a sequence that provides the individual steps.  swiftui animates between them automatically.  Simpyl use boolean values.

apply modifiers to change appearance of view based on current phase.

opacity - fully opaque when highlighted, or 50% transparent otherwise.  Right away, the view starts animating.

what is swftui doing on our behalf?

false -> true
then advances back to false.  Loop around to the beginning.  This causes our animation to cycle between states.

by default, we use a spring animation.  in this case we want a more consistent animation.

here, we're building an ambient effect - okay for thigns to move slower.  Now that we've solved the urgent problem, lets look at event-driven animations.


###  Custom Phases - 6:20
```swift
ReactionView()
.phaseAnimator(
    Phase.allCases, 
    trigger: reactionCount
) { content, phase in
    content
        .scaleEffect(phase.scale)
        .offset(y: phase.verticalOffset)
} animation: { phase in
    switch phase {
    case .initial: .smooth
    case .move: .easeInOut(duration: 0.3)
    case .scale: .spring(
        duration: 0.3, bounce: 0.7)
    } 
}

enum Phase: CaseIterable {
    case initial
    case move
    case scale

    var verticalOffset: Double {
        switch self {
        case .initial: 0
        case .move, .scale: -64
        }
    }

    var scale: Double {
        switch self {
        case .initial: 1.0
        case .move: 1.1
        case .scale: 1.8
        }
    }
}
```


# Keyframes

when you need more control, use keyframes.

first, let's talk about how keyframes are different than phases.  phases define discrete states.  swiftui animates between them, using same animation types you already know.  Works well for animations modeled as discrete states.

All properties are animated at the same time.  SwiftUI animates to the next state.  Continues across all phases of the animation.

How to animate each property idnependently?  keyframes!


###  Keyframes - 9:48
```swift
ReactionView()
    .keyframeAnimator(initialValue: AnimationValues()) { content, value in
        content
            .foregroundStyle(.red)
            .rotationEffect(value.angle)
            .scaleEffect(value.scale)
            .scaleEffect(y: value.verticalStretch)
            .offset(y: value.verticalTranslation)
    } keyframes: { _ in
        KeyframeTrack(\.angle) {
            CubicKeyframe(.zero, duration: 0.58)
            CubicKeyframe(.degrees(16), duration: 0.125)
            CubicKeyframe(.degrees(-16), duration: 0.125)
            CubicKeyframe(.degrees(16), duration: 0.125)
            CubicKeyframe(.zero, duration: 0.125)
        }

        KeyframeTrack(\.verticalStretch) {
            CubicKeyframe(1.0, duration: 0.1)
            CubicKeyframe(0.6, duration: 0.15)
            CubicKeyframe(1.5, duration: 0.1)
            CubicKeyframe(1.05, duration: 0.15)
            CubicKeyframe(1.0, duration: 0.88)
            CubicKeyframe(0.8, duration: 0.1)
            CubicKeyframe(1.04, duration: 0.4)
            CubicKeyframe(1.0, duration: 0.22)
        }
        
        KeyframeTrack(\.scale) {
            LinearKeyframe(1.0, duration: 0.36)
            SpringKeyframe(1.5, duration: 0.8, spring: .bouncy)
            SpringKeyframe(1.0, spring: .bouncy)
        }

        KeyframeTrack(\.verticalTranslation) {
            LinearKeyframe(0.0, duration: 0.1)
            SpringKeyframe(20.0, duration: 0.15, spring: .bouncy)
            SpringKeyframe(-60.0, duration: 1.0, spring: .bouncy)
            SpringKeyframe(0.0, spring: .bouncy)
        }
    }

struct AnimationValues {
    var scale = 1.0
    var verticalStretch = 1.0
    var verticalTranslation = 0.0
    var angle = Angle.zero
}
```

swiftui provides you a value of types on each frame.  

`.keyframeAnimator` modifier accepts keyframes.
* initialValue - struct
* content - modifiers applied to the properties on the struct
* keyframes: `KeyframeTrack`.  
making an animation may take experimentation.  

* LinearKeyframe - liunear interpolation
* spring - spring function to interpolate to target value
* cubic - cubic bezier curve (catmull rom spline for multiple)
* move - immediate jump with no interpolation

swiftui maintains velocity between keyframes.

## remember
* predefined animation
	* think of them like video clips that can be played.  Because you specify exactly how it progersses, they can't gracefully retarget like springs.  Avoid changing mid-animation
* keyframes animate values, which you apply to a view
* the view updates on every frame, so be mindful of performance

# Tips and tricks

###  Map Keyframes - 15:22
```swift
struct RaceMap: View {
    let route: Route

    @State private var trigger = false

    var body: some View {
        Map(initialPosition: .rect(route.rect)) {
            MapPolyline(coordinates: route.coordinates)
                .stroke(.orange, lineWidth: 4.0)
            Marker("Start", coordinate: route.start)
                .tint(.green)
            Marker("End", coordinate: route.end)
                .tint(.red)
        }
        .toolbar {
            Button("Tour") { trigger.toggle() }
        }
        .mapCameraKeyframeAnimation(trigger: playTrigger) { initialCamera in
            KeyframeTrack(\MapCamera.centerCoordinate) {
                let points = route.points
                for point in points {
                    CubicKeyframe(point.coordinate, duration: 16.0 / Double(points.count))
                }
                CubicKeyframe(initialCamera.centerCoordinate, duration: 4.0)
            }
            KeyframeTrack(\.heading) {
                CubicKeyframe(heading(from: route.start.coordinate, to: route.end.coordinate), duration: 6.0)
                CubicKeyframe(heading(from: route.end.coordinate, to: route.end.coordinate), duration: 8.0)
                CubicKeyframe(initialCamera.heading, duration: 6.0)
            }
            KeyframeTrack(\.distance) {
                CubicKeyframe(24000, duration: 4)
                CubicKeyframe(18000, duration: 12)
                CubicKeyframe(initialCamera.distance, duration: 4)
            }
        }
    }
}
```

`.mapCameraKeyframeAnimator`.

If the user performs a gesture, the animation will be removed.

Keyframe manual evaluation.  Outside of the modifier, you can use the timeline type.  Initialize with initial value and tracks.

###  KeyframeTimeline - 16:26
```swift
// Keyframes
let myKeyframes = KeyframeTimeline(initialValue: CGPoint.zero) {
    KeyframeTrack(\.x) {...}
    KeyframeTrack(\.y) {...}
}

// Duration in seconds
let duration: TimeInterval = myKeyframes.duration

// Value for time
let value = myKeyframes.value(time: 1.2)
```

get the duration (of longest track)
calculate values for any time within range.  Easy to visualize keyframes with swift charts.  Use keyframe-defined curves however you want.  ex, geometryproxy to scrub with scroll position, etc.

# Conclusion
* use phases for chained animations
* use keyframes for more control
* have fun exploring

