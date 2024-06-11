Discover how the Timeline view in Reality Composer Pro can bring your 3D content to life. Learn how to create an animated story in which characters and objects interact with each other and the world around them using inverse kinematics, blend shapes, and skeletal poses. We'll also show you how to use built-in and custom actions, sequence your actions, apply triggers, and implement natural movements.

### Setup IKComponent - 20:31
```swift
// Setup IKComponent
import RealityKit

struct HeroRobotRuntimeComponent: Component {
    var rig = try? IKRig(for: modelSkeleton)
    rig.maxIterations = 30
    rig.globalFkWeight = 0.02
    
    let hipsJointName = "root/hips"
    let chestJointName = "root/hips/spine1/spine2/chest"
    let leftHandJointName = "root/hips/spine1/spine2/chest/…/L_arm3/L_arm4/L_arm5/L_wrist"
    
    rig.constraints = [
        .parent(named: "hips_constraint", on: hipsJointName, positionWeight: SIMD3(repeating: 90.0), orientationWeight: SIMD3(repeating: 90.0)),
        .parent(named: "chest_constraint", on: chestJointName, positionWeight: SIMD3(repeating: 120.0), orientationWeight: SIMD3(repeating: 120.0)),
        .point(named: "left_hand_constraint", on: leftHandJointName, positionWeight: SIMD3(repeating: 10.0))
    ]
    
    let resource = try? IKResource(rig: rig)
    modelComponentEntity.components.set(IKComponent(resource: resource))
}
```

### Update IKComponent - 21:33
```swift
// Update IKComponent
import RealityKit

struct HeroRobotRuntimeComponent: Component {
    guard let reachTarget = sceneRoot.findEntity(named: "reachTargetName") else { return }
    var reachPosition = reachTarget.position(relativeTo: entity)
    
    let time = sin(simTime)
    reachPosition.x += (20.0 + 50.0 * time)
    reachPosition.y += (40.0 + 30.0 * abs(time))
    reachPosition.z += (20.0 + 20.0 * abs(time))
    
    guard let ikComponent = modelComponentEntity.components[IKComponent.self] else { return }
    var reachPosition = reachTarget.position(relativeTo: entity)

    // Rest of the code snippet goes here
    
    modelComponentEntity.components.set(ikComponent)
}
```

### Sequence and play animation actions - 24:36
```swift
// Play Animation Actions
import RealityKit

struct HeroRobotRuntimeComponent: Component {
    let rotateAnimationResource = createRotateAnimationResource()
    let walkAndMoveAnimationGroup = createWalkAndMoveAnimationGroup()
    let alignAtHomeActionResource = createAlignAtHomeActionResource()
    let robotTravelHomeCompleteActionResource = createRobotTravelHomeCompleteAction()

    let moveHomeSequence = try? AnimationResource.sequence(with: [rotateAnimationResource, walkAndMoveAnimationGroup, alignAtHomeActionResource, robotTravelHomeCompleteActionResource])
    
    _ = robotEntity.playAnimation(moveHomeSequence)
}
```

### Setup EntityActions - 25:59
```swift
// Setup EntityActions
import RealityKit

struct HeroRobotRuntimeComponent: Component {
    struct RobotMoveToHomeComplete: EntityAction {
        var animatedValueType: (any AnimatableData.Type)? { nil }
    }
    
    let travelCompleteAction = RobotMoveToHomeComplete()
    let actionResource = try! AnimationResource.makeActionAnimation(for: travelCompleteAction, duration: 0.1)
    
    let _ = robotEntity.playAnimation(actionResource)
}
```

### EntityAction subscription - 26:39
```swift
// EntityAction subscription
import RealityKit

struct HeroRobotRuntimeComponent: Component {
    public enum HeroRobotState: String, Codable {
        case available
        case arrivedHome
    }
    
    RobotMoveToHomeComplete.subscribe(to: .started) { event in
        if event.playbackController.entity != nil {
            event.playbackController.stop()
        }
    }
    
    RobotMoveToHomeComplete.subscribe(to: .ended) { event in
        if let robotEntity = event.playbackController.entity, var component = robotEntity.components[HeroRobotRuntimeComponent.self] {
            component.setState(newState:.arrivedHome)
        }
    }
}
```

### Setup BlendshapeWeightsComponent - 29:17
```swift
// Setup BlendShapeWeightsComponent
import RealityKit

struct HeroPlantComponent: Component, Codable {
    guard let modelComponentEntity = findModelComponentEntity(entity: entity), let modelComponent = modelComponentEntity.components[ModelComponent.self] else { return }
    
    let blendShapeWeightsMapping = BlendShapeWeightsMapping(meshResource: modelComponent.mesh)
    entity.components.set(BlendShapeWeightsComponent(weightsMapping: blendShapeWeightsMapping))
}
```

### Update BlendshapeWeightsComponent - 29:38
```swift
// Update BlendShapeWeightsComponent
struct HeroPlantComponent: Component, Codable {
    guard let component = entity.components[BlendShapeWeightsComponent.self] else { return }
    
    var blendWeightSet = blendShapeComponent.weightSet
    
    for weightIndex in 0..<blendWeightSet[blendWeightsIndex].weights.count {
        blendWeightSet[blendWeightsIndex].weights[weightIndex] = 0.0
    }
    
    for index in 0..<blendWeightSet.count {
        component?.weightSet[blendWeightsIndex].weights = blendWeightSet[index].weights
    }
}
```

### Setup and update Skeletal Poses - 32:01
```swift
// Update Skeletal Poses
import RealityKit

struct StationaryRobotRuntimeComponent: Component {
    guard var component = entity.components[SkeletalPosesComponent.self] else { return }
    
    let neckRotation = calculateRotation()
    component.poses.default?.jointTransforms[neckJointIndex].rotation = neckRotation
}
```
# Resources
* https://developer.apple.com/documentation/RealityKit/composing-interactive-3d-content-with-realitykit-and-reality-composer-pro
