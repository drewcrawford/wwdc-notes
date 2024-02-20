Explore SwiftUI's powerful animation capabilities and find out how these features work together to produce impressive visual effects. Learn how SwiftUI refreshes the rendering of a view, determines what to animate, interpolates values over time, and propagates context for the current transaction.

Overview of swiftui's animation capabilities.

# Anatomy of an update

### 2:14 - Pet Avatar - Unanimated
```swift
struct Avatar: View {
    var pet: Pet
    @State private var selected: Bool = false

    var body: some View {
        Image(pet.type)
            .scaleEffect(selected ? 1.5 : 1.0)
            .onTapGesture {
                selected.toggle()
            }
    }
}
```

When an event comes in, an update transaction is opened.  At the close of the transaction, we call body to refresh the value.

1. transaction opens due to gesture
2. dependency is updated
3. attributes need to be updated by running body
4. transaction closes

scaleEFfect is a special attribute - animatable.  When teh value changes, it checks if an animation is set for transaction.   If so, makes a copy.  uses the animation to interpolate from old to new value as time passes.

Animatable attributes - ex param start/end
animation - like a curve.

# Animatable

### 4:13 - Pet Avatar - Animated
```swift
struct Avatar: View {
    var pet: Pet
    @State private var selected: Bool = false

    var body: some View {
        Image(pet.type)
            .scaleEffect(selected ? 1.5 : 1.0)
            .onTapGesture {
                withAnimation {
                    selected.toggle()
                }
            }
    }
}
```

We build attributes for any view conforming to protocol.  View must provide read-write vector with VectorArithmetic.  

In reality, scaleffect lets you independently configure width/height, etc.  All animatable.

scaleffect defines a 4-dimensional vector.

size - width/height
unit point - relative anchor

AnimatablePair<> fuses the two types together.

many animatable visual effects built into swiftui.  Usually, not an api you need to use directly.  But you may want to conform your own view.

consider the case we want to move along a curved line, like pivoting around a circle.

Note that a custom animatable performance may be much more expensive than a built-in effect.  Only use this if needed.

# Animation

Explicit animation like `.bouncy`

### 11:49 - Pet Avatar - Explicit Animation
```swift
struct Avatar: View {
    var pet: Pet
    @State private var selected: Bool = false

    var body: some View {
        Image(pet.type)
            .scaleEffect(selected ? 1.5 : 1.0)
            .onTapGesture {
                withAnimation(.bouncy) {
                    selected.toggle()
                }
            }
    }
}
```
### 12:48 - UnitCurve Model
```swift
let curve = UnitCurve(
    startControlPoint: UnitPoint(x: 0.25, y: 0.1),
    endControlPoint: UnitPoint(x: 0.25, y: 1))
curve.value(at: 0.25)
curve.velocity(at: 0.25)
```

### 13:56 - Spring Model
```swift
let spring = Spring(duration: 1.0, bounce: 0)
spring.value(target: 1, time: 0.25)
spring.velocity(target: 1, time: 0.25)
```
3 buckets
* timing curve
	* linear, easeIn, etc.
	* tke a curve which takes speed/duration
	* can create with bezier control points.  
* spring
	* may be familiar with mass, stiffness, damping.  We haven't found them particularly intuitive
	* duration and bounce
	* smooth (no bounce), snappy (small bounce), bouncy (medium bounce).  Consider using the presets.  Presets can also be adjusted for duration or bounciness.
	* So strongly about the benefits of sprint animations that we made these the default for iOS 17+.
* higher order
	* slow down, speed up, delay, etc.
	* repeat
	* autoreverse
* Custom!!
	* Same entrypoints we use.  `CustomAnimation`.

* animate - gets vector, time, context.  Returns value of animation or nil if animation is finished.  Comes from a view's aniimatableData.  
* shouldMerge
* velocity


### 17:25 - MyLinearAnimation
```swift
struct MyLinearAnimation: CustomAnimation {
    var duration: TimeInterval

    func animate<V: VectorArithmetic>(
        value: V,
        time: TimeInterval,
        context: inout AnimationContext<V>
    ) -> V? {
        if time <= duration {
            value.scaled(by: time / duration)
        } else {
            nil // animation has finished
        }
    }
}
```

shouldMerge -> deal with multiple animations.

by default, shouldMerge is false, both animations run together and are combined by the system.

spring animations ahve shouldmerge is true and can do velocity-matching.

