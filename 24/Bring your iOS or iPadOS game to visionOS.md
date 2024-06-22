Discover how to transform your iOS or iPadOS game into a uniquely visionOS experience. Increase the immersion (and fun factor!) with a 3D frame or an immersive background. And invite players further into your world by adding depth to the window with stereoscopy or head tracking.

### Render with Metal in a UIView - 5:44
```swift
// Render with Metal in a UIView.
class CAMetalLayerBackedView: UIView, CAMetalDisplayLinkDelegate {
  var displayLink: CAMetalDisplayLink!
  
  override class var layerClass: AnyClass {
    return CAMetalLayer.self
  }
  
  func setup(device: MTLDevice) {
    let displayLink = CAMetalDisplayLink(metalLayer: self.layer as! CAMetalLayer)
    displayLink.add(to: .current, forMode: .default)
    self.displayLink.delegate = self
  }
  
  func metalDisplayLink(_ link: CAMetalDisplayLink, needsUpdate update: CAMetalDisplayLink.Update) {
    let drawable = update.drawable
    renderFunction?(drawable)
  }
}
```

### Render with Metal to a RealityKit LowLevelTexture - 6:20
```swift
// Render Metal to a RealityKit LowLevelTexture.
let lowLevelTexture = try! LowLevelTexture(descriptor: .init(
  pixelFormat: .rgba8Unorm,
  width: resolutionX,
  height: resolutionY,
  depth: 1,
  mipmapLevelCount: 1,
  textureUsage: [.renderTarget]
))

let textureResource = try! TextureResource(from: lowLevelTexture)
// assign textureResource to a material

let commandBuffer: MTLCommandBuffer = queue.makeCommandBuffer()!
let mtlTexture: MTLTexture = texture.replace(using: commandBuffer)
// Draw into the mtlTexture
```

### Metal viewport with a 3D RealityKit frame around it - 7:06
```swift
// Metal viewport with a 3D RealityKit frame
// around it.
struct ContentView: View {
  @State var game = Game()
  var body: some View {
    ZStack {
      CAMetalLayerView { drawable in
        game.render(drawable)
      }
      RealityView { content in
        content.add(try! await Entity(named: "Frame"))
      }.frame(depth: 0)
    }
  }
}
```

### Windowed game with an immersive background - 7:45
```swift
// Windowed game with an immersive background
@main
struct TestApp: App {
  @State private var appModel = AppModel()
  
  var body: some Scene {
    WindowGroup {
      // Metal render
      ContentView(appModel)
    }
    ImmersiveSpace(id: "ImmersiveSpace") {
      // RealityKit background
      ImmersiveView(appModel)
    }.immersionStyle(selection: .constant(.progressive), in: .progressive)
  }
}
```

### Render to multiple views for stereoscopy - 13:11
```swift
// Render to multiple views for stereoscopy.
override func draw(provider: DrawableProviding) {
  encodeShadowMapPass()
  
  for viewIndex in 0..<provider.viewCount {
    scene.update(
      viewMatrix: provider.viewMatrix(viewIndex: viewIndex),
      projectionMatrix: provider.projectionMatrix(viewIndex: viewIndex)
    )
    
    var commandBuffer = beginDrawableCommands()
    
    if let color = provider.colorTexture(viewIndex: viewIndex, for: commandBuffer),
       let depthStencil = provider.depthStencilTexture(viewIndex: viewIndex, for: commandBuffer) {
      encodePass(into: commandBuffer, color: color, depth: depth)
    }
    
    endFrame(commandBuffer)
  }
}
```

### Query the head position from ARKit every frame - 13:55
```swift
// Query the head position from ARKit every frame.
import ARKit

let arSession = ARKitSession()
let worldTracking = WorldTrackingProvider()
try await arSession.run([worldTracking])

// Every frame
guard let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: CACurrentMediaTime() + presentationTime) else { return }

let transform: simd_float4x4 = deviceAnchor.originFromAnchorTransform
```

### Convert the head position from the ImmersiveSpace to a window - 14:22
```swift
// Convert the head position from the ImmersiveSpace to a window.
let headPositionInImmersiveSpace: SIMD3<Float> = deviceAnchor.originFromAnchorTransform.position

let windowInImmersiveSpace: float4x4 = windowEntity.transformMatrix(relativeTo: .immersiveSpace)

let headPositionInWindow: SIMD3<Float> = windowInImmersiveSpace.inverse.transform(headPositionInImmersiveSpace)

renderer.setCameraPosition(headPositionInWindow)
```

### Query the head position from ARKit every frame - 15:05
```swift
// Query the head position from ARKit every frame.
import ARKit

let arSession = ARKitSession()
let worldTracking = WorldTrackingProvider()
try await arSession.run([worldTracking])

// Every frame
guard let deviceAnchor = worldTracking.queryDeviceAnchor(atTimestamp: CACurrentMediaTime() + presentationTime) else { return }

let transform: simd_float4x4 = deviceAnchor.originFromAnchorTransform
```

### Build the camera and projection matrices - 15:47
```swift
// Build the camera and projection matrices.
let cameraPosition: SIMD3<Float>
let viewportBounds: BoundingBox

// Camera facing -Z
let cameraTransform = simd_float4x4(AffineTransform3D(translation: Size3D(cameraPosition)))

let zNear: Float = viewportBounds.max.z - cameraPosition.z

let l /* left */: Float = viewportBounds.min.x - cameraPosition.x
let r /* right */: Float = viewportBounds.max.x - cameraPosition.x
let b /* bottom */: Float = viewportBounds.min.y - cameraPosition.y
let t /* top */: Float = viewportBounds.max.y - cameraPosition.y

let cameraProjection = simd_float4x4(rows: [
  [2 * zNear / (r - l), 0, (r + l) / (r - l), 0],
  [0, 2 * zNear / (t - b), (t + b) / (t - b), 0],
  [0, 0, 1, -zNear],
  [0, 0, 1, 0]
])
```

# Resources
https://developer.apple.com/documentation/RealityKit/rendering-a-windowed-game-in-stereo
