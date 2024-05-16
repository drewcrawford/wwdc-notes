Find out how you can take control of RealityKit rendering to improve the look and feel of your apps and games on visionOS. Discover how you can customize lighting, add grounding shadows, and control tone mapping for your content. We'll also go over best practices for two key treatments on the platform: rasterization rate maps and dynamic content scaling.

# Lighting and shadows

Two main components to IBL
* environment probe texture provided by ARKit.
* System IBL texture.  Packaged with OS.  This adds extra . to ensure content looks great in any environment
* These are combined to form the combined IBL texture

This year you can customize system IBL texture.

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

Add these receiver components to other entities, etc.  To light them using the same IBL.

By simply adding RealkitKit's grounding shadow, it becomes a lot clearer that the vase is above the center of the plane.


### 4:28 - Grounding shadows
```swift
RealityView { content in
    if let vase = try? await Entity(named: "flower_tulip") {
        content.add(vase)

        vase.components.set(GroundingShadowComponent(castsShadow: true))
    }
}
```


# materials

Can also tweak materials.  

PhysicallyBasedMaterial.  Reacts to lighting, can be used to represent a variety of real-world materials.

SimpleMaterial -> smaller subset of parameters.  Quick experiments.

UnlitMaterial -> doesn't react.  Constant look under changing lighting conditions
VideoMaterial -> variation of unlit, that can map a movie file.

Now supports ShaderGraphMaterial.  Author in RC pro or load via MaterialX file.

[[Explore materials in Reality Composer Pro]]

tone mapping
* enabled by default in realitykit
* Allows more natural perceived colors
* Re-mapping values above 1 into the visible range

If you want to display exact object colros, opt out of one-mapping.

Keep in mind that unlitMaterial is still affected by tone mapping.  So even if the same color is assigned to swiftui control and material, they're different.  Disable tone mapping to make them match.




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

Can also do this in RC pro.

# Rasterization rate maps

OS must update displays many fps.

Fewer calculations are performed in areas that are darkened.  

Significant memory and performance savings.

In RK we automatically enable this optimization.  In some situations you may have to adjust your content to make it work well with this optimization.

Flickering demo.  Yellow circle representing eye direction.  

Triangles too small. Reduce flickering by making triangles larger and store fine details in opacity texture.

Low-resolution mipmaps are used.  For more details on rasterization rate maps, see the article "Rendering at different rasterization rates".

# Dynamic content scaling

Rasterization at varying levels of detail depending on what they eys are looking at.

Draw UI content at the right scale.  

Affects the relative size in memory of rasterized content.  Text labels are different sizes depending on how close they are to the point where your eyes are looking.

| content type   | dynamic scaling |
| -------------- | --------------- |
| UIKit, swiftUI | Enabled         |
| Core animation | available       |

 Enable on CALayer:
### 15:34 - Dynamic content scaling
```swift
// Enable dynamic content scaling on CALayer with:

var wantsDynamicContentScaling: Bool { get set }
```

This technique relies on rasterizing at high resolution, not recommended with primarily bitmap-based content

see docs


# Wrap up

* Lighting and shadows
* materials
* Rasterization rate maps
* Dynamic content scaling
* 
# Resources
* https://developer.apple.com/documentation/metal/render_passes/rendering_at_different_rasterization_rates
