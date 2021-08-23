# Motivation
Audio engine is often done through middleware different from the renderer.

We want to bring audio system closer to other subsystems.  And make it easier for applications.

* Geometry-aware audio engine
* Event-driven playback system
* Provide consistent spatial audio experience on supported apple devices
* Integrate with existing pipelines

## Common workflow
* Listener
* Sound source
* Occluder

Typically, you place multiple sources in an area.  Use raytracing to determine mix ratio etc.

If visual scenes change, have to update and hand-tune your audio experiences.

# Features


## Volumetric sounds
PHASE allows volumetric sources.  Pass sources to the underlying audio engine.

Can also pass occluders as shapes.  Can also choose acoustic material properties as presets.

If your app has indoor scenes, you can choose early reflections.

PHASE does the heavy lifting of occluding effects.

Now that your app's audio system is geometry-aware, adapts much quicker.

## Sound events
* Switch
* Blend
* Playback of loops or one-shots

## Sound events tree
Random node that will choose between its children

## Rendering system
Stereo, spatial, ambient bed



# Concepts
3 main concepts
## Engine
Engine can be broken down into 3 sections
* Asset registery
* Scene graph
* Rendering state

### Registry
Sound assets => Load directly from an audio file, or packed and embedded in assets and loaded directly into the engine.
Event assets => Collection of one or more hierarchical nodes, and downstream mixers.

### Scene graph
Hierarchy of objects.  Listener, occluder, source

PHASE supports point sources and volumetric sources.  Occluder is an object that represents geometry.  Occluders are assigned materials that affect how they absorb and transmit sound.

Library of occluder materials.  

### Rendering state
Playing sound events, audio IO.

INitially, audio IO is disabled.  Register your assets, build your scene graph, etc.  Without having to run audio IO.

Once you're ready to play back sound events, start the engine.  Likewise, when you're finished playing back events, you can stop the engine.  
## Nodes
Control playback of content.  Either generate or control playback.

### Generators
Always leaf nodes

#### sampler node
Play back registered sound assets.  
* asset
* playback mode
	* One-shot: play once and automatically stop.
	* Looping: audio file will play indefinitely until you stop the sampler
* cull option: what to do when inaudible.  Terminate => stops.  Sleep => stops rendering and start rendering when it becomes audible.  You don't have to manually start/stop sounds
* Calibration level
	* Sets real world level of sound in dB


### Control nodes
Set logic for how generators are selected, mixed, etc.

Always parent nodes and can be organized into hierarchies.

#### Random
Weighted random choce to select one child.  
#### Switch
Switch between its children depending on a parameter name.

#### Blend
Blends between its children based on a parameter value.  ex, assign a wetness parameter to a blend node which will blend between a loud footstep and a quiet splash on dry end.
#### Container
Plays all its children at once.  Every time the container is triggered, both samples will play back at the same time.



## Mixers
Control spatialization of audio content.

### Channel
Without spatialization.  Use for stem-based content that needs to be rendered directly to the output, such as stereo music or dialog
### Ambient
Partial spatialization (panning only)
Externalization over headphones
Use for multichannel that isn't being simulated in the environment.
### Spatial
Full spatialization (panning, distance, directivity)
Geometry-aware environmental effects
Externalization over headphones "through the application of binaural filters"

* Standard geometric spreading lost
	* Natural attenuation over distance
	* Increase or decrease effect
* Piecewise curved segments

For point sources,
* Cardioid directivity, hypercardioid
* Cone directivity

Geometry-aware environmental effects

* Direct path transmission
* Early reflections
* Late reverb

#### Direct path transmission
Direct and occluded paths

#### Early reflections
Intensity modification and coloration to the direct path.  ex specular reflection off walls and floor.

#### Late reverb
Sound of the environment.  Dense buildup of diffuse scattered energy.  Cues for room size and shape, and sense of development.

# Sample use cases
## Play an audio file
#### Create an engine
```swift
// Create an Engine in Automatic Update Mode.
let engine = PHASEEngine(updateMode: .automatic)

// Retrieve the URL to an Audio File stored in our Application Bundle.
let audioFileUrl = Bundle.main.url(forResource: "DrumLoop_24_48_Mono", withExtension: "wav")!

// Register the Audio File at the URL.
// Name it "drums", load it into resident memory and apply dynamic normalization to prepare it for playback.
let soundAsset = try engine.assetRegistry.registerSoundAsset(url: audioFileUrl,
                                                             identifier: "drums",
                                                             assetType: .resident,
                                                             channelLayout: nil,
                                                             normalizationMode: .dynamic)
															 
```

For more precise synchronization with frame update, prefer `.manual` over `.automatic`.  See documentation for more details.

