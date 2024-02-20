*Learn* how to bring content from Reality Composer Pro to life in Xcode. We'll show you how to load 3D scenes into Xcode, integrate your content with your code, and add interactivity to your app. We'll also share best practices and tips for using these tools together in your development workflow. To get the most out of this session, we recommend first watching “Meet Reality Composer Pro” and “Explore materials in Reality Composer Pro" to learn more about creating 3D scenes.

In this session, we'll continue iterating on a project from an earlier session.

[[Meet Reality Composer Pro]]
[[Explore materials in Reality Composer Pro]]

# Load 3D content
### 3:12 - Loading an entity
```swift
RealityView { content in
    do {
        let entity = try await Entity(named: "DioramaAssembled", in: realityKitContentBundle)
        content.add(entity)
    } catch {
        // Handle error
    }
}
```


Put your assets in a swift package with rkassets directory inside of it.  Xcode compiles this into a format that's faster to load at runtime.

Child entities, and they in turn have child entities, etc.  

# Components
ECS - like OOP but different in various ways.

[[Dive into RealityKit 2]]
[[Build spatial experiences with RealityKit]]

### 6:39 - Adding a component
```swift
let component = MyComponent()
entity.components.set(component)
```

Only one component of each type per entity - it's a set.

RC pro generates a swift file (swift package).  The swift code we geenrated lives in the package.  That's where this `count` property is coming from by default.

We're using our POI component like a signifier, marker.  These entities act as placeholders for where we put our swiftUI buttons at runtime.

# user interface

Let's combine attachments with our POI component to create hovering buttons at runtime.

* make closure
* updates closure
* attachments view builder
### 12:21 - Attachments data flow
```swift
RealityView { _, _ in
    // load entities from your Reality Composer Pro package bundle
} update: { content, attachments in
           
   if let attachmentEntity = attachments.entity(for: "🐠") {
     		content.add(attachmentEntity)
 	 }
           
} attachments: {
    Button { ... }
       .background(.green)
       .tag("🐠")
}
```

Here's how data flows from one part of the realityview to another.  3 parameters to the initializer.

1.  Make -> where you load your initial setup scene from your RC pro bundle as an entity and add to scene
2. updates -> closure called when there are changes to your view's state.  Change things about your entities.  Not executed every frame, just when swiftui view state changes
3. attachments.  Make swiftUI views to put into your RK scene.  SwiftUI views start out in attachments view builder, and delivered tot he update closure in the attachments parameter.

Make closure has an attachments parameter.  Ready to go when you've first evaluated.  Only run once, etc.

Let's get further into update closure.
### 15:48 - Adding attachments
```swift
let myEntity = Entity()

RealityView { content, _ in
    if let entity = try? await Entity(named: "MyScene", in: realityKitContentBundle) {
        content.add(entity)
    }
} update: { content, attachments in

    if let attachmentEntity = attachments.entity(for: "🐠") {
        content.add(attachmentEntity)
    }

    content.add(myEntity)

} attachments: {
    Button { ... }
       .background(.green)
       .tag("🐠")
}
```

Note that if you create new entities in your update closure, you'll get duplicates.  To guard against that, onyl add entities that are created somewhere that's only run once.  Then you can add them from update as needed.

This is because it's a no-op to add the same entity twice, so as long as it's only been created once, it can be added to scene multiple times with no effect.

High-level, here's what we do.

1.  Add invisible entities.
2. Query for entities
3. Create a swiftui view for each
4. STore views in `@State` collection.Serve them in attachments viewbuilder
5. add them as entities

### 20:43 - Adding point of interest attachment entities
```swift
static let markersQuery = EntityQuery(where: .has(PointOfInterestComponent.self))
@State var attachmentsProvider = AttachmentsProvider()

rootEntity.scene?.performQuery(Self.markersQuery).forEach { entity in
  guard let pointOfInterest = entity.components[PointOfInterestComponent.self] else { return }
  
  let attachmentTag: ObjectIdentifier = entity.id

  let view = LearnMoreView(name: pointOfInterest.name, description: pointOfInterest.description)
                           .tag(attachmentTag)

   attachmentsProvider.attachments[attachmentTag] = AnyView(view)
}
```



### 21:40 - AttachmentsProvider
```swift
@Observable final class AttachmentsProvider {
    var attachments: [ObjectIdentifier: AnyView] = [:]
    var sortedTagViewPairs: [(tag: ObjectIdentifier, view: AnyView)] { ... }
}

...

@State var attachmentsProvider = AttachmentsProvider()

RealityView { _, _ in

} update: { _, _ in

} attachments: {
    ForEach(attachmentsProvider.sortedTagViewPairs, id: \.tag) { pair in
        pair.view
    }
}
```

Note that attachments have to be unique.  And since RC pro shows all custom component properties in the inspector panel, we keep the tag outside the inspector panel and therefore guarantee it's unique.

