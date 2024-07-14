Learn how new cross-platform APIs in RealityKit can help you build immersive apps for iOS, macOS, and visionOS. Check out the new hover effects, lights and shadows, and portal crossing features, and view them in action through real examples.

High-performance simulation and rendering capabilities.  Wide variety of capabilities for your 3d content to blend seamlessly in a real-world environment.  Immersive spatial computing apps and games.

We have been getting amazing feedback.

# Hover FX and input
On visionOS you can display your app's content in a window, body, or space.  Show the spasceship in the hangar, etc.

Player can inspect the spaceship up close by dragging.  Base plate for this volume.  

Unlike the base plate, the spaceship does not show any visual highlights when the player looks at it...



### Add a highlight HoverEffectComponent - 4:24
```swift
// Add a highlight HoverEffectComponent
let highlightStyle = HoverEffectComponent.HighlightHoverEffectStyle(color: .lightYellow, strength: 0.8)
let hoverEffect = HoverEffectComponent(.highlight(highlightStyle))
spaceship.components.set(hoverEffect)
```


Now we can customize hover effects.

* highlight -> uniform highlight to the whole mesh
* shader -> integrating with shader graph material

New shader style is powerful, etc.

Use a shader graph material.



### Add a shader effect - 5:55
```swift
// Add a shader effect
let hoverEffect = HoverEffectComponent(.shader(.default))
spaceship.components.set(hoverEffect)
```

HoverState node.  Inputs: intensity value, animated from 0 to 1 when the player looks at the spaceship.  

[[Build a spatial drawing app with RealityKit]]

[[Create custom hover effects in visionOS]]


# Force FX and joints

SpatialTrackingSession.

This year, RK is introducing new spatial tracking API to make task even easier.  

Two anchor entities: index finger tip, and thumb tip.

### Control acceleration with left hand - 8:04
```swift
// Control acceleration with left hand
class HandTrackingSystem: System {
    func update(context: SceneUpdateContext) {
        let indexTipPosition = indexTipEntity.position(relativeTo: nil)
        let thumbTipPosition = thumbTipEntity.position(relativeTo: nil)
        let distance = distance(indexTipPosition, thumbTipPosition)
        let throttle = computeThrottle(with: distance)
        let force = spaceship.transform.forward * throttle
        spaceship.addForce(force, relativeTo: nil)
    }
}
```

[[Build a spatial drawing app with RealityKit]]


Force effects API.
Define a volume and continuously apply forces to physics bodies.  Pulls asteroids towards the center.  RK offers 4 built in effects types

* constrant radial - constant force towards center
* vortex -> circulate bodies around axis
* drag -> slows down physics bodies within its volume by applying afforce proportional to velocity
* turbulence -> random forces

We want magnitude to vary based on distance from planet. So we can define a custom force effect.


### Adding a gravity force effect - 10:50
```swift
// Adding a gravity force effect
struct Gravity: ForceEffectProtocol {
    var parameterTypes: PhysicsBodyParameterTypes {
        [.position, .distance]
    }
    
    var forceMode: ForceMode = .force
    
    func update(parameters: inout ForceEffectParameters) {
        guard let distances = parameters.distances, let positions = parameters.positions else { return }
        
        for i in 0..<parameters.physicsBodyCount {
            let force = computeForce(distances[i], positions[i])
            parameters.setForce(force, index: i)
        }
    }
}
```

Now we have a custom force effect.

Activate in our scene.


### Activating the gravity force effect - 12:14
```swift
// Activating the gravity force effect
let gravity = ForceEffect(effect: Gravity(), spatialFalloff: SpatialForceFalloff(bounds: .sphere(radius: 8.0)), mask: .asteroids)
planet.components.set(ForceEffectComponent(effects: [gravity]))
```

Demo.

Give asteroids initial velocity that follows orbit's trajectory.  With offsetting velocity we achieve orbital motion.

### Using PhysicsMotionComponent - 13:11
```swift
// Calculate initial velocity of the asteroid using radius and angle
let velocity = calculateVelocity(radius, angle)
let physicsMotion = PhysicsMotionComponent(linearVelocity: velocity)
asteroid.components.set(physicsMotion)
```

To add a trailer, the easiest approach is to make the trailer entity a child of the spaceship.  However, the connection is very rigid.  We can make it feel more playful using a physics joint.

Pins define a position and orientation relative to the entity. One joint connects 2 pins.

Like force effects, we offer several built-in joints.

* Fixed -> disallows both translation and rotation
* spherical -> no translation, but limited rotations around y/z axis.  And free rotations around the x axis
* revolute -> rotation around X axis only.
* Prismatic joint -> slide joint.  Translation along x axis only
* distance joint allows 3 movement in all 3 axes but only as long as the distance is between a given range.


