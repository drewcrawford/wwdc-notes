Harness the power of RealityKit through the process of building a spatial drawing app. As you create an eye-catching spatial experience that integrates RealityKit with ARKit and SwiftUI, you'll explore how resources work in RealityKit and how to use features like low-level mesh and texture APIs to achieve fast updates of the users' brush strokes.

### 4:18 - Using SpatialTrackingSession
```swift
// Retain the SpatialTrackingSession while your app needs access
let session = SpatialTrackingSession()

// Declare needed tracking capabilities
let configuration = SpatialTrackingSession.Configuration(tracking: [.hand])

// Request authorization for spatial tracking
let unapprovedCapabilities = await session.run(configuration)

if let unapprovedCapabilities, unapprovedCapabilities.anchor.contains(.hand) {
    // User has rejected hand data for your app.
    // AnchorEntities will continue to remain anchored and update visually
    // However, AnchorEntity.transform will not receive updates
} else {
    // User has approved hand data for your app.
    // AnchorEntity.transform will report hand anchor pose
}
```

### 7:07 - Use MeshResource extrusion
```swift
// Use MeshResource(extruding:) to generate the canvas edge
let path = SwiftUI.Path { path in
   // Generate two concentric circles as a SwiftUI.Path
   path.addArc(center: .zero, radius: outerRadius, startAngle: .degrees(0), endAngle: .degrees(360), clockwise: true)
   path.addArc(center: .zero, radius: innerRadius, startAngle: .degrees(0), endAngle: .degrees(360), clockwise: true)
}.normalized(eoFill: true)

var options = MeshResource.ShapeExtrusionOptions()
options.boundaryResolution = .uniformSegmentsPerSpan(segmentCount: 64)
options.extrusionMethod = .linear(depth: extrusionDepth)

return try MeshResource(extruding: path, extrusionOptions: extrusionOptions)
```

### 9:33 - Highlight HoverEffectComponent
```swift
// Use HoverEffectComponent with .highlight
let placementEntity: Entity = // ...

let hover = HoverEffectComponent(
    .highlight(.init(
        color: UIColor(/* ... */),
        strength: 5.0
    ))
)

placementEntity.components.set(hover)
```

### 9:54 - Using Blend Modes
```swift
// Create an UnlitMaterial with Additive Blend Mode
var descriptor = UnlitMaterial.Program.Descriptor()
descriptor.blendMode = .add

let prog = await UnlitMaterial.Program(descriptor: descriptor)
var material = UnlitMaterial(program: prog)
material.color = UnlitMaterial.BaseColor(tint: UIColor(/* ... */))
```

### 13:45 - Shader based hover effects
```swift
// Use shader-based hover effects
let hoverEffectComponent = HoverEffectComponent(.shader(.default))
entity.components.set(hoverEffectComponent)

let material = try await ShaderGraphMaterial(named: "/Root/SolidPresetBrushMaterial", from: "PresetBrushMaterial", in: realityKitContentBundle)
entity.components.set(ModelComponent(mesh: /* ... */, materials: [material]))
```

### 16:56 - Defining a vertex buffer struct for the solid brush
```swift
struct SolidBrushVertex {
   packed_float3 position;
   packed_float3 normal;
   packed_float3 bitangent;
   packed_float2 materialProperties;
   float curveDistance;
   packed_half3 color;
};
```

### 19:27 - Defining LowLevelMesh Attributes for solid brush
```swift
extension SolidBrushVertex {
   static var vertexAttributes: [LowLevelMesh.Attribute] {
       typealias Attribute = LowLevelMesh.Attribute
       return [
           Attribute(semantic: .position, format: MTLVertexFormat.float3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.position)!),
           Attribute(semantic: .normal, format: MTLVertexFormat.float3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.normal)!),
           Attribute(semantic: .bitangent, format: MTLVertexFormat.float3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.bitangent)!),
           Attribute(semantic: .color, format: MTLVertexFormat.half3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.color)!),
           Attribute(semantic: .uv1, format: MTLVertexFormat.float, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.curveDistance)!),
           Attribute(semantic: .uv3, format: MTLVertexFormat.float2, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.materialProperties)!)
       ]
   }
}
```

### 21:14 - Make LowLevelMesh
```swift
private static func makeLowLevelMesh(vertexBufferSize: Int, indexBufferSize: Int, meshBounds: BoundingBox) throws -> LowLevelMesh {
   var descriptor = LowLevelMesh.Descriptor()

   // Similar to MTLVertexDescriptor
   descriptor.vertexCapacity = vertexBufferSize
   descriptor.indexCapacity = indexBufferSize
   descriptor.vertexAttributes = SolidBrushVertex.vertexAttributes

   let stride = MemoryLayout<SolidBrushVertex>.stride
   descriptor.vertexLayouts = [LowLevelMesh.Layout(bufferIndex: 0, bufferOffset: 0, bufferStride: stride)]

   let mesh = try LowLevelMesh(descriptor: descriptor)
   mesh.parts.append(LowLevelMesh.Part(indexOffset: 0, indexCount: indexBufferSize, topology: .triangleStrip, materialIndex: 0, bounds: meshBounds))

   return mesh
}
```

### 22:28 - Creating a MeshResource
```swift
let mesh: LowLevelMesh
let resource = try MeshResource(from: mesh)
entity.components[ModelComponent.self] = ModelComponent(mesh: resource, materials: [...])
```

### 22:37 - Updating vertex data of LowLevelMesh using withUnsafeMutableBytes API
```swift
let mesh: LowLevelMesh
mesh.withUnsafeMutableBytes(bufferIndex: 0) { buffer in
   let vertices: UnsafeMutableBufferPointer<SolidBrushVertex> = buffer.bindMemory(to: SolidBrushVertex.self)
   // Write to vertex buffer `vertices`
}
```