`.resident` vs `.streaming`.

`normalizationMode: .dynamic`.  Advised to normalize input.


#### Register a sound asset

```swift
// Create a Channel Layout from a Mono Layout Tag.
let channelLayout = AVAudioChannelLayout(layoutTag: kAudioChannelLayoutTag_Mono)!
     
// Create a Channel Mixer from the Channel Layout.
let channelMixerDefinition = PHASEChannelMixerDefinition(channelLayout: channelLayout)

// Create a Sampler Node from "drums" and hook it into the downstream Channel Mixer.
let samplerNodeDefinition = PHASESamplerNodeDefinition(soundAssetIdentifier: "drums",
                                                       mixerDefinition: channelMixerDefinition)

// Set the Sampler Node's Playback Mode to Looping.
samplerNodeDefinition.playbackMode = .looping;

// Set the Sampler Node's Calibration Mode to Relative SPL and Level to 0 dB.
samplerNodeDefinition.setCalibrationMode(.relativeSpl, level: 0)

// Register a Sound Event Asset with the Engine named "drumEvent".
let soundEventAsset = try engine.assetRegistry.registerSoundEventAsset(rootNode:samplerNodeDefinition,
                                             identifier: "drumEvent")
										
```

Once a sound event asset is registered, I can create an instance and start playback.  

```swift
// Create a Sound Event from the Sound Event Asset "drumEvent".
let soundEvent = try PHASESoundEvent(engine: engine, assetIdentifier: "drumEvent")

// Start the Engine.
// This will internally start the Audio IO Thread.
try engine.start()

// Start the Sound Event.
try soundEvent.start()
```

shutdown
```swift
// Stop and invalidate the Sound Event.
soundEvent.stopAndInvalidate()

// Stop the Engine.
// This will internally stop the Audio IO Thread.
engine.stop()

// Unregister the Sound Event Asset.
engine.assetRegistry.unregisterAsset(identifier: "drumEvent", completionHandler:nil)

// Unregister the Audio File.
engine.assetRegistry.unregisterAsset(identifier: "drums", completionHandler:nil)

// Destroy the Engine.
engine = nil
```


## Build a spatial audio experience
Upgrade the mix to go to spatialization.

Selectively apply different environmental effects to the sound source.

Cull option will tell phase what to do when an event becomes inaudible.

Register asset with the engine.

```swift
// Create a Spatial Pipeline.
let spatialPipelineOptions: PHASESpatialPipeline.Options = [.directPathTransmission, .lateReverb]
let spatialPipeline = PHASESpatialPipeline(options: spatialPipelineOptions)!
spatialPipeline.entries[PHASESpatialCategory.lateReverb]!.sendLevel = 0.1;
engine.defaultReverbPreset = .mediumRoom

// Create a Spatial Mixer with the Spatial Pipeline.
let spatialMixerDefinition = PHASESpatialMixerDefinition(spatialPipeline: spatialPipeline)

// Set the Spatial Mixer's Distance Model.
let distanceModelParameters = PHASEGeometricSpreadingDistanceModelParameters()
distanceModelParameters.fadeOutParameters = PHASEDistanceModelFadeOutParameters(cullDistance: 10.0)
distanceModelParameters.rolloffFactor = 0.25
spatialMixerDefinition.distanceModelParameters = distanceModelParameters

// Create a Sampler Node from "drums" and hook it into the downstream Spatial Mixer.
let samplerNodeDefinition = PHASESamplerNodeDefinition(soundAssetIdentifier: "drums", mixerDefinition:spatialMixerDefinition)

// Set the Sampler Node's Playback Mode to Looping.
samplerNodeDefinition.playbackMode = .looping

// Set the Sampler Node's Calibration Mode to Relative SPL and Level to 12 dB.
samplerNodeDefinition.setCalibrationMode(.relativeSpl, level: 12)

// Set the Sampler Node's Cull Option to Sleep.
samplerNodeDefinition.cullOption = .sleepWakeAtRealtimeOffset;

// Register a Sound Event Asset with the Engine named "drumEvent".
let soundEventAsset = try engine.assetRegistry.registerSoundEventAsset(rootNode: samplerNodeDefinition, identifier: "drumEvent")
```

Need to create a scene for the simulation.  Listener, source, and occluder.

```swift
// Create a Listener.
let listener = PHASEListener(engine: engine)

// Set the Listener's transform to the origin with no rotation.
listener.transform = matrix_identity_float4x4;

// Attach the Listener to the Engine's Scene Graph via its Root Object.
// This actives the Listener within the simulation.
try engine.rootObject.addChild(listener)
```

