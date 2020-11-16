#metal #applesilicon 

# Metal capabilities for Apple M1
* Designed for Metal
* Unifies iOS and macOS Metal Features
* Maximizes graphics and compute performance

Have been exposing 2 distinct families
* MetalMacGPUFamily2
* Metal AppleGPUfamily7

Apple M1 supports both.  This means all features are available.

## Metal feature detection
Will return true for both families.

## GPU-driven pieline
* Tier 2 argument buffers
* Indirect render command
* Indirect compute command

## Rendering
* Barycentric coordinates and primitive ID
* MSAA layered rendering
* Raster Order Groups
* Ray tracing

## Texture
* Sparse textures with access counters
* BC pixel formats
* F32 filtering
* Texture clamp modes
## Compute
* SIMD reduction and permute functions
* SIMD matrix multiply

## GPU performance
* Powerful TBDR architecture
* Designed for Metal
* Multi-app responsiveness

## Apple M1 shader core
* fast floating point math
* fast threadgroup memory
* increased ALU utilization
* Fast fnction calls and stack accesses

## GPU pipeline and texture performance
* Fast GPU-driven pipeline
* FP16 texture filtering
* FP32 and int32 texture image reads
* Lossless compressed textures

## Multi-app and UI responsiveness
* multi-channel compute
* Fast context switch
* Multi-app scheduling


# Make Metal applications shine on the new Apple silicon Macs

We've been considering how apps could be optimized further.  We've found a few features you may consider.

Tile shaders, memoryless render targets, programmable blending, sparse textures.

## Tile shading

Quick recap of TBDR (tile-based deferred rendering).

1.  Vertex processing (geometries)
2.  Fragment processing (pixels)

Tile shading is a fragment feature.
Depth/stencil is used for hidden-surface removal.
Tile memory is used for color values.

You can insert a compute pass in the middle of your fragment pass, which can access tile memory.  Avoiding a separate pass improves your bandwidth significantly.

Tile shading can mix render and compute in a single pass.  No repeated load/store between system and tile memory.  

### example
Tile-based light culling.  Split into tiles and create a light list per-tile.  Then the final pass only needs to think about lights in the list.

1.  G-buffer laydown
2.  Light culling (tile shading)
3.  Light accumulation (tile shading)

There is no break in the render pass, all computation is done in 1 pass.  Keeping all data in tile memory w/o going to system memory.  Since data stays in tile memory, use memoryless render targets.

* access to tile, threadgroup, and device memory
* Dispatches barrier against earlier and later draws
* Mixing compute and render to reduce memory bandwidth.

Note that, all of this seems like it might be specific to fragment passes, as opposed to vertex passes.

## Memoryless render targets
* Render targets not backed by system memory
* Created with storage mode `MTLStorageModeMemoryless`
* Use for render pass attachments that are not stored

### Use case: MSAA
Use `MTLStoreActionResolve` to save memory bandwidth.

### Use case: depth/stencil
Depth attachments are required for depth testing, but if you don't need it... you can make it memoryless

Use `MTLStoreActionDontCare` and `MTLStorageModeMemoryless`.

### Use case: Deferred rending with light culling
Attachments that are stored in tile memory don't need to be accessed in system memory.

## Programmable blending
Fragment shaders can access pixel data from tile memory
Allows more complex effects to be rendered in a single pass (merge multiple passes)
Save memory bandwidth.

### Classic deferred shading.

Three passes
1.  Vertex shader
2.  Draw G-buffer
3.  Shade with lighting

If you only need to read the output from the previous step at the current (that is, same) pixel position, consider using programmable blending.  It lets you fold 2 render passes into a single pass.

Now your targets can be marked as memoryless as well.

```cpp
struct GBufferData
{
    half4 lighting        [[color(RTLightingIndex), raster_order_group(LightingROG)]];
    half4 albedo_specular [[color(RTAlbedoIndex),   raster_order_group(GBufferROG)]];
    half4 normal_shadow   [[color(RTNormalIndex),   raster_order_group(GBufferROG)]];
    float depth           [[color(RTDepthIndex),    raster_order_group(GBufferROG)]];
};

fragment float4 deferred_lighting(GBufferData GBuffer) {…}
```

We encourage you to check if your fragment shaders are reading the pixel value from a previous fragment shader. 

In this code snippet, you add the fragment shader info to your inputs and then you can read back the data.  This allows more complex fx without memory bandwidth.

