Discover how you can use ARKit's tracking and scene understanding features to develop a whole new universe of immersive apps and games. Learn how visionOS and ARKit work together to help you create apps that understand a person's surroundings — all while preserving privacy. Explore the latest updates to the ARKit API and follow along as we demonstrate how to take advantage of hand tracking and scene geometry in your apps.

Deeply woven into the OS fabric, powering everything from interacting with the window into playing with an immersive game.

New API, new design.  Everything we learned on iOS, plus unique needs of spatial computing.

Truly a magical experience.  Now taht you've seen the demo, let's dive in.

# Overview
## ARKit for spatial computing
* availabe in Swift and C
* Use any combination of features together!
* Privacy-first design for secure access to data

ARKitSession
data providers
anchors

## Anchors
Position/orientation in the real world.  Unique identifier, trasnform.

Some types of anchors are trackable.  When not being tracked, hide virtual content.

## data provider - 

ARKit feature. Poll/observe data updates.

## Session
combined set of ARKit features to use together for a particular experience.  Run a session by providing it with data providers.  Once runnning, the providers start receiving data.  Updates arrive asynchronously.

## Privacy
one of our core values.

data is not sent to client space.  It's sent to ARKit daemon.  Resulting data is carefully curated before being forwarded.

Prerequisites to accessing data.

Apps must enter a Full Space.  ARKit does not send data to shared space app.

| ?               | authorization type |
| --------------- | ------------------ |
| world tracking  | ?                  |
| plane detection | .worldSensing      |
| scene geometry  | .worldSensing      |
| Image tracking  | .worldSensing      |
| hand tracking   | .handTracking      |


Using session, can request authorization for kinds of data.  If you od not do this, ARKit will automatically prompt for permission when running the session, if necessary.

Batch all authorization types in a single rqeuest.
###  Authorization API - 1:51
```swift
session = ARKitSession()

Task {
    let authorizationResult = await session.requestAuthorization(for: [.handTracking])

    for (authorizationType, authorizationStatus) in authorizationResult {
        print("Authorization status for \(authorizationType): \(authorizationStatus)")

        switch authorizationStatus {
        case .allowed:
            // All good!
            break
        case .denied:
            // Need to handle this.
            break
        ...
        }
    }
}
```




# World tracking
virtual contenti nt he real world.  Tracks device movement in 6dof.  Type of data provider is called `WorldTrackingProvider`.

* add worldanchors for anchoring virtual content
* automatic persistence of WorldAnchors
	* There are some cases where persistence is not available
* get device's pose relative to the app's origin

worldanchor - trackable anchor, initializer takes a transform.  

demo involving world anchors.  Basically, when you press the diigtal crown, we recenter.  World anchored objects do not recenter.


## Device pose
provides the pose of the device relative to the app's origin
primarily tyargeted at developers doing their own rendering
Querying for the pose is relatively expensive.  Avoid for simple cases.


###  World Tracking Device Pose Render Struct - 10:20
```swift
#include <ARKit/ARKit.h>
#include <CompositorServices/CompositorServices.h>

struct Renderer {
    ar_session_t                 session;
    ar_world_tracking_provider_t world_tracking;
    ar_pose_t                    pose;

    ...
};

void renderer_init(struct Renderer *renderer) {
    renderer->session = ar_session_create();

    ar_world_tracking_configuration_t config = ar_world_tracking_configuration_create();
    renderer->world_tracking = ar_world_tracking_provider_create(config);

    ar_data_providers_t providers = ar_data_providers_create();
    ar_data_providers_add_data_provider(providers, renderer->world_tracking);
    ar_session_run(renderer->session, providers);

    renderer->pose = ar_pose_create();

    ...
}
```

###  World Tracking Device Pose Render function - 10:21
```swift
void render(struct Renderer *renderer,
            cp_layer_t       layer,
            cp_frame_t       frame_encoder,
            cp_drawable_t    drawable) {
    const cp_frame_timing_t timing_info = cp_drawable_get_frame_timing(drawable);
    const cp_time_t presentation_time = cp_frame_timing_get_presentation_time(timing_info);
    const CFTimeInterval target_render_time = cp_time_to_cf_time_interval(presentation_time);

    simd_float4x4 pose = matrix_identity_float4x4;

    const ar_pose_status_t status =
        ar_world_tracking_provider_query_pose_at_timestamp(renderer->world_tracking,
                                                           target_render_time,
                                                           renderer->pose);

    if (status == ar_pose_status_success) {
        pose = ar_pose_get_origin_from_device_transform(renderer->pose);
    }

    ...

    cp_drawable_set_ar_pose(drawable, renderer->pose);

    ...
}
```

