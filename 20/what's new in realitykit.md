# Video materials
Use videos as materials
Video materials get textures from videos
Instructional videos, bring images to life.
Combine ARKit image anchors with video materials to bring videos to life.

Video materials also play spatialized audio.  That entity becomes a spatialized audio source.  

1.  Load video file.  Use AVPlayer
2.  Create and use `VideoMaterial`

AVPlayer - seek lets you use a "video atlas" instead of having separate videos??
Play remote media served using HTTP Live Streaming
[[advances in AVFoundation]]

# Scene underlstanding using LiDAR
Make virtual content interact with the real world.  

* ARView
  * Environment
	  * scene understanding
		  * occlusion
		  * receives lighting - automatically turns on occlusion
		  * physics - automatically turns on collision
		  * collision

## Occlusion
Bug can be hidden behind the tree.  

Add `.occlusion` to your scene understanding options.

## `receivesLighting`

Bug looks like it's floating because of the lack of shadows.

With `receivesLighting`, it casts a shadow.

No longer have to anchor objects to horizontal planes.  Note that the shadows imitate a light shining down.  You have to anchor your entity vertically.

## Physics

Static objects with infinite mass.  They're not moveable.
Meshes are constantly updating.  Don't expect objects to stay still, especially on nonplanar surfaces
Only visible regions are scanned.  If the user hasn't scanned a floor, objects will fal through
Mesh is approximate.  While the occlusion mask is very accurate, the mesh is less accurate.  Don't expect it to have super crisp edges.
Physics is not supported in collaborative sessions.

## Collision
Two usecases
1.  Raycasting against real world.  Use it to do pathfinding, object placement, line-of-sighte testing, etc.
2.  Collision against real-world objects.  (Detecting a collision, apparently)

### Raycasting

Cast from the body towards the tree to find the closest contact point.  Raycasting is also used to find different points for the bug to visit.

Note that in either case, we need an entity.  A scene understanding enity

### Scene understanding entity

* entity
	* components
		* transform
		* collision
		* physics
		* scene understanding component – unique to scene understanding entity.

Identify if an entity is real or not.

`HasSceneUnderstanding` trait.  All scene understanding entities will conform to this trait.

Consider these entities read-only, and don't modify properties.

`scene.subscribe(to: CollisionEvents.Began.self) { event in `

Check if sceneUnderstandingEntity conforms to `HasSceneUnderstanding`

`SceneUnderstanding` collision group
Use to filter for/against collisions with the real world

## Debug mesh visualization.

`arView.debugOptions.insert(.showSceneUnderstanding)`

Chart color-coding is here

Note this is raw mesh, if you want the physics mesh, use the physics debug view which is already available.

## Takeaways
Goal of scene understanding is to make your virtual content interact with teh real wrrold
occlusion and shadows help improve realism
physics and collision against real world meshes
scene understanding entity created and managed by realitykit
debug features


# improved rendering debugging

Added the ability to inspect various properties related to the rendering of our entity.
Can now display normal map, etc.  You can inspect this to look at your model and confirm it's loaded correctly.
Verify material parameters set on entity are correct
Visualize PBR-related output for tweaking materials.

`DebugModelComponent`.  Choose a property, create this thing, and then assign it to entity.  Currently we have 16 properties we support.  Vertex attributes, material parameters, and PBR outputs.

Visualization only appleis to targeted entity, does not inherit to children.  Add to each entity if you have a usdz file with multiple children.

# Integration with ARKit4

## Face tracking

Now extended to devices without TrueDepth camera, on A12 and later
FaceAnchors work with no change


## Location anchors

Create anchors using real-world coordinates.  `ARGeoAnchor` can be used with `AnchorEntity`.

`ARGEoAnchor` is a subclass of `ARAnchor`

ARKit also introduced a new `ARGeoTrackingConfiguration` for location anchors.  And since this is not a world-tracking configuration, you need... something
scene understanding etc. will not work

# Summarize
* Video materials
* Scene understanding lets content interact with the real world
* inspect rendering related properties
* face anchors available on more devices with no code change
* location anchors allow location-based AR content

