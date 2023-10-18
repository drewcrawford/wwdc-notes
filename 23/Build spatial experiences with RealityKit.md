Discover how RealityKit can bring your apps into a new dimension. Get started with RealityKit entities, components, and systems, and learn how you can add 3D models and effects to your app on visionOS. We'll also take you through the RealityView API and demonstrate how to add 3D objects to windows, volumes, and spaces to make your apps more immersive. And we'll explore combining RealityKit with spatial input, animation, and spatial audio.

#realitykit 

# RealityKit and SwiftUI
###  Build a new trait collection instance from scratch - 3:40
```swift
import SwiftUI
import RealityKit

struct GlobeModule: View {
    var body: some View {
        Model3D(named: "Globe") { model in
            model
                .resizable()
                .scaledToFit()
        } placeholder: {
          	ProgressView()
        }
    }
}
```

###  Volumetric window - 5:52
```swift
// Define a volumetric window.
struct WorldApp: App {
    var body: some Scene {
        // ...

        WindowGroup(id: "planet-earth") {
            Model3D(named: "Globe")
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 0.8, height: 0.8, depth: 0.8, in: .meters)
    }
}
```

###  ImmersiveSpace - 7:31
```swift
// Define a immersive space.
struct WorldApp: App {
    var body: some Scene {
        // ...

        ImmersiveSpace(id: "objects-in-orbit") {
            RealityView { content in
                // ...
            }
        }
    }
}
```

note that this is async. It copmletes once the space is finished opening.

[[Meet SwiftuI for spatial computing]]
[[Take SwiftUI to the next dimension]]

[[Go beyond the window with SwiftUI]]


# Entities and components
If you create an empty entity from code, it wont' do anything.  To render, it must have components.

earth - 2 components.  A model component, and a transform component.  Which places it in 3d space.

[[Explore materials in Reality Composer Pro]]


y -> up
x -> right
z -> towards you

1 unit - 1m

different from swiftUI:
x - right
y - down
z - towards you

components.


# RealityView
A SwiftUI view that contains RealityKit entities
###  RealityView - 12:40
```swift
import SwiftUI
import RealityKit

struct Orbit: View {
    let earth: Entity

    var body: some View {
        RealityView { content in
            content.add(earth)
        }
    }
}
```

lets you add entities to the view.  Easy way to get started, etc.

###  RealityView asynchronous loading and entity positioning - 12:54
```swift
import SwiftUI
import RealityKit

struct Orbit: View {
    var body: some View {
        RealityView { content in
            async let earth = ModelEntity(named: "Earth")
            async let moon = ModelEntity(named: "Moon")

            if let earth = try? await earth, let moon = try? await moon {
                content.add(earth)
                content.add(moon)
                moon.position = [0.5, 0, 0]
            }
        }
    }
}
```

connect observable state to component properties

convert between view and entity coordinate spaces

###  Earth rotation - 13:54
```swift
import SwiftUI
import RealityKit

struct RotatedModel: View {
    var entity: Entity
    var rotation: Rotation3D

    var body: some View {
        RealityView { content in
            content.add(entity)
        } update: { content in
            entity.orientation = .init(rotation)
        }
   }
}
```

###  Converting co-ordinate spaces - 14:27
```swift
import SwiftUI
import RealityKit

struct ResizableModel: View {
    var body: some View {
        GeometryReader3D { geometry in
            RealityView { content in
                if let earth = try? await ModelEntity(named: "Earth") {
                    let bounds = content.convert(geometry.frame(in: .local),
                                                 from: .local, to: content)
                    let minExtent = bounds.extents.min()
                    earth.scale = [minExtent, minExtent, minExtent]
                }
            }
        }
    }
}
```

