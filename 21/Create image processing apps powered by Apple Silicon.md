#metal 
How to create image processing apps?
# Optimizing for Apple GPUs
## architecture
Many apps designed with discrete GPUs.  But apple gpus have unified memory architecture.
All blocks, CPU, GPU, neural, media, etc., have access to the same system memory.

TBDR – two main phases.  Tiling (whole render surface is split into tiles).  Rendering (pixels processed).

## Leveraging apple silicon
account for
* Unified memory
* TBDR

[[Harness apple GPUs with metal]]
[[Optimize metal performance for apple silicon macs]]

## Image processing – lessons learned

### Blits considerations
Most processing, are designed on discrete GPUs.  
* RAM and VRAM are separate
* Explicit copy or blit is required for residency
* Needed twice – CPU to VRAM, and back

#### ex
1.  CPU decodes
2.  Blit to VRAM
3.  GPU process
4.  blit to RAM
5.  CPU encode

On apple GPUs, blitting for residency is not needed.  CPU/GPU can access directly.  Check for unified memory
`device.hasUnifiedMemory`.  Memory, time, etc.

By removing blits, we completely avoid copy gaps and can start processing immediately.

#### Implement UMA no-blit path in your app
Large runtime cost for blist
* Consumes memory bandwidth
* Must share shader cores with other shaders and kernels
* May result in commands scheduling overhead

NO need for separate VRAM allocation.  Inspect your app blits and only do copies required.

### Render instead of compute when possible
Every serial compute dispatch is GPU global sync point.
Metal guarantees all dispatches see all writes.  Subsequent dispatches can see writes from all previous dispatches.

Tile dispatches only have tile-scope sync points.  Unlike regular compute, the operate in tile memory.  Convolutions cannot be mapped to tile, but many can.  
Deferring sstem memory until encoder endpoint provides huge efficiency gains.  Not limited by system bandwidth.

Many per-pixel operations don't require tile sync-points.  Fragmenta nd tile kernels can be easily intermixed.  

Switch to render instead of compute where possible.  Use `MTLRenderCommandEncoder` for esequential piepline parts

* Per-pixel operations - use fragment functions instead
* Threadgroup scoped operations – use tile kernels instead
* Scatter-gather/convolution kernels - keep compute dispatches

`MTLRenderCommandEncoder` enables unique apple GPU feature: Lossless bandwidth compression.

Lossless can't be applied, if
* texture is already block-compressed
* Texture options `MTLTextureUsagePixelFormatView | MTLTextureUsageShaderWrite | MTLTextureUsageUnknown`
* Texture is buffer-backed via `MTLBuffer.newTextureWithDescriptor()`.
* `MTLStorageModeShared` or `MTLStorageModeManaged` require explicit `optimizeContentsForGPUAccess()`, if the texture is initialized using `MTLTexture.replaceRegion()`.

Metal debugger now identifies this as an "insight"
### leveraging tile memory
Tile memory concepts are new to desktop Gpu
* load/store actions
* memoryless attachments

Render-target is split into tiles
Load/store actions are executed for each tile
* at the beginning of the render pass
* At the end of the pass

Don't load if not necessary!  If we're overwriting the whole image, set load action to dontCare

Don't clear with explicit blit/compute.  Use clear instead.

Don't store if not necessary!
Use dontcare instead.

Explicitly define resource as "fully transient" (not loaded or stored).  Memoryless.  Tile-only memory.  Resource only exists in tilememory for the lifetime of the encoder.

This greatly reduces memory footprint.  Every frame takes hundreds of mb.

How can be done?

```swift
let textureDescriptor = MTLTextureDescriptor.texture2DDescriptor(…)
let outputTexture = device.makeTexture(descriptor: textureDescriptor)

textureDescriptor.storageMode = .memoryless
let tempTexture = device.makeTexture(descriptor: textureDescriptor) 

let renderPassDesc = MTLRenderPassDescriptor()
renderPassDesc.colorAttachments[0].texture      = outputTexture
renderPassDesc.colorAttachments[0].loadAction   = .dontCare
renderPassDesc.colorAttachments[0].storeAction  = .store
renderPassDesc.colorAttachments[1].texture      = tempTexture
renderPassDesc.colorAttachments[1].loadAction   = .clear
renderPassDesc.colorAttachments[1].storeAction  = .dontCare

let renderPass = commandBuffer.makeRenderCommandEncoder(descriptor: renderPassDesc)
```


### Uber-shaders and function constants
Often to simplify kernel, you pass in control structure.
For example
* If tone map is enabled
* If data is in one format or another

Uber-shader approach has benefits of reusing same pipeline state objects.

#### impact on registers
Increased register usage to keep up with control flow
Reduces maximum occupancy

```cpp
fragment float4 processPixel(const constant ParamsStr* cs [[ buffer(0) ]])
{
  if (cs->inputIsHDR) {
    // do HDR stuff
  } else {
    // do non-HDR stuff
  }
  if (cs->tonemapEnabled) {
    // tone map
  }
}
```
since we cannot deduce at compiletime, we have to assume we can take both paths.  Combine based on flag.

Tone mapping: mask in or out based on input.  Problem here is registers.  Every control flow path needs live registers.

Registers used by the kernel defines max occupancy.  Because registers are shared by all SIMD lanes.  If we could only run what's needed, would enable higher SIMD group concurrency.

