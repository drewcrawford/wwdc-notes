Learn how new cross-platform APIs in RealityKit can help you build immersive apps for iOS, macOS, and visionOS. Check out the new hover effects, lights and shadows, and portal crossing features, and view them in action through real examples.

### Add a highlight HoverEffectComponent - 4:24
```swift
// Add a highlight HoverEffectComponent
let highlightStyle = HoverEffectComponent.HighlightHoverEffectStyle(color: .lightYellow, strength: 0.8)
let hoverEffect = HoverEffectComponent(.highlight(highlightStyle))
spaceship.components.set(hoverEffect)
```

### Add a shader effect - 5:55
```swift
// Add a shader effect
let hoverEffect = HoverEffectComponent(.shader(.default))
spaceship.components.set(hoverEffect)
```

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

### Activating the gravity force effect - 12:14
```swift
// Activating the gravity force effect
let gravity = ForceEffect(effect: Gravity(), spatialFalloff: SpatialForceFalloff(bounds: .sphere(radius: 8.0)), mask: .asteroids)
planet.components.set(ForceEffectComponent(effects: [gravity]))
```

### Using PhysicsMotionComponent - 13:11
```swift
// Calculate initial velocity of the asteroid using radius and angle
let velocity = calculateVelocity(radius, angle)
let physicsMotion = PhysicsMotionComponent(linearVelocity: velocity)
asteroid.components.set(physicsMotion)
```

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

### Add a spotlight with shadow - 19:12
```swift
// Add a spotlight with shadow
guard let lightEntity = spaceship.findEntity(named: "HeadLight") else { return }
lightEntity.components.set(SpotLightComponent(color: .yellow, intensity: 10000.0, attenuationRadius: 6.0))
lightEntity.components.set(SpotLightComponent.Shadow())
```

### Disable shadow - 20:01
```swift
// Insert code snippet.
```

### Enable portal crossing - 21:36
```swift
// Enable portal crossing
portal.components.set(PortalComponent(target: portalWorld, clippingMode: .plane(.positiveZ), crossingMode: .plane(.positiveZ)))
spaceship.components.set(PortalCrossingComponent())
```

### Configure environmental lighting on the spaceship - 24:33
```swift
// Configure environmental lighting on the spaceship
var lightingConfig = EnvironmentLightingConfigurationComponent()
let distance: Float = computeShipDistanceFromPortal()
lightingConfig.environmentLightingWeight = mapDistanceToWeight(distance)
spaceship.components.set(lightingConfig)
```

### World tracking camera - 27:21
```swift
// World tracking camera
RealityView { content in
    #if os(iOS)
    content.camera = .worldTracking
    #endif
}
```

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

# Resources
* https://developer.apple.com/documentation/RealityKit/creating-a-spaceship-game