```swift
// Create an Icosahedron Mesh.
let mesh = MDLMesh.newIcosahedron(withRadius: 0.0142, inwardNormals: false, allocator:nil)

// Create a Shape from the Icosahedron Mesh.
let shape = PHASEShape(engine: engine, mesh: mesh)

// Create a Volumetric Source from the Shape.
let source = PHASESource(engine: engine, shapes: [shape])

// Translate the Source 2 meters in front of the Listener and rotated back toward the Listener.
var sourceTransform: simd_float4x4
sourceTransform.columns.0 = simd_make_float4(-1.0, 0.0, 0.0, 0.0)
sourceTransform.columns.1 = simd_make_float4(0.0, 1.0, 0.0, 0.0)
sourceTransform.columns.2 = simd_make_float4(0.0, 0.0, -1.0, 0.0)
sourceTransform.columns.3 = simd_make_float4(0.0, 0.0, 2.0, 1.0)
source.transform = sourceTransform;

// Attach the Source to the Engine's Scene Graph.
// This actives the Listener within the simulation.
try engine.rootObject.addChild(source)
```

Occluder.

```swift
// Create a Box Mesh.
let boxMesh = MDLMesh.newBox(withDimensions: simd_make_float3(0.6096, 0.3048, 0.1016),
                             segments: simd_uint3(repeating: 6),
                             geometryType: .triangles,
                             inwardNormals: false,
                             allocator: nil)

// Create a Shape from the Box Mesh.
let boxShape = PHASEShape(engine: engine, mesh:boxMesh)

// Create a Material.
// In this case, we'll make it 'Cardboard'.
let material = PHASEMaterial(engine: engine, preset: .cardboard)

// Set the Material on the Shape.
boxShape.elements[0].material = material

// Create an Occluder from the Shape.
let occluder = PHASEOccluder(engine: engine, shapes: [boxShape])
    
// Translate the Occluder 1 meter in front of the Listener and rotated back toward the Listener.
// This puts the Occluder half way between the Source and Listener.
var occluderTransform: simd_float4x4
occluderTransform.columns.0 = simd_make_float4(-1.0, 0.0, 0.0, 0.0)
occluderTransform.columns.1 = simd_make_float4(0.0, 1.0, 0.0, 0.0)
occluderTransform.columns.2 = simd_make_float4(0.0, 0.0, -1.0, 0.0)
occluderTransform.columns.3 = simd_make_float4(0.0, 0.0, 1.0, 1.0)
occluder.transform = occluderTransform

// Attach the Occluder to the Engine's Scene Graph.
// This actives the Occluder within the simulation.
try engine.rootObject.addChild(occluder)
```

Play the sound event

```swift
// Associate the Source and Listener with the Spatial Mixer in the Sound Event.
let mixerParameters = PHASEMixerParameters()
mixerParameters.addSpatialMixerParameters(identifier: spatialMixerDefinition.identifier, source: source, listener: listener)

// Create a Sound Event from the built Sound Event Asset "drumEvent".
let soundEvent = try PHASESoundEvent(engine: engine, assetIdentifier: "drumEvent", mixerParameters: mixerParameters)
```


## Build a behavioral Sound Event
Organized into behavioral hierarchies.  Here I walk trhoguh consecutive samples of soundevent notes.

```swift
// Create a Sampler Node from "footstep_wood_clip_1" and hook it into a Channel Mixer.
let footstep_wood_sampler_1 = PHASESamplerNodeDefinition(soundAssetIdentifier: "footstep_wood_clip_1", mixerDefinition: channelMixerDefinition)
```

Random 2 footsteps
```swift
// Create a Sampler Node from "footstep_wood_clip_1" and hook it into a Channel Mixer.
let footstep_wood_sampler_1 = PHASESamplerNodeDefinition(soundAssetIdentifier: "footstep_wood_clip_1", mixerDefinition: channelMixerDefinition)

// Create a Sampler Node from "footstep_wood_clip_2" and hook it into a Channel Mixer.
let footstep_wood_sampler_2 = PHASESamplerNodeDefinition(soundAssetIdentifier: "footstep_wood_clip_2", mixerDefinition: channelMixerDefinition)

// Create a Random Node.
// Add 'Footstep on Creaky Wood' Sampler Nodes as children of the Random Node.
// Note that higher weights increase the likelihood of that child being chosen.
let footstep_wood_random = PHASERandomNodeDefinition()
footstep_wood_random.addSubtree(footstep_wood_sampler_1, weight: 2)
footstep_wood_random.addSubtree(footstep_wood_sampler_2, weight: 1)
```

Gravel vs wood.

