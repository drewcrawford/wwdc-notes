Create 3d models fo real-world objects and bring them into AR.

# Object Capture recap
Leverage to easily turn images of real-world objects into detailed 3d models.  Taking photos of your object from various angles with iPhone, Ipad, or DSLR.  Then copy those photos to a mac which supports OC.

Using Photogrammetry API, #realitykit  translates to a 3d model.  Includes both geometric mesh and various material maps.

[[Create 3D models with Object Capture]]

Amazing 3d capture apps.  Unity, Cinema4d, etc.

We are thrilled to see stunning and widespready use.

# ARKit camera enhancements
A great OC experience starts with taking good photos of objects from all sides.
Use any high-res camera.
iPHone/iPad for automatic scale and orientation
Leverage ARKit for 3d guidance UI
Higher the image resolution, better the quality of the output model

## Higher-resolution background photos
* capture photos at full camera resolution
* Uninterrupted video stream of current ARSession. #arkit 
* Provides EXIF tags in photos

```swift
if let hiResCaptureVideoFormat = ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing {
    // Assign the video format that supports hi-res capturing.
config.videoFormat = hiResCaptureVideoFormat
}
// Run the session.
session.run(config)

session.captureHighResolutionFrame { (frame, error) in
   if let frame = frame {
      // save frame.capturedImage 
      // …   
   }
}
```

## Precise camera control
* get access to AVCAptureDevice for fine-grained control
`configurableCaptureDeviceForPrimaryCamera`.

[[Discover ARKit 6]]


# Best practice guidelines
## Choosing good objects
* sufficient texture detail
* minimal reflective surfaces
* rigid (when flipping)
* limited fine structure

## Setting up the environment
* diffuse, consistent lighting
* ensure enough space around object

## Demo
## Detail levels
reduced, medium => web-based, mobile experiences.
full,raw => pro.

reduced,medium,full are baked materials
raw is unbaked.


# End to end workflow
Bring real-life objects to AR using RK.  Let's dive into a demo.

Refer to [[Create 3D models with Object Capture]]

`PhotogrammetrySession` API.

#realityconverter to invert colors of our chess pieces.  So we have a lighter and darker version.

[[Dive into RealityKit 2]]
[[Explore advanced rendering with RealityKit]]

## Startup animation
* translate and scale entities without animation
* animate back to original transform

```swift
// Board Animation
class Chessboard: Entity {
    func playAnimation() {
        checkers
            .forEach { entity in
                let currentTransform = entity.transform
        // Move checker square 10cm up
                entity.transform.translation += SIMD3<Float>(0, 0.1, 0)
                entity.move(to: currentTransform,
                    relativeTo: entity.parent,
                    duration: BoardGame.startupAnimationDuration)
            }
        
        // Play built-in animation for board border
        border.availableAnimations.forEach {
            border.playAnimation($0)
        }
    }
}
```

We want to select chess pieces.  #arkit 

```swift
// Select chess piece
class ChessViewport: ARView {
    @objc
    func handleTap(sender: UITapGestureRecognizer) {
        guard let ray = ray(through: sender.location(in: self)) else { return }

        // No piece is selected yet, we want to select one
        guard let raycastResult = scene.raycast(origin: ray.origin,
                                                direction: ray.direction,
                                                length: 5,
                                                query: .nearest,
                                                mask: .piece).first,
              let piece = raycastResult.entity.parentChessPiece else {
            return
        }
        boardGame.select(piece)
        gameManager.selectedPiece = piece
    }
}
```

raycast ignores all entities without collision component.

## Glow effect using a surface shader
Calculate smaterial parameters through metal.  Called by #realitykit 's fragment shader for each pixel.


(seems we are missing this code)

## Geometry modifier
Change vertex data such as position, normal, texture coordinates, etc.  Each metal functions called once per vertex by RK shader.

Modifications are purely transiet and do not affect vertex information.


```swift
// Capture Geometry Modifier
class ChessPiece: Entity, HasChessPiece {
    var capturedProgress: Float
        get {
            (pieceEntity?.model?.materials.first as? CustomMaterial)?.custom.value[0] ?? 0
        }
        set {
            pieceEntity?.modifyMaterials { material in
                guard var customMaterial = material as? CustomMaterial else {
                    return material
                }
                customMaterial.custom.value = SIMD4<Float>(newValue, 0, 0, 0)
                return customMaterial
            }
        }
    }
}
```

`custom` proprerty lets us share data between CPU/GPU.  This passes the progress to the modifier.

(we seem to be missing the metal shader code)

## Highlight potential moves using bloom.
Apply a pulsing effect with a surface shader.  Post-process with bloom.
```cpp
// Checker animation to show potential moves
void checkerSurface(realitykit::surface_parameters params,
                    float amplitude,
                    bool isBlack = false)
{
    // ...
    bool isPossibleMove = params.uniforms().custom_parameter()[0];
    if (isPossibleMove) {
        const float a = amplitude * sin(params.uniforms().time() * M_PI_F) + amplitude;
        params.surface().set_emissive_color(half3(a));
        if (isBlack) {
            params.surface().set_base_color(half3(a));
        }
    }
}
```
seem to be some code missing here as well
```swift
import MetalPerformanceShaders

class ChessViewport: ARView {
    init(gameManager: GameManager) {
        /// ...
        renderCallbacks.postProcess = postEffectBloom
    }

    func postEffectBloom(context: ARView.PostProcessContext) {
        let brightness = MPSImageThresholdToZero(device: context.device,
                                                 thresholdValue: 0.85,
                                                 linearGrayColorTransform: nil)
        brightness.encode(commandBuffer: context.commandBuffer,
                          sourceTexture: context.sourceColorTexture,
                          destinationTexture: bloomTexture!)
        /// ...
    }
}
```

# Wrap up
* Object capture recap
* ARKit camera enhancements
* Best practice guidelines
* End to end workflow





* https://developer.apple.com/forums/tags/wwdc2022-10128
* https://developer.apple.com/documentation/realitykit/using_object_capture_assets_in_realitykit
* https://developer.apple.com/documentation/realitykit/creating_a_photogrammetry_command-line_app
* https://developer.apple.com/documentation/realitykit/taking_pictures_for_3d_object_capture
* https://developer.apple.com/documentation/RealityKit/capturing-photographs-for-realitykit-object-capture
* https://developer.apple.com/documentation/realitykit/building_an_immersive_experience_with_realitykit
* https://developer.apple.com/documentation/RealityKit
* https://developer.apple.com/documentation/realitykit/creating_a_game_with_reality_composer