## Sparse Textures

### Quick recap of texture streaming
Render massive scenes within fixed memory
Only load textures needed by current view
Texture residency managed at mip granularity (mipmap)
Keep lowest levels resident

### Sparse texture streaming

Provide texture access counters that tell you how often textures are needed

* access counters help prioritize loading
* Load sparse tiles instead of mipmaps.  this means we can load invididual pieces (tiles) of a mipmap
* Higher quality within same footprint
* Same quality with smaller footprint

### Using sparse textures
1.  Initialization
2.  Render
3.  Access counters
4.  Stream

#### Initialization
Bind sparse heap.  Defines memory budget.  Sampling textures will automatically read the contents.  You load the lowest mip levels.

```swift
// Initializing sparse heap/textures

// determine size of sparse heap
let sparseHeapSizeInBytes = 64 * 1024 * 1024 * 1024
let sparseTileSize = device.sparseTileSizeInBytes
let alignedHeapSize = alignUp(sparseHeapSizeInBytes, sparseTileSize)

// create heap descriptor
let descriptor = MTLHeapDescriptor()
descriptor.type = .sparse
descriptor.storageMode = .private
descriptor.size = alignedHeapSize
descriptor.hazardTrackingMode = .tracked

// create heap
let sparseHeap = device.makeHeap(descriptor: descriptor)
```

```swift
// Initializing sparse heap/textures

let textureDescriptor = MTLTextureDescriptor.texture2DDescriptor(
                         pixelFormat: .bgra8Unorm_srgb,
                         width: 1024,
                         height: 1024,
                         mipmapped: true)

textureDescriptor.storageMode = sparseHeap.storageMode

let sparseTexture = sparseHeap.makeTexture(descriptor:textureDescriptor)
```

#### rendering
will sample the sparse textures.

```cpp
// Rendering

fragment float4 FragmentShader(constant Uniforms& uniforms     [[buffer(0)]],
                               texture2d<float>   colorMap     [[texture(0)]],
                               constant float&    firstTailMip [[buffer(1)]],
                               VertexOutput       in           [[stage_in]])
{

    constexpr sampler s(mip_filter::linear, mag_filter::linear, min_filter::linear);
    sparse_color<float4> sparseSample = colorMap.sparse_sample(s, in.texCoord.xy);

    if (sparseSample.resident()) {  //can determine if mapped in memory or not
        // return sample if mapped
        return sparseSample.value();
    }
    else { 
        // fall-back: read the always resident mip
        return colorMap.sample(s, in.texCoord.xy, min_lod_clamp(firstTailMip));
    }
}
```

#### query access counters
```swift
// Texture access counters

let counterBufferSize = MemoryLayout<UInt32>.stride *
                        tileRegion.size.height *
                        tileRegion.size.width *
                        tileRegion.size.depth
let counters = device.makeBuffer(length: counterBufferSize, options: .storageModeShared)

let blitEncoder = commandBuffer.makeBlitCommandEncoder()
blitEncoder!.getTextureAccessCounters(texture,
                                      region: tileRegion,
                                      mipLevel: 0,
                                      slice: 0,
                                      resetCounters: true,
                                      countersBuffer: counters!,
                                      countersBufferOffset: 0)
blitEncoder!.endEncoding()
```

Once you know which tiles are the most visible based on access counters, you can stream in texture contents.  To do this, you map sparse tiles and load into the tiles.

Can also unmap tiles that are not visible

```swift
// Map, unmap, upload

// create MTLResourceStateCommandEncoder
let encoder = commandBuffer.makeResourceStateCommandEncoder()

// Schedule a map/unmap operations
encoder.updateTextureMappings(texture,
                              mode: .map,
                              regions: tileRegionMapArray,
                              mipLevels: mipLevelMapArray,
                              slices: sliceMapSlices,
                              numRegions: numMap)
encoder.updateTextureMappings(texture,
                              mode: .unmap,
                              regions: tileRegionUnmapArray,
                              mipLevels: mipLevelUnmapArray,
                              slices: sliceUnmapArray,
                              numRegions: numUnmap)
encoder.endEncoding()
```
# Wrap up
* Unifies iOS and macOS Metal features
* Maximizes graphics and compute performance
* Enables advanced rendering techniques

https://developer.apple.com/documentation/metalperformanceshaders
https://developer.apple.com/documentation/metal
https://developer.apple.com/metal/metal-shading-language-specification.pdf