So we sepaerate our data into design-time and run-time data.  RuntimeComponent is for things we dont' want to deal with at design time.

RCPro automatically builds the component UI for you.  Inspects the swift code in your package and makes any codable components available for you in your scenes.  Note that it must be both within the component *and* Codable.

You'll see an error if it's of a type RC Pro won't serialize?


### 22:31 - Design-time and Run-time components
```swift
// Design-time component
public struct PointOfInterestComponent: Component, Codable {
    public var region: Region = .yosemite
    public var name: String = "Ribbon Beach"
    public var description: String?
}

// Run-time component
public struct PointOfInterestRuntimeComponent: Component {
    public let attachmentTag: ObjectIdentifier
}
```

### 25:38 - Adding a run-time component for each design-time component
```swift
static let markersQuery = EntityQuery(where: .has(PointOfInterestComponent.self))
@State var attachmentsProvider = AttachmentsProvider()

rootEntity.scene?.performQuery(Self.markersQuery).forEach { entity in
  guard let pointOfInterest = entity.components[PointOfInterestComponent.self] else { return }
  
  let attachmentTag: ObjectIdentifier = entity.id

  let view = LearnMoreView(name: pointOfInterest.name, description: pointOfInterest.description)
                           .tag(attachmentTag)

   attachmentsProvider.attachments[attachmentTag] = AnyView(view)
   let runtimeComponent = PointOfInterestRuntimeComponent(attachmentTag: attachmentTag)
   entity.components.set(runtimeComponent)
}
```



### 26:19 - Adding and positioning the attachment entities
```swift
static let runtimeQuery = EntityQuery(where: .has(PointOfInterestRuntimeComponent.self))

RealityView { _, _ in

} update: { content, attachments in x
   
    rootEntity.scene?.performQuery(Self.runtimeQuery).forEach { entity in
        guard let component = entity.components[PointOfInterestRuntimeComponent.self],
              let attachmentEntity = attachments.entity(for: component.attachmentTag) else { 
            return 
        }        
        content.add(attachmentEntity)
        attachmentEntity.setPosition([0, 0.5, 0], relativeTo: entity)
    }
} attachments: {
    ForEach(attachmentsProvider.sortedTagViewPairs, id: \.tag) { pair in
        pair.view
    }
}
```

# Play audio

Bring in an audio entity.

Preview your component by selecting the sound in the preview menu.  This won't automatically play the selected sound when entity is loaded in your app.  We must load the resource and tell it to play.

### 28:55 - Audio Playback
```swift
func playOceanSound() {
        
    guard let entity = entity.findEntity(named: "OceanEmitter"),
        let resource = try? AudioFileResource(named: "/Root/Resources/Ocean_Sounds_wav",
                                   from: "DioramaAssembled.usda",
                                   in: RealityContent.realityContentBundle) else { return }
        
    let audioPlaybackController = entity.prepareAudio(resource)
    audioPlaybackController.play()
}
```


# Material properties


We'll use a property from shader graph material to morph between two terrains.

Promote a node with RC Pro.  We geta  reference to entity, then we get its model components.  From model component, we get first material.  Cast to `ShaderGraphMaterial`.  Now we can set parameter value.  Then we set everything back where it was.
### 31:02 - Terrain material transition using the slider
```swift
@State private var sliderValue: Float = 0.0

Slider(value: $sliderValue, in: (0.0)...(1.0))
    .onChange(of: sliderValue) { _, _ in
        guard let terrain = rootEntity.findEntity(named: "DioramaTerrain"),
                var modelComponent = terrain.components[ModelComponent.self],
                var shaderGraphMaterial = modelComponent.materials.first 
                    as? ShaderGraphMaterial else { return }
        do {
            try shaderGraphMaterial.setParameter(name: "Progress", value: .float(sliderValue))
            modelComponent.materials = [shaderGraphMaterial]
            terrain.components.set(modelComponent)
        } catch { }
    }
}
```



### 31:57 - Audio transition using the slider
```swift
@State private var sliderValue: Float = 0.0
static let audioQuery = EntityQuery(where: .has(RegionSpecificComponent.self) 
                                    && .has(AmbientAudioComponent.self))

Slider(value: $sliderValue, in: (0.0)...(1.0))
    .onChange(of: sliderValue) { _, _ in
        // ... Change the terrain material property ...
                                
        rootEntity?.scene?.performQuery(Self.audioQuery).forEach({ audioEmitter in
            guard var audioComponent = audioEmitter.components[AmbientAudioComponent.self],
                  let regionComponent = audioEmitter.components[RegionSpecificComponent.self]
            else { return }

            let gain = regionComponent.region.gain(forSliderValue: sliderValue)
            audioComponent.gain = gain
            audioEmitter.components.set(audioComponent)
        })
    }
}
```

# Wrap up
* Use reality composer pro to prepare RK content
* place attachment placeholders in reality composer pro
* load and play audio
* drive material properties
