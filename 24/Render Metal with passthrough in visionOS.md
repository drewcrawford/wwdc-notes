Get ready to extend your Metal experiences for visionOS. Learn best practices for integrating your rendered content with people's physical environments with passthrough. Find out how to position rendered content to match the physical world, reduce latency with trackable anchor prediction, and more.
### Add mixed immersion - 3:07
```swift
@main struct MyApp: App { 
    var body: some Scene { 
        ImmersiveSpace { 
            CompositorLayer(configuration: MyConfiguration()) { layerRenderer in 
                let engine = my_engine_create(layerRenderer) 
                let renderThread = Thread { my_engine_render_loop(engine) } 
                renderThread.name = "Render Thread" 
                renderThread.start() 
            } 
            .immersionStyle(selection: $style, in: .mixed, .full) 
        } 
    } 
}
```

### Create a renderPassDescriptor - 4:43
```swift
let renderPassDescriptor = MTLRenderPassDescriptor() 
renderPassDescriptor.colorAttachments[0].texture = drawable.colorTextures[0] 
renderPassDescriptor.colorAttachments[0].loadAction = .clear 
renderPassDescriptor.colorAttachments[0].storeAction = .store 
renderPassDescriptor.colorAttachments[0].clearColor = .init(red: 0.0, green: 0.0, blue: 0.0, alpha: 0.0) 
renderPassDescriptor.depthAttachment.texture = drawable.depthTextures[0] 
renderPassDescriptor.depthAttachment.loadAction = .clear 
renderPassDescriptor.depthAttachment.storeAction = .store 
renderPassDescriptor.depthAttachment.clearDepth = 0.0
```

### Set Upper Limb Visibility - 9:08
```swift
@main struct MyApp: App { 
    var body: some Scene { 
        ImmersiveSpace { 
            CompositorLayer(configuration: MyConfiguration()) { layerRenderer in 
                let engine = my_engine_create(layerRenderer) 
                let renderThread = Thread { my_engine_render_loop(engine) } 
                renderThread.name = "Render Thread" 
                renderThread.start() 
            } 
            .immersionStyle(selection: $style, in: .mixed, .full) 
            .upperLimbVisiblity(.automatic) 
        } 
    } 
}
```

### Compose a projection view matrix - 13:37
```swift
func renderLoop { 
    //... 
    let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: presentationTime) 
    drawable.deviceAnchor = deviceAnchor 
    for viewIndex in 0...drawable.views.count { 
        let view = drawable.views[viewIndex] 
        let originFromDevice = deviceAnchor?.originFromAnchorTransform 
        let deviceFromView = view.transform 
        let viewMatrix = (originFromDevice * deviceFromView).inverse 
        let projection = drawable.computeProjection(normalizedDeviceCoordinatesConvention: .rightUpBack, viewIndex: viewIndex) 
        let projectionViewMatrix = projection * viewMatrix; 
        //... 
    } 
}
```

### Trackable anchor prediction - 18:27
```swift
func renderFrame() { 
    //... 
    let presentationTime = drawable.frameTiming.presentationTime 
    let trackableAnchorTime = drawable.frameTiming.trackableAnchorTime 
    let devicePredictionTime = LayerRenderer.Clock.Instant.epoch.duration(to: presentationTime).timeInterval 
    let anchorPredictionTime = LayerRenderer.Clock.Instant.epoch.duration(to: trackableAnchorTime).timeInterval 
    let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: devicePredictionTime) 
    let leftAnchor = handTracking.handAnchors(at: anchorPredictionTime) 
    if (leftAnchor.isTracked) { 
        //... 
    }
```
# Resources
* https://developer.apple.com/news/?id=5cda5ipr
* https://developer.apple.com/documentation/metal/render_passes/improving_rendering_performance_with_vertex_amplification
* https://developer.apple.com/metal/
* https://developer.apple.com/documentation/metal/metal_sample_code_library/rendering_a_scene_with_deferred_lighting_in_swift
* https://developer.apple.com/documentation/metal/render_passes/rendering_at_different_rasterization_rates
* 