[[Discover Metal for immersive apps]]
[[Optimize app power and performance for spatial computing]]

# Scene understanding

* plane detection
* scene geometry
* image tracking

## plane detection

horizontal/vertical surfaces.  PlaneDetectionProvider.  Each plane is provided as a PlaneAnchor.
Useful for content placement or low-fidelity physics simulations

alignment, geometry, and semantic classification.  ex wall, floor, window, etc.  we have 3 fail cases "unknown" "undetermined" and "not available".  Unclear waht the difference is


## scene geometry
Mesh geometry provided as MeshAnhors from sceneReconstructionProvider.

Useful for content placement or high-fidelity physics simulations

vertices, normals, faces, classifications (per face)
variety of types of objects.  fail case is 'none'
## image tracking
detect 2d images in the real world.  ImageTrackingProvider.
* specify a set of ReferenceImages to detect

load from ARResourceGroup, or provide via CVPixelBuffer or CGImage.
specify a set of `ReferenceImages` to detect
Detected images are ImageAnchors
useful for placing content at known, statically placed images

estimatedScaleFactor, reference image
# Hand tracking
anchors containing skeletal data for each of your hands.  
HandTrackingProvider
* each  hand is a HandAnchor
* trackable
* skeleton
* chirality (left, right)
* transform: wrist's transform relative to app origin

skeletons have joints queried by name.  parentJOint, name, localTransform, rootTransform, isTracked.

26 joints in hand skeleton.

wrist - root joint for the hand.  For each finger, each joint is parented to the wrist.  Subsequent joints are parented to the previous joint.

Useful for content placement or detecting custom gestures

poll for latest HandAnchors or receive HandAnchors when updates are available.



###  Hand tracking joints - 16:00
```swift
@available(xrOS 1.0, *)
public struct Skeleton : @unchecked Sendable, CustomStringConvertible {

    public func joint(named: SkeletonDefinition.JointName) -> Skeleton.Joint 

    public struct Joint : CustomStringConvertible, @unchecked Sendable {

        public var parentJoint: Skeleton.Joint? { get }

        public var name: String { get }

        public var localTransform: simd_float4x4 { get }

        public var rootTransform: simd_float4x4 { get }

        public var isTracked: Bool { get }
    }
}
```

###  Hand tracking with Render struct - 17:00
```swift
struct Renderer {
    ar_hand_tracking_provider_t  hand_tracking;
    struct {
        ar_hand_anchor_t left;
        ar_hand_anchor_t right;
    } hands;

    ...
};

void renderer_init(struct Renderer *renderer) {
    ...

    ar_hand_tracking_configuration_t hand_config = ar_hand_tracking_configuration_create();
    renderer->hand_tracking = ar_hand_tracking_provider_create(hand_config);

    ar_data_providers_t providers = ar_data_providers_create();
    ar_data_providers_add_data_provider(providers, renderer->world_tracking);
    ar_data_providers_add_data_provider(providers, renderer->hand_tracking);
    ar_session_run(renderer->session, providers);

    renderer->hands.left = ar_hand_anchor_create();
    renderer->hands.right = ar_hand_anchor_create();

    ...
}
```



###  Hand tracking call in render function - 17:25
```swift
void render(struct Renderer *renderer,
            ... ) {
    ...

    ar_hand_tracking_provider_get_latest_anchors(renderer->hand_tracking,
                                                 renderer->hands.left,
                                                 renderer->hands.right);

    if (ar_trackable_anchor_is_tracked(renderer->hands.left)) {
        const simd_float4x4 origin_from_wrist 
            = ar_anchor_get_origin_from_anchor_transform(renderer->hands.left);

        ...
    }

    ...
}
```


# Example

## app and view model
###  Demo app TimeForCube - 18:00
```swift
@main
struct TimeForCube: App {
   @StateObject var model = TimeForCubeViewModel()

    var body: some SwiftUI.Scene {
        ImmersiveSpace {
            RealityView { content in
                content.add(model.setupContentEntity())
            }
            .task {
                await model.runSession()
            }
            .task {
                await model.processHandUpdates()
            }
            .task {
                await model.processReconstructionUpdates()
            }
            .gesture(SpatialTapGesture().targetedToAnyEntity().onEnded({ value in
                let location3D = value.convert(value.location3D, from: .global, to: .scene)
                model.addCube(tapLocation: location3D)
            }))
        }
    }
}
```