```cpp
constant bool featureAEnabled[[function_constant(0)]];
constant bool featureBEnabled[[function_constant(1)]];

fragment float4 processPixel(...)
{
  if (featureAEnabled) {
    // do A stuff
  } else {
    // do not-A stuff
  }
  if (featureBEnabled) {
    // do B stuff
  }
}
```




### Leveraging half and short
Apple GPUS have native 16-bit support
Fewer registers used equals better occupancy
Faster arithmetic equals better ALU utilization

Use half and short when possible
Conversions are typically free, even between float and half

### Using short arithmetic

e.g. thread position, thread group, etc.  Using 16-bit uses less registers, increasing occupancy.

GPU frame debugger in xcode 13 now has advanced pipeline info.  Can see register usage in there.

### `MTLPixelFormat` best practices
Different pixel formats might have different (slower) sampling rates.
Wide FP32 might have reduced point sampling rate
FP32 has reduced filtering rate for 2-4 components sampling
Smaller types reduce memory bandwidth and cache footprint
Use smallest type possible, especially when filtering is involved.

#### 3D LUT case study
Many apps use `rgba32Float` for 3D LUT step
* Consider switching to `rgba16Float` to get peak sampling rate
* If FP16 is not enough, use `rgba16Unorm`
	* Encode fixed-function unit values
	* Provide LUT range from the host
 

# Redesigning image processing pipelines
Image processing phase is compute and memory bandwidth intensive
* UMA
* Local image blocks backed by tile memory

## Video editing worfklow recap
[[Metal for pro apps]]

1.  Import
2.  Decode on CPU
3.  Image processing (GPU)
4.  Display
5.  Or encode

## image processing pipeline
* Unpack the image channels
* Colorspace conversion
* Apply 3D LUT
* Color correction
* Spatial-temporal noise reduction, blurs, convolutions, effects
* Pack the channels for output

Most of these operate on a single pixel.  

Spatial requires access to a large number of pixels.  Suited for compute kernels.

Every filter is its own kernel processing previous state and output for next stage.  ARrow means a buffer being returned from one stage and input to next stage.

Applications linearlize graph to keep the total number of intermediate resources low while avoiding race conditions.

But this is bandwidth-intensive.  Every operation has to go to/from dram.  

For 4k frame fp32 image processing,
* 67MB - fp16, 135mb - fp32
* 2.16GB of device memory traffic for chain of point filters
* Cache thrashing

Regular compute doesn't implicitly use on-chip tile memory

* Threadgroup memory is backed by tile memory.
* Not persistent across dispatches within a compute encoder.

In contrast, tile memory is persisted across draw passes within one *render command encoder*.

1.  Compute pass to render pass
2.  Merge eligible sequential filters into a single render pass
3.  Handle convolution filters and filters with scatter-gather access patterns.

1.  Beginning and end of the pipeline can be converted.
2.  Fragment shader can only update the image data.
3.  Next shader in the same pass can pick up output from tile memory.

How output generated?

```cpp
typedef struct
{
    float4 OPTexture        [[ color(0) ]];
    float4 IntermediateTex  [[ color(1) ]];
} FragmentIO;

fragment FragmentIO Unpack(RasterizerData in [[ stage_in ]],
                           texture2d<float, access::sample> srcImageTexture [[texture(0)]])
{
    FragmentIO out;
    
    //...
                         
    // Run necessary per-pixel operations
    out.OPTexture       = // assign computed value;
    out.IntermediateTex = // assign computed value;
    return out;
}

fragment FragmentIO CSC(RasterizerData in [[ stage_in ]], FragmentIO Input)
{
    FragmentIO out;

    //...    
    
    out.IntermediateTex = // assign computed value;
    return out;
}
```

Set the appropriate load/store properties as discussed.

Output is consumed as input.  I guess we create a whole bunch of fragment shaders here.

Now you have various things happening on tile memory with no memory passes.  At the end, non-memoryless target are flushed.

Filters with scatter/gather.  Kernels can operate on data in device memory.  Convolution is well-suited for tile-based compute kernels.  Here, we can declare threadgroup memory.  Bring in block of pixels into tile memory along with pixels etc.

Remember, tile memory is not persistent across compute dispatches within a compute encoder.  So after filter 1, have to flush to dram.

## results
For 4k fp32 image processing
* Bandwidth down from 2.16GB to 810mb (62%)
* NO need for 2 intermediate device buffers (270mb)
* Reduced cache thrasing

Key feature of apple silicon is unified memory architecture.  How to leverage?

# Utilize UMA for video encoding phase

Final output frame rendered by GPU can be consumed directly by media engines with no external copies.


Idea here is GPU writes directly into the final memory buffer.

1.  `CVPixelPool` backed by `IOSurface`
	1.  kCVPixelFormatType_420
2.  Pass to CVMtalTextureCache.  Both planes of biplanar buffer
3.  Get underlying metal texture from `CVMetalTextureRef` for each plane.   Recall these are backed by `IOSurface`.
4.  Render into Luma and Chroma plane textures. Will udpate the IOSurface.  We highly recommend doing chroma subsampling on GPU
5.  Send `CVPixelBuffer` to encoder
6.  Release CVPixelBuffer and CVMetalTexture.  

# Summary
* Leverage UMA
* Use `MTLRenderCommandEncoder` instead of compute
* Merge passes within a single encoder
* Set proper load/store actions
* Use memoryless for transient resources
* Leverage tile shading
* Use buffer pools with other APIs for 0-copy

