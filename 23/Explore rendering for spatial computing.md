Find out how you can take control of RealityKit rendering to improve the look and feel of your apps and games on visionOS. Discover how you can customize lighting, add grounding shadows, and control tone mapping for your content. We'll also go over best practices for two key treatments on the platform: rasterization rate maps and dynamic content scaling.
### 3:05 - Image based lighting
```swift
RealityView { content in
    async let satellite = Entity(named: "Satellite", in: worldAssetsBundle)
    async let environment = EnvironmentResource(named: "Sunlight")

    if let satellite = try? await satellite, let environment = try? await environment {
        content.add(satellite)

        satellite.components.set(ImageBasedLightComponent(
           source: .single(environment)))

        satellite.components.set(ImageBasedLightReceiverComponent(
           imageBasedLight: satellite))
   }
}
```

### 4:28 - Grounding shadows
```swift
RealityView { content in
    if let vase = try? await Entity(named: "flower_tulip") {
        content.add(vase)

        vase.components.set(GroundingShadowComponent(castsShadow: true))
    }
}
```

### 8:48 - Disable tone mapping
```swift
RealityView { content in
    if let trafficLight = try? await Entity(named: "traffic_light") {
        content.add(trafficLight)

        if let lamp = trafficLight.findEntity(named: "red_light") {
            if var model = lamp.components[ModelComponent.self] {
                let material = UnlitMaterial(color: .init(color), 
                                             applyPostProcessToneMap: false)

                model.materials = [material]

                lamp.components[ModelComponent.self] = model
            }
        }
    }
}
```

### 15:34 - Dynamic content scaling
```swift
// Enable dynamic content scaling on CALayer with:

var wantsDynamicContentScaling: Bool { get set }
```
# Resources
* https://developer.apple.com/documentation/metal/render_passes/rendering_at_different_rasterization_rates