### Add a custom joint - 16:19
```swift
// Add a custom joint
guard let hookEntity = spaceship.findEntity(named: "Hook") else { return }
let hookOffset: SIMD3<Float> = hookEntity.position(relativeTo: spaceship)
let hookPin = spaceship.pins.set(named: "Hook", position: hookOffset)
let trailerPin = trailer.pins.set(named: "Trailer", position: .zero)

var joint = PhysicsCustomJoint(pin0: hookPin, pin1: trailerPin)
joint.angularMotionAroundX = .range(-.pi * 0.05 ... .pi * 0.05)
joint.angularMotionAroundY = .range(-.pi * 0.2 ... .pi * 0.2)
joint.angularMotionAroundZ = .range(-.pi * 0.2 ... .pi * 0.2)
joint.linearMotionAlongX = .fixed
joint.linearMotionAlongY = .fixed
joint.linearMotionAlongZ = .fixed

try joint.addToSimulation()
```

Constrain angular motion along all 3 axes separately.  
Since we don't want any translation we set linear motion to be fixed.
# Dynamic lights

3 types of lights:
* spotlight, illuminates objects in a cone-shaped volume
* customize angle, distance, attenuation
* Directional light, all objects in a scene
* Point light, applied to objects.  You can customize its attenuation radius and fall off exponent.

Only spotlight and directional light can cast shadows.  As you introduce lights and shadows, check perf frequently as they can be expensive to render.

Swift API in code, or RC pro.
### Add a spotlight with shadow - 19:12
```swift
// Add a spotlight with shadow
guard let lightEntity = spaceship.findEntity(named: "HeadLight") else { return }
lightEntity.components.set(SpotLightComponent(color: .yellow, intensity: 10000.0, attenuationRadius: 6.0))
lightEntity.components.set(SpotLightComponent.Shadow())
```


Spotlight to cast shadow so I will also create an add shadow component to the entity.


### Disable shadow - 20:01
```swift
// Insert code snippet.
```

If you odn't want some objects to cast shadow, add `DynamicLightShadowComponent` with `castsshadow: false`.

# Portal enhancements

How to implement this feature with the portal API.  New enhancements.

Entities are masked by the portal geometry.  Only render inside of the portal surface.

[[enhance your spatial computing app with RealityKit]]

in visionOS 1, it's fully inside or outside the portal.  This year we add portal crossing.  So it transits the portal surface.

### Enable portal crossing - 21:36
```swift
// Enable portal crossing
portal.components.set(PortalComponent(target: portalWorld, clippingMode: .plane(.positiveZ), crossingMode: .plane(.positiveZ)))
spaceship.components.set(PortalCrossingComponent())
```

Set up crossing mode when creating the portal component.  Set mode to a plane to enable crossing, or disabled.  Make sure your plane coincides with the portal geometry itself.

Entities in the portal world don't have crossing enabled by default.  Add a portal crossing component.  Our spaceship can fly into the portal now.

for models with portal crossing enabled, inside is lit by a custom ImageBasedLighting component.

When model is outside, it receives additional lighting coming from an environment probe.  when crossing, we're inside and outside at the same time.  Inside part lit by IBL, outside part by IBL and environment probe.  To make the lighting transition smoother, we can use EnvironmentLightingConfigurationComponent.

### Configure environmental lighting on the spaceship - 24:33
```swift
// Configure environmental lighting on the spaceship
var lightingConfig = EnvironmentLightingConfigurationComponent()
let distance: Float = computeShipDistanceFromPortal()
lightingConfig.environmentLightingWeight = mapDistanceToWeight(distance)
spaceship.components.set(lightingConfig)
```

To dive deeper, I recommend checking out PortalComponent documentation.
# Cross platform capabilities

With this release of RK, now bring your spatial-computing experiences from visionOS seamlessly to iOS, iPadOS, macOS.  Possible to build our experience for other apple platforms.  Added this support for a number of RK features to make it seamless.

RealityView, ShaderGraph, HoverEffect, Text, and more.  Great opportunity to develop cross-platform tools and pipelines to accelerate app development.

On visionOS we can layout our UI in separate windows around the player.  On iPadOS we place UI directly on screen.

On visionOS we use immersiveSpace as entrypoint of spatial experience.  On iPadOS, we just show on the screen through RealityView.

### World tracking camera - 27:21
```swift
// World tracking camera
RealityView { content in
    #if os(iOS)
    content.camera = .worldTracking
    #endif
}
```

On visionOS we use hand-tracking based input.  On iPadOS, multitouch gestures work better.



### Multi-touch control views - 27:59
```swift
// Multi-touch control views
#if os(iOS)
struct MultiTouchControlView : View {
    var body: some View {
        HStack {
            ThrottleControlView()
            Spacer()
            PitchRollControlView()
        }
    }
}
#endif
```

[[Enhance your spatial computing app with RealityKit audio]]

Many more RK features that we didn't have time to cover.  Quick overview:

* LowLevelMesh and LowLevelTexture -> constructing updating texture resources
* Animations.  animation timelines in RC pro
* BillboardComponent.  Privacy-preserving way to make entities face user
* PixelCast
* Subdivision surface - smooth surfaces without creating a dense mesh

see docs.

# Recap
* Hover effects and spatial tracking
* Force effects and joints to add physics to our scene
* Dynamic lights and shadows
* Portal crossing
* Bringing spaceship to iPadOS

[[Build a spatial drawing app with RealityKit]]
[[Enhance your spatial computing app with RealityKit audio]]


# Resources
* https://developer.apple.com/documentation/RealityKit/creating-a-spaceship-game
