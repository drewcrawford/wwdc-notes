Find out how you can use object tracking to turn real-world objects into virtual anchors in your visionOS app. Learn how you can build spatial experiences with object tracking from start to finish. Find out how to create a reference object using machine learning in Create ML and attach content relative to your target object in Reality Composer Pro, RealityKit or ARKit APIs.

### Coaching UI - display object USDZ preview - 13:55
```swift
// Display object USDZ
struct ImmersiveView: View {
    @State var globeAnchor: Entity? = nil
    
    var body: some View {
        RealityView { content in
            // Load the reference object with ARKit API
            let refObjURL = Bundle.main.url(forResource: "globe", withExtension: ".referenceobject")
            let refObject = try? await ReferenceObject(from: refObjURL!)
            
            // Load the model entity with USDZ path extracted from reference object
            let globePreviewEntity = try? await Entity.init(contentsOf: (refObject?.usdzFile)!)
            
            // Set opacity to 0.5 and add to scene
            globePreviewEntity!.components.set(OpacityComponent(opacity: 0.5))
            content.add(globePreviewEntity!)
        }
    }
}
```

### Coaching UI - check anchor state - 14:13
```swift
// Check anchor state
struct ImmersiveView: View {
    @State var globeAnchor: Entity? = nil
    
    var body: some View {
        RealityView { content in
            if let scene = try? await Entity(named: "Immersive", in: realityKitContentBundle) {
                globeAnchor = scene.findEntity(named: "GlobeAnchor")
                content.add(scene)
            }
            
            let updateSub = content.subscribe(to: SceneEvents.Update.self) {event in
                if let anchor = globeAnchor, anchor.isAnchored {
                    // Object Anchor found, trigger transition animation
                } else {
                    // Object Anchor not found, display coaching UI
                }
            }
        }
    }
}
```

### Coaching UI - Transform space with SpatialSession - 14:31
```swift
// Transform space
struct ImmersiveView: View {
    @State var globeAnchor: Entity? = nil
    
    var body: some View {
        RealityView { content in
            // Setup anchor transform space for object and world anchor
            let trackingSession = SpatialTrackingSession()
            let config = SpatialTrackingSession.Configuration(tracking: [.object, .world])
            
            if let result = await trackingSession.run(config) {
                if result.anchor.contains(.object) {
                    // Tracking not authorized, adjust experience accordingly
                }
            }
            
            // Get tracked object's world transform, identity if tracking not authorized
            let objectTransform = globeAnchor?.transformMatrix(relativeTo: nil)
            
            // Implement animation ...
        }
    }
}
```

# Resources
https://developer.apple.com/documentation/visionOS/exploring_object_tracking_with_arkit

