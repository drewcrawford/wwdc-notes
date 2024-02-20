Discover how you can bring your app's AR experience to visionOS. Learn how ARKit and RealityKit have evolved for spatial computing: We'll highlight conceptual and API changes for those coming from iPadOS and iOS and guide you to sessions with more details to help you bring your AR experience to this platform.

# ARKit on iOS

Scene understanding.  Insight about the real world around you.  Using the provided geometry and semantic knowledge, your content can be intelligently placed and realistically interact with surroundings.

Rendering -> render and composite.  Use camera transforms and intrinsics.

Initially we started with sceneKit on iOS.  We then introduced realitykit.  Laying out the foundation for an engine capable of highly realistic PBR and accurate object simulation.

To enable spatial computing, aRKit and RK have matured and are deeply integrated into the OS.  ex, 

* ARKit and RealityKit deeply integrated
* Built-in camera passthrough and hand matting
* World map persistence is a system service
	* your app doesn't need to do persistence

ARKit now provides hand tracking to your app.  Reach out and directly interact with virtual content to interact with your surroundings.  To take advantage, you'll need to update your iOS ARKit-based experience.  Reimagine your app and AR experiene for spatial computing.

# Prepare your experience

* Present your app
* prepare your content

Your app can open one or more widnows to display ocntent.  Create a 3-dimensional volume.  Now, you can show a list of available board games in one window, the rules ina nother, and open the selected game in its own volume.  Play while keeping the safari window open.

Content you add to the window/volume stays contained within its bounds to allow sharing with other applications.

You may want your app to have more control.  Open a dedicated full space, in which onlny your app's windows, volumes, and 3d objects appear.  Once in a full space, your application has access to more features.  Using realitykit's anchor entities, target/attach objects to surroundings, tables, floors, palm, wrist, etc.

ARKit.  Only in full space.  Data in real-world surfaces, scene geometry, and skeletal hand tracking.  

[[Meet SwiftuI for spatial computing]]
[[Go beyond the window with SwiftUI]]

USD.  Production-proven, scales from creators with single assets, large studios, etc.  Bring them into reality composer pro.  Compose, edit, and preview your 3d content.  If you're using custom materials on iOS, you will need to rebuild with shader graph.  Ability to edit your RK componetns directly through UI.  Import RC pro project directly into xcode.  Easily bundle all USD accents, materials, components, etc., into your xcode project.

[[Meet Reality Composer Pro]]
[[Explore materials in Reality Composer Pro]]

# Use RealityView
SwiftUI - best 2d ui, system gesture events, etc.
realitykit - 3d rendering for experiences
powerful ECS.

The way to interfac with both is through RealityView, a new SwiftUI view to cater to spatial computing.  Bridges SwiftUI and RealityKit.  Colmbine 2d/3d elements to create a memorable spatial experience.

Container for entities.  Interact with SwiftUI gestures
Realistic simulations with surroundings


quick review of ECS.
systems - interact on entities that have the right component.  Many concepts from RK carry over from ARKit.

* contain entities
* gesture support
* anchorentity

ARView on iOS - permission required.
RealityView - uses system services to enable anchor entities.  So spatial experiences can anchor content without requiring permissions.  App susing this approach do not receive the underlying scene understanding data.  Not having transform data has some implications.

Learn more:
gestures
realityview
anchorentity
[[Build spatial experiences with RealityKit]]
# Bring in your content

Shared space.

Add them directly to RealityView's Content.  By adding entities to your view.
Entities are relative to space origin
Can interact with other entities in the space

consider transitioning to full space.  Apps can now additionally anchor content to surroundings.  
specify target for system to find, automatically anchor, etc.
no user permission is required.
transforms are not shared
AnchorEntity now supports hands

Use aRKit scene understanding
Leverage ARKit anchor data
Utilize world anchors
Transforms are relative to space origin
Requires uesr permission.

# Raycasting

Translate 2d input to 3d position.  But with new platform, we can use hands to naturally interact with experiences directly in 3d.

Requires collision components.
Raycast with ssytem gestures or hand tracking
Add world anchor

Need input and collision targets for gestures.  By adding SpatialTapGesture...

can raycast with hand anchors.  finger joint information to build origin and direction of the ray.  
# ARKit updates

* Run your session
* Receive anchor update
* Persist world anchors

On iOS, arkit provides different configurations.  Bundles capabilities necessary for your experience.  ex world tracking configuration, to get plane detection, etc.  We then create our ARSession with this configuration.

On xros, arkit now exposes a data provider, for each capability.  Hand tracking is a new capability with its own provider.  Instead of choosing from a catalog of preset configurations, you get a la carte selection.

## anchor updates
On iOS, single delegate receives anchor and frame updates.  Anchors are aggregated and synced with frames.  Apps responsible for displaying camera pixel buffer and using transforms.  Mesh/frame anchors are relative to base andhors, and up to you to disambiguate which is which.

On xrOS, data providers provide updates.  Onc eyou run the session, each provider publishes updates.  Scene reconstruction provider produces meshanchors, and plane detection provider provides planeanchors.  Anchor updates come as soon as they're available, decoupled.  Important to note that ARFrames are no longer provided.  Spatial computing applications do not need camera data, done automatically by the system.

ARKit can now deliver frames immediately, reducing latency.  

## persist world anchors

On iOS, app's responsibility to handle world map persistence.  Request/save, add logic to load map, relocalize, get anchors, show content, etc.

New platform, system continuously persists world map in bg.  Loading, unloading, creating, relocalizing, etc.  App doe snot have to handle maps anymore, system does it for you.  

Use WorldTrackingProvider to add anchors to world map.  System saves these for you.  Updates tracking state and transforms, use id to load/unload content.  

# Learn more
[[Meet ARKit for spatial computing]]

# Wrap up
* update your AR app to take advantage of spacial computing
* Familiar frameworks, now integrated in new ways

