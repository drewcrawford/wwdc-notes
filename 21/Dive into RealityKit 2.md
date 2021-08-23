# Entity Component System
* Pattern for structuring data and behavior
* Composition vs inheritance
* Now more customizable

## Entities
* Represent a thing in the scene
* Hierarchical structure
* A container for a set of Components
* Behavior comes from Components and Systems

## components
* A marker of entity's participation in a System
* State that persists between frames
* No functionality

## Entity queries
Performed on a scene to find matching components

We add forces on each fish's motion component to keep state between frames, sticking together, saying apart, point noses in the same direction.

Motion rolls up all these forces to decide new acceleration, velocity, and position.

```swift
class FlockingSystem: RealityKit.System {

    required init(scene: RealityKit.Scene) { }
    
    static var dependencies: [SystemDependency] { [.before(MotionSystem.self)] }

    private static let query = EntityQuery(where: .has(FlockingComponent.self)
                                                && .has(MotionComponent.self)
                                                && .has(SettingsComponent.self))
```

Telling the engine to init one of these by subclassing I guess?

If you don't specify dependencies, update functions will be executed in registration order.

In multiplayer AR, components: Codable are synced over the network.  However data/systems are not automatically synced.  Store in components please.

```swift
func update(context: SceneUpdateContext) {

        context.scene.performQuery(Self.query).forEach { entity in

            guard var motion: MotionComponent = entity.components[MotionComponent.self]
                else { continue }

            // ... Using a Boids simulation, add forces to the MotionComponent
            motion.forces.append(/* separation, cohesion, alignment forces */)

            entity.components[MotionComponent.self] = motion
        }
    }
```

It can be useful to create a system which only provides `init`, like registering event handlers.

## Architectural changes
Before, you would subscribe to `SceneEvents.update` closure.  Instead of that, you can call `.update()` and formally ordered in separate functions.  So `GameManager` plays less of a role.

Now the GameManager only have to add components to the entities to signify to the system those entities should be matched by their queries.

Prevously you would declare protocol conformances on Entity types to express that that type, has certain components.  Now, you don't need to subclass entity anymore, since it too can play less of a role.  Its attributes can be modeled as entity.  Free to add and remove components during the experience.

> The world is your oyster

## TransientComponent
* Is not included when an Entity is cloned
* Is included in a network sync in a shared experience if it conforms to Codable

## Store subscription while entity active
* A new extension on `Cancellable`
* Automatically unsubscribes from an event when the Entity is deactivated
* Is not included for clones of the entity

```swift
arView.scene.subscribe(to: CollisionEvents.Began.self, on: fish) { [weak self] event in
    // ... handle collisions with this particular fish
}.storeWhileEntityActive(fish)
```

```swift
class Settings: ObservableObject {
    @Published var separationWeight: Float = 1.6
    // ...
}

struct ContentView : View {
    @StateObject var settings = Settings()
    var body: some View { 
        ZStack {
            ARViewContainer(settings: settings) 
            MovementSettingsView()
              .environmentObject(settings)
        }
    }
} 





struct SettingsComponent: RealityKit.Component {
    var settings: Settings
}

class UnderwaterView: ARView {
    let settings: Settings
    private func addEntity(_ entity: Entity) {
        entity.components[SettingsComponent.self] = 
          SettingsComponent(settings: self.settings)
    }
}
```


# Material advancements
## SimpleMaterial
recap
* basecolor
* roughness
* metallic
## UnlimitMaterial
* recap
* color

## OcclusionMaterial
recap
* Masks virtual objects

## VideoMaterial
* Unlit
* Video
* Transparency (new).  If video file contains transparency
## PhysicallyBasedMaterial
USD.

* Normal map
* Blending (transparency)
	* +opacity threshold
* Ambient occlusion
* Clearcoat

[[Explore advanced rendering with RealityKit 2]]

# Animation advancements
## Animation API Recap
* Playback
	* Play
	* Repeat
	* Pause
	* Resume
	* Stop
	* Transition

## Blend layers
Play walk+idle on two separate blend layers
blend factor, speed.  

## Loading animations
AnimationResource per USDZ?
Single USD on the same timeline and then use animationView to slice them up.  Requires knowing timecode to slice them up.

## Octopus' animations
`FromToByAnimation<Transform>`.  Animate the position.

Sequence of animations.
# Character controller
Character can collide with objects.  Diver automatically interacts with mesh.

* Capsule shape
	* Height
	* Radius
* move(to:) each frame
* teleport(to:) alternatively 

I get the feeling this involves pathfinding in some way, although I'm uncertain
# Generated resources
* Face mesh

```swift
static let sceneUnderstandingQuery = 
    EntityQuery(where: .has(SceneUnderstandingComponent.self) && .has(ModelComponent.self))

func findFaceEntity(scene: RealityKit.Scene) -> HasModel? {
    let faceEntity = scene.performQuery(sceneUnderstandingQuery).first {
        $0.components[SceneUnderstandingComponent.self]?.entityType == .face
    }
    return faceEntity as? HasModel
}
```

```swift
func updateFaceEntityTextureUsing(cgImage: CGImage) {
  guard let faceEntity = self.faceEntity else { return }
  guard let faceTexture =
  try? TextureResource.generate(from: cgImage,
                                options: .init(semantic: .color))
  else { return }

  var faceMaterial = PhysicallyBasedMaterial()        
  faceMaterial.roughness = 0.1
  faceMaterial.metallic = 1.0
  faceMaterial.blending = .transparent(opacity: .init(scale: 1.0))

  let sparklyNormalMap = try! TextureResource.load(named: "sparkly")
  faceMaterial.normal.texture = PhysicallyBasedMaterial.Texture.init(sparklyNormalMap)

  faceMaterial.baseColor.texture = PhysicallyBasedMaterial.Texture.init(faceTexture)

  faceEntity.model!.materials = [faceMaterial]
}
```

## AudioBufferResource
* AVSpeechSynthesizer
* AVAudioBuffer

```swift
let synthesizer = AVSpeechSynthesizer()

func speakText(_ text: String, forEntity entity: Entity) {

    let utterance = AVSpeechUtterance(string: text)
    utterance.voice = AVSpeechSynthesisVoice(language: "en-IE")

    synthesizer.write(utterance) { audioBuffer in

        guard
            let audioResource = try? AudioBufferResource(buffer: audioBuffer,
                                                         inputMode: .spatial,
                                                         shouldLoop: true)
        else { return }

        entity.playAudio(audioResource)
    }
}
```

# Wrap up
* ECS
* material advancement
* Animation advancements
* Character controller
* Generated resources

[[Explore advanced rendering with RealityKit 2]]

[[Building apps with RealityKit - 19]]

* Geometry modifiers
* Surface shaders => octopus
* Post processing => fog, water austics
* Dynamic meshes

https://developer.apple.com/documentation/realitykit/building_an_immersive_experience_with_realitykit
https://developer.apple.com/documentation/realitykit/modelcomponent/applying_realistic_material_and_lighting_effects_to_entities
https://developer.apple.com/documentation/realitykit/physicallybasedmaterial
https://developer.apple.com/documentation/arkit/environmental_analysis/creating_a_fog_effect_using_scene_depth
https://developer.apple.com/documentation/realitykit