### 19:50 - MyLinearAnimation with Velocity
```swift
struct MyLinearAnimation: CustomAnimation {
    var duration: TimeInterval

    func animate<V: VectorArithmetic>(
        value: V, time: TimeInterval, context: inout AnimationContext<V>
    ) -> V? {
        if time <= duration {
            value.scaled(by: time / duration)
        } else {
            nil // animation has finished
        }
    }

    func velocity<V: VectorArithmetic>(
        value: V, time: TimeInterval, context: AnimationContext<V>
    ) -> V? {
        value.scaled(by: 1.0 / duration)
    }
}```
# Transaction

you may be familiar with
* environment
* preferences

transaction is similar.  Dictionary that implicitly propagates all context for the update.  Notably, the animation.

withAnimation -> animation in the root transaction dictionary.
when it reaches an animatable attribute, the attribute checks if the animation is set, and if so makes a copy to drive its value.

transaction is only relevant for a specific update, so it's discarded when not needed.



### 22:44 - Pet Avatar - Animation Modifier
```swift
struct Avatar: View {
    var pet: Pet
    @Binding var selected: Bool

    var body: some View {
        Image(pet.type)
            .scaleEffect(selected ? 1.5 : 1.0)
            .animation(.bouncy, value: selected)
            .onTapGesture {
                selected.toggle()
            }
    }
}
```

we can override the animation with `.transaction`.  Keep in mind that overriding animation for all descendents can lead to accidental animations.  For usecases like this, use `.animation(...)` modifier.  Scope the effect much more precisely, for a particular binding that is changing.


here we use two modifiers to get different animations on a per-attribute basis, for the same selected binding.
### 23:44 - Pet Avatar - Multiple Animation Modifiers
```swift
struct Avatar: View {
    var pet: Pet
    @Binding var selected: Bool

    var body: some View {
        Image(pet.type)
            .shadow(radius: selected ? 12 : 8)
            .animation(.smooth, value: selected)
            .scaleEffect(selected ? 1.5 : 1.0)
            .animation(.bouncy, value: selected)
            .onTapGesture {
                selected.toggle()
            }
    }
}
```

animation view modifier works well for leaf components with hierarchy under your control.  But arbitrary view hierarchies, might get an accidental animation.

Now we can use scoped animation modifiers.
### 25:20 - Generic Avatar - Scoped Animation Modifiers
```swift
struct Avatar<Content: View>: View {
    var content: Content
    @Binding var selected: Bool

    var body: some View {
        content
            .animation(.smooth) {
                $0.shadow(radius: selected ? 12 : 8)
            }
            .animation(.bouncy) {
                $0.scaleEffect(selected ? 1.5 : 1.0)
            }
            .onTapGesture {
                selected.toggle()
            }
    }
}
```

Now you can use custom transaction keys.  Kinda like custom environment keys, but different.

### 28:45 - Pet Avatar - Transaction Modifier
```swift
struct Avatar: View {
    var pet: Pet
    @Binding var selected: Bool

    var body: some View {
        Image(pet.type)
            .scaleEffect(selected ? 1.5 : 1.0)
            .transaction(value: selected) {
                $0.animation = $0.avatarTapped
                    ? .bouncy : .smooth
            }
            .onTapGesture {
                withTransaction(\.avatarTapped, true) {
                    selected.toggle()
                }
            }
    }
}

private struct AvatarTappedKey: TransactionKey {
    static let defaultValue = false
}

extension Transaction {
    var avatarTapped: Bool {
        get { self[AvatarTappedKey.self] }
        set { self[AvatarTappedKey.self] = newValue }
    }
}
```

Every value int he transaction dictionary reverts to the default value at the end of the transaction.

Like before, you can use scoped variants.  Value argument and subhierarchy in body closure.  Similar to animation modifiers discussed above.

### 28:58 - Generic Avatar - Scoped Transaction Modifier
```swift
struct Avatar<Content: View>: View {
    var content: Content
    @Binding var selected: Bool

    var body: some View {
        content
            .transaction {
                $0.animation = $0.avatarTapped
                    ? .bouncy : .smooth
            } body: {
                $0.scaleEffect(selected ? 1.5 : 1.0)
            }
            .onTapGesture {
                withTransaction(\.avatarTapped, true) {
                    selected.toggle()
                }
            }
    }
}

private struct AvatarTappedKey: TransactionKey {
    static let defaultValue = false
}

extension Transaction {
    var avatarTapped: Bool {
        get { self[AvatarTappedKey.self] }
        set { self[AvatarTappedKey.self] = newValue }
    }
}```
# Resources

* animatable determines what to aniamte
* animation interprets over time
* transaction
[[Animate with springs]]
[[Wind your way through advanced animations in SwiftUI]]