### 23:07 - Updating LowLevelMesh index buffers using withUnsafeMutableBytes API
```swift
let mesh: LowLevelMesh
mesh.withUnsafeMutableIndices { buffer in
   let indices: UnsafeMutableBufferPointer<UInt32> = buffer.bindMemory(to: UInt32.self)
   // Write to index buffer `indices`
}
```

### 23:58 - Creating a particle brush using LowLevelMesh
```swift
struct SparkleBrushAttributes {
   packed_float3 position;
   packed_half3 color;
   float curveDistance;
   float size;
};

// Describes a particle in the simulation
struct SparkleBrushParticle {
   struct SparkleBrushAttributes attributes;
   packed_float3 velocity;
};

// One quad (4 vertices) is created per particle
struct SparkleBrushVertex {
   struct SparkleBrushAttributes attributes;
   simd_half2 uv;
};
```

### 24:58 - Defining LowLevelMesh Attributes for sparkle brush
```swift
extension SparkleBrushVertex {
   static var vertexAttributes: [LowLevelMesh.Attribute] {
       typealias Attribute = LowLevelMesh.Attribute
       return [
           Attribute(semantic: .position, format: .float3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.attributes.position)!),
           Attribute(semantic: .color, format: .half3, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.attributes.color)!),
           Attribute(semantic: .uv0, format: .half2, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.uv)!),
           Attribute(semantic: .uv1, format: .float, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.attributes.curveDistance)!),
           Attribute(semantic: .uv2, format: .float, layoutIndex: 0, offset: MemoryLayout.offset(of: \Self.attributes.size)!)
       ]
   }
}
```

### 25:28 - Populate LowLevelMesh on GPU
```swift
let inputParticleBuffer: MTLBuffer
let lowLevelMesh: LowLevelMesh
let commandBuffer: MTLCommandBuffer
let encoder: MTLComputeCommandEncoder
let populatePipeline: MTLComputePipelineState

commandBuffer.enqueue()
encoder.setComputePipelineState(populatePipeline)

let vertexBuffer: MTLBuffer = lowLevelMesh.replace(bufferIndex: 0, using: commandBuffer)
encoder.setBuffer(inputParticleBuffer, offset: 0, index: 0)
encoder.setBuffer(vertexBuffer, offset: 0, index: 1)

encoder.dispatchThreadgroups(/* ... */)
// ...

encoder.endEncoding()
commandBuffer.commit()
```

### 27:01 - Use MeshResource extrusion to generate 3D text
```swift
// Use MeshResource(extruding:) to generate 3D text
var textString = AttributedString("RealityKit")
textString.font = .systemFont(ofSize: 8.0)

let secondLineFont = UIFont(name: "ArialRoundedMTBold", size: 14.0)
let attributes = AttributeContainer([.font: secondLineFont])
textString.append(AttributedString("\nDrawing App", attributes: attributes))

let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.alignment = .center
let centerAttributes = AttributeContainer([.paragraphStyle: paragraphStyle])
textString.mergeAttributes(centerAttributes)

var extrusionOptions = MeshResource.ShapeExtrusionOptions()
extrusionOptions.extrusionMethod = .linear(depth: 2)
extrusionOptions.materialAssignment = .init(front: 0, back: 0, extrusion: 1, frontChamfer: 1, backChamfer: 1)
extrusionOptions.chamferRadius = 0.1

let textMesh = try await MeshResource(extruding: textString, extrusionOptions: extrusionOptions)
```

### 28:25 - Use MeshResource extrusion to turn a SwiftUI Path into 3D mesh
```swift
// Use MeshResource(extruding:) to bring SwiftUI.Path to 3D
let graphic = SwiftUI.Path { path in
   path.move(to: CGPoint(x: -0.7, y: 0.135413))
   path.addCurve(to: CGPoint(x: -0.7, y: 0.042066), control1: CGPoint(x: -0.85, y: 0.067707), control2: CGPoint(x: -0.85, y: 0.021033))
   // ...
}

var options = MeshResource.ShapeExtrusionOptions()
// ...

let graphicMesh = try await MeshResource(extruding: graphic, extrusionOptions: options)
```

### 29:44 - Defining a LowLevelTexture
```swift
let descriptor = LowLevelTexture.Descriptor(pixelFormat: .rg16Float, width: textureResolution, height: textureResolution, textureUsage: [.shaderWrite, .shaderRead])
let lowLevelTexture = try LowLevelTexture(descriptor: descriptor)

var textureResource = try TextureResource(from: lowLevelTexture)
var material = UnlitMaterial()
material.color = .init(tint: .white, texture: .init(textureResource))
```

### 30:27 - Update a LowLevelTexture on the GPU
```swift
let lowLevelTexture: LowLevelTexture
let commandBuffer: MTLCommandBuffer
let encoder: MTLComputeCommandEncoder
let computePipeline: MTLComputePipelineState

commandBuffer.enqueue()
encoder.setComputePipelineState(computePipeline)

let writeTexture: MTLTexture = lowLevelTexture.replace(using: commandBuffer)
encoder.setTexture(writeTexture, index: 0)
// ...

encoder.dispatchThreadgroups(/* ... */)
encoder.endEncoding()

commandBuffer.commit()
```
# Resources
* https://developer.apple.com/documentation/RealityKit/creating-a-spatial-drawing-app-with-realitykit
* https://developer.apple.com/tutorials/SwiftUI/drawing-paths-and-shapes