subscribe to events
###  Play an animation - 14:56
```swift
import SwiftUI
import RealityKit

struct AnimatedModel: View {
    @State var subscription: EventSubscription? 

    var body: some View {
        RealityView { content in
            if let moon = try? await Entity(named: "Moon"),
               let animation = moon.availableAnimations.first {
                moon.playAnimation(animation)
                content.add(moon)
            }
            subscription = content.subscribe(to: AnimationEvents.PlaybackCompleted.self) {
                // ...
            }
       }
   }
}
```

attach views to entities
[[enhance your spatial computing app with RealityKit]]

# Input, animation, and audio
* add an input target component
* add a collision component
* Set up a DragGesture targeted to an entity

realitycomposer pro. #realitycomposer 

[[Meet Reality Composer Pro]]

important for collision shape to be reasonable match to the model.

###  Adding a drag gesture - 18:31
```swift
struct DraggableModel: View {
    var earth: Entity

    var body: some View {
        RealityView { content in
            content.add(earth)
        }
        .gesture(DragGesture()
            .targetedToEntity(earth)
            .onChanged { value in
                earth.position = value.convert(value.location3D,
                                               from: .local, to: earth.parent!)
            })
    }
}
```

mind the conversion.


hover effects
HoverEffectComponent
* applied outside of your app's process

animations
from-to-by
orbit
sampled
###  Playing a transform animation - 20:20
```swift
// Playing a transform animation
let orbit = OrbitAnimation(name: "Orbit",
                           duration: 30,
                           axis: [0, 1, 0],
                           startTransform: moon.transform,
                           bindTarget: .transform,
                           repeatMode: .repeat)

if let animation = try? AnimationResource.generate(with: orbit) {
    moon.playAnimation(animation)
}
```


## audio
spatial audio
ambient audio
channel - headtracked stereo.

###  Adding audio - 22:12
```swift
// Create an empty entity to act as an audio source.
let audioSource = Entity()

// Configure the audio source to project sound out in a tight beam.
audioSource.spatialAudio = SpatialAudioComponent(directivity: .beam(focus: 0.75))

// Change the orientation of the audio source (rotate 180º around the Y axis).
audioSource.orientation = .init(angle: .pi, axis: [0, 1, 0])

// Add the audio source to a parent entity, and play a looping sound on it.
if let audio = try? await AudioFileResource(named: "SatelliteLoop",
                                            configuration: .init(shouldLoop: true)) {
    satellite.addChild(audioSource)
    audioSource.playAudio(audio)
}
```


# Custom systems

component - data controlling one aspect of a 3d experience
grouped into entities - without components, an entity does nothing

###  Defining a custom component - 23:47
```swift
// Components are data attached to an Entity.
struct TraceComponent: Component {
    var mesh = TraceMesh()
}

// Entities contain components, identified by the component’s type.
func updateTrace(for entity: Entity) {
    var component = entity.components[TraceComponent.self] ?? TraceComponent()
    component.update()
    entity.components[TraceComponent.self] = component
}

// Codable components can be added to entities in Reality Composer Pro.
struct PointOfInterestComponent: Component, Codable {
    var name = ""
}
```
[[Work with Reality Composer Pro content in Xcode]]
systems act on entities/components.

###  Defining a system - 24:51
```swift
// Systems supply logic and behavior.
struct TraceSystem: System {
    static let query = EntityQuery(where: .has(TraceComponent.self))
    
    init(scene: Scene) {
        // ...
    }

    func update(context: SceneUpdateContext) {
         // Systems often act on all entities matching certain conditions.
        for entity in context.entities(Self.query, when: .rendering) {
            addCurrentPositionToTrace(entity)
        }
    }
}

// Systems run on all RealityKit content in your app once registered.
struct MyApp: App {
    init() {
        TraceSystem.registerSystem()
    }
}
```

# Wrap up
* display 3d content with realitykit
* respond to gestures, play animations, and spatial audio
* create custom components and systems

[[enhance your spatial computing app with RealityKit]]
[[Work with Reality Composer Pro content in Xcode]]

# Resources
* https://developer.apple.com/documentation/visionOS/diorama