###  Demo app View Model - 18:50
```swift
@MainActor class TimeForCubeViewModel: ObservableObject {
    private let session = ARKitSession()
    private let handTracking = HandTrackingProvider()
    private let sceneReconstruction = SceneReconstructionProvider()

    private var contentEntity = Entity()

    private var meshEntities = [UUID: ModelEntity]()

    private let fingerEntities: [HandAnchor.Chirality: ModelEntity] = [
        .left: .createFingertip(),
        .right: .createFingertip()
    ]

    func setupContentEntity() { ... }

    func runSession() async { ... }

    func processHandUpdates() async { ... }

    func processReconstructionUpdates() async { ... }

    func addCube(tapLocation: SIMD3<Float>) { ... }
}
```

## session initialization



## hand colliders
###  HandTrackingProvider function - 20:00
```swift
class TimeForCubeViewModel: ObservableObject {
    ...
    private let fingerEntities: [HandAnchor.Chirality: ModelEntity] = [
        .left: .createFingertip(),
        .right: .createFingertip()
    ]

    ...
    func processHandUpdates() async {
        for await update in handTracking.anchorUpdates {
            let handAnchor = update.anchor

            guard handAnchor.isTracked else { continue }

            let fingertip = handAnchor.skeleton.joint(named: .handIndexFingerTip)

            guard fingertip.isTracked else { continue }

            let originFromWrist = handAnchor.transform
            let wristFromIndex = fingertip.rootTransform
            let originFromIndex = originFromWrist * wristFromIndex

            fingerEntities[handAnchor.chirality]?.setTransformMatrix(originFromIndex,             relativeTo: nil)
        }
```

hand occlusion - see your hands on top of virtual content.  `.upperlimbVisibility(.hidden)`.  Now we see the entire sphere regardless of where our hands are.


## scene colliders


###  SceneReconstruction function - 21:20
```swift
func processReconstructionUpdates() async {
        for await update in sceneReconstruction.anchorUpdates {
            let meshAnchor = update.anchor
            
            guard let shape = try? await ShapeResource.generateStaticMesh(from: meshAnchor)             else { continue }
            
            switch update.event {
            case .added:
                let entity = ModelEntity()
                entity.transform = Transform(matrix: meshAnchor.transform)
                entity.collision = CollisionComponent(shapes: [shape], isStatic: true)
                entity.physicsBody = PhysicsBodyComponent()
                entity.components.set(InputTargetComponent())

                meshEntities[meshAnchor.id] = entity
                contentEntity.addChild(entity)
            case .updated:
                guard let entity = meshEntities[meshAnchor.id] else { fatalError("...") }
                entity.transform = Transform(matrix: meshAnchor.transform)
                entity.collision?.shapes = [shape]
            case .removed:
                meshEntities[meshAnchor.id]?.removeFromParent()
                meshEntities.removeValue(forKey: meshAnchor.id)
            @unknown default:
                fatalError("Unsupported anchor event")
            }
        }
    }
```

iterate over anchorUpdates.  Generat a shape resource and then switch on the anchor updates event.  If we're adding an chor, we create a new entity, set transform, add collision/physics body, add input target component, etc.

Add a new entity to our map, as a child of our content entity.

To update, we retrieve from map and update its transform/collision components shape.

removal - we remove the entity.
## cubes



###  Add cube at tap location - 22:20
```swift
class TimeForCubeViewModel: ObservableObject {
    func addCube(tapLocation: SIMD3<Float>) {
        let placementLocation = tapLocation + SIMD3<Float>(0, 0.2, 0)

        let entity = ModelEntity(
            mesh: .generateBox(size: 0.1, cornerRadius: 0.0),
            materials: [SimpleMaterial(color: .systemPink, isMetallic: false)],
            collisionShape: .generateBox(size: SIMD3<Float>(repeating: 0.1)),
            mass: 1.0)

        entity.setPosition(placementLocation, relativeTo: nil)
        entity.components.set(InputTargetComponent(allowedInputTypes: .indirect))

        let material = PhysicsMaterialResource.generate(friction: 0.8, restitution: 0.0)
        entity.components.set(PhysicsBodyComponent(shapes: entity.collision!.shapes,
                                                   mass: 1.0,
                                                   material: material,
                                                   mode: .dynamic))

        contentEntity.addChild(entity)
    }
}```

# Leanr more
[[Build spatial experiences with RealityKit]]
[[Evolve your ARKit app for spatial experiences]]