```swift
// Create a Sampler Node from "footstep_gravel_clip_1" and hook it into a Channel Mixer.
let footstep_gravel_sampler_1 = PHASESamplerNodeDefinition(soundAssetIdentifier: "footstep_gravel_clip_1", mixerDefinition: channelMixerDefinition)

// Create a Sampler Node from "footstep_gravel_clip_2" and hook it into a Channel Mixer.
let footstep_gravel_sampler_2 = PHASESamplerNodeDefinition(soundAssetIdentifier: "footstep_gravel_clip_2", mixerDefinition: channelMixerDefinition)

// Create a Random Node.
// Add 'Footstep on Soft Gravel' Sampler Nodes as children of the Random Node.
// Note that higher weights increase the likelihood of that child being chosen.
let footstep_gravel_random = PHASERandomNodeDefinition()
footstep_gravel_random.addSubtree(footstep_gravel_sampler_1, weight: 2)
footstep_gravel_random.addSubtree(footstep_gravel_sampler_2, weight: 1)

// Create a Terrain String MetaParameter.
// Set the default value to "creaky_wood".
let terrain = PHASEStringMetaParameterDefinition(value: "creaky_wood")

// Create a Terrain Switch Node.
// Add 'Random Footstep on Creaky Wood' and 'Random Footstep on Soft Gravel' as Children.
let terrain_switch = PHASESwitchNodeDefinition(switchMetaParameterDefinition: terrain)
terrain_switch.addSubtree(footstep_wood_random, switchValue: "creaky_wood")
terrain_switch.addSubtree(footstep_gravel_random, switchValue: "soft_gravel")
```

Add a wetness blend

```swift
// Create a Sampler Node from "splash_clip_1" and hook it into a Channel Mixer.
let splash_sampler_1 = PHASESamplerNodeDefinition(soundAssetIdentifier: "splash_clip_1", mixerDefinition: channelMixerDefinition)
    
// Create a Sampler Node from "splash_clip_2" and hook it into a Channel Mixer.
let splash_sampler_2 = PHASESamplerNodeDefinition(soundAssetIdentifier: "splash_clip_2", mixerDefinition: channelMixerDefinition)

// Create a Random Node.
// Add 'Splash' Sampler Nodes as children of the Random Node.
// Note that higher weights increase the likelihood of that child being chosen.
let splash_random = PHASERandomNodeDefinition()
splash_random.addSubtree(splash_sampler_1, weight: 9)
splash_random.addSubtree(splash_sampler_2, weight: 7)

// Create a Wetness Number MetaParameter.
// The range is [0, 1], from dry to wet. The default value is 0.5.
let wetness = PHASENumberMetaParameterDefinition(value: 0.5, minimum: 0, maximum: 1)
    
// Create a 'Wetness' Blend Node that blends between dry and wet terrain.
// Add 'Terrain' Switch Node and 'Splash' Random Node as children.
// As you increase the wetness, the mix between the dry footsteps and splashes will change.
let wetness_blend = PHASEBlendNodeDefinition(blendMetaParameterDefinition: wetness)
wetness_blend.addRangeForInputValues(belowValue: 1, fullGainAtValue: 0, fadeCurveType: .linear, subtree: terrain_switch)
wetness_blend.addRangeForInputValues(aboveValue: 0, fullGainAtValue: 1, fadeCurveType: .linear, subTree: splash_random)
```


Somehow all these nodes are synchronized I guess.  Maybe playing the top node?

```swift
// Create a Sampler Node from "gortex_clip_1" and hook it into a Channel Mixer.
let noisy_clothing_sampler_1 = PHASESamplerNodeDefinition(soundAssetIdentifier: "gortex_clip_1", mixerDefinition: channelMixerDefinition)

// Create a Sampler Node from "gortex_clip_2" and hook it into a Channel Mixer.
let noisy_clothing_sampler_2 = PHASESamplerNodeDefinition(soundAssetIdentifier: "gortex_clip_2", mixerDefinition: channelMixerDefinition)

// Create a Random Node.
// Add 'Noisy Clothing' Sampler Nodes as children of the Random Node.
// Note that higher weights increase the likelihood of that child being chosen.
let noisy_clothing_random = PHASERandomNodeDefinition()
noisy_clothing_random.addSubtree(noisy_clothing_sampler_1, weight: 3)
noisy_clothing_random.addSubtree(noisy_clothing_sampler_2, weight: 5)

// Create a Container Node.
// Add 'Wetness' Blend Node and 'Noisy Clothing' Random Node as children.
let actor_container = PHASEContainerNodeDefinition()
actor_container.addSubtree(wetness_blend)
actor_container.addSubtree(noisy_clothing_random)
```

Together these represent the complete sound of the actor.

# Wrap up
Play an audio file
Build a spatial audio experience
Build a behavioral sound event

* https://developer.apple.com/documentation/phase

