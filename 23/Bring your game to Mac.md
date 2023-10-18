# Part 1: make a game plan
#games  #metal  #applesilicon 
Bring modern, high-end games to Mac and iPad with the powerful features of Metal and Apple silicon. Discover the game porting toolkit and learn how it can help you evaluate your existing Windows game for graphics feature compatibility and performance. We'll share best practices and technical resources for handling audio, input, and advanced display features. Once you've watched this session, check out “Bring your game to Mac, Part 2: Compile your shaders” to learn more about bringing HLSL shaders to Metal.

Game porting toolkit provides an emulation environment to modify your existing, unmodified games.  Graphics features, performance potential, etc.

It doesn't take months to understand how your game looks, sounds, and plays.  

sourc eporting
HLSL shader porting
graphics
audio, input, HDR
first launch.
debugging/optimizing

with the help of GPT, you can see you running on the platform much earlier.

metal shader converter.
HLSL GPU shaders.



## Make a game plan
DirectInput, XAudio2, Other APIs, Direct3D

baseline performance includes all the overhead of the GPT as well as rosetta, etc.

Play with metal performance HUD, and look specifically at that.  

GPU/resolution
instantanous framerate
approximation of the amount of GPU time being used.  High/low value of the statistics.  Shows memory usage, and provides a rolling graph.
Instruction and direct3D translation type
render encoders,
dipsatches, etc.
types of standard and emulated draw calls
resource usage.

developer.apple.com/metal

## compile your shaders

see part 2.
## Render with metal
MetalFX
fast resource loading
offlien compilation
mesh shaders
ray tracing

see part 3.
## input
time to port natively, and take advantage of advanced featuers like rumble/haptics.

windows game is already built over a cross-platform game engine.  Input plugins, input frameworks, input hardware, etc.

on apple platform, game controller framework provides good APIs, etc.  

complete support for
* haptics
* motion
* remapping
* captures

GameController framework resources

[[Tap into virtual and physical game controllers]]
[[Advancements in Game Controllers]]

## Audio
apple devices have state of the art built in speakers, etc.  No loud fans drowning them out.  Many excellent wired and wireless speakers.

you probably use cross-platform audio midldleware like wwise, unity, fmod.  Generally abstract system audio APIs and frameworks.

On windows, you may be using directsound, xaudio2, wasapi.  These exist on apple platforms.

audio frameworks include spatial miser audio unit, PHASE, AVAudioEngine.  All these middleware products are well-supported with native SDKs.

Can also use platform-native frmaeworks already.  Check out PHASE documentation, and sample code.

[[Discover geometry-aware audio with the Physical Audio Spatialization Engine (PHASE)]]
[[Plug-in and play Add Apple frameworks to your Unity game projects]]


## display/HDR?

With GPT, you can see your games' rendering code running over the basic APIs, etc.

you might be using IDXGISwapChain, AdvancedColorInfo, etc.

we use CAmetalLayer, and CAMetalDisplayLink.  Easily move your HDR and tonemapping logic whether it's based on floating point or etc.

Fine-grained control to achieve lowest input latency for your game.

[[Explore EDR on iOS]]
[[Explore HDR rendering with EDR]]

[[Bring your game to Mac]]

## part 2: Compile your shaders
Discover how the Metal shader converter streamlines the process of bringing your HLSL shaders to Metal as we continue our three-part series on bringing your game to Mac. Find out how to build a fast, end-to-end shader pipeline from DXIL that supports all shader stages and allows you to leverage the advanced features of Apple GPUs. We'll also show you how to reduce app launch time and stutters by generating GPU binaries with the offline compiler. To get the most out of this session, we recommend first watching “Bring your game to Mac, Part 1: Make a game plan." And once you're ready to level up, check out “Bring your game to Mac, Part 2: Render with Metal" from WWDC23.

### 14:28 - Json Metal Script
```json
{
    "libraries": {
        "paths": [
            {
                "path": "ba.metallib",
                "label": "myMetalLib"
            }
        ]
    },
    "pipelines": {
        "render_pipelines": [
            {
                "vertex_function": "alias:myMetalLib#v",
                "fragment_function": "alias:myMetalLib#f",
                "raster_sample_count": 2,
                "color_attachments": [
                    {
                        "pixel_format": "BGRA8Unorm"
                    }
                ],
                "depth_attachment_pixel_format": "Depth32Float"
            }
        ]
    }
}
```

### 16:30 - Testing Binary Archive hit
```objc
// Create Pipeline Descriptor
MTLComputePipelineDescriptor *computeDesc = [MTLComputePipelineDescriptor new];
computeDesc.binaryArchives = @[existingBinaryArchive];
computeDesc.computeFunction = computeFn;
id<MTLComputePipelineState> computePS = [device newComputePipelineStateWithDescriptor:computeDesc
                                     options:MTLPipelineOptionFailonBinaryArchiveMiss
                                     error:&err];                                                                                        

if(computePS == nil)
{
    // Binary archive is missing compiled shader.
}
```

### 17:03 - Loading appropriate Binary Archive
```objc
// Load OS-specific binary archives
MTLComputePipelineDescriptor *computeDesc = [MTLComputePipelineDescriptor new];

if (@available(macOS 14, *)) {
    computeDesc.binaryArchives = @[binaryArchive_macOS14];
} else {
    computeDesc.binaryArchives = @[binaryArchive_macOS13_3];
}  
computeDesc.computeFunction = computeFn;
id<MTLComputePipelineState> computePS = [device newComputePipelineStateWithDescriptor:computeDesc
                                     options:nil
                                     error:&err];
```
## Resources
* https://developer.apple.com/download/all/?q=game%20porting%20toolkit
* https://developer.apple.com/download/all/?q=shader%20converter
* https://developer.apple.com/metal/shader-converter/
* https://developer.apple.com/documentation/metal

# Part 3: Render with Metal

Discover how you can support Metal in your rendering code as we close out our three-part series on bringing your game to Mac. Once you've evaluated your existing Windows binary with the game porting toolkit and brought your HLSL shaders over to Metal, learn how you can optimally implement the features that high-end, modern games require. We'll show you how to manage GPU resource bindings, residency, and synchronization. Find out how to optimize GPU commands submission, render rich visuals with MetalFX Upscaling, and more. To get the most out of this session, we recommend first watching “Bring your game to Mac, Part 1: Make a game plan” and “Bring your game to Mac, Part 2: Compile your shaders" from WWDC23.

### Encode the texture tables. - 3:55
```objc
// Encode the texture tables outside of the rendering loop.

id<MTLBuffer> textureTable  = [device newBufferWithLength:sizeof(MTLResourceID) * texturesCount
                                                  options:MTLResourceStorageModeShared];


MTLResourceID* textureTableCPUPtr = (MTLResourceID*)textureTable.contents;
for (uint32_t i = 0; i < texturesCount; ++i)
{
    // create the textures.
    id<MTLTexture> texture = [device newTextureWithDescriptor:textureDesc[i]];

    // encode texture in argument buffer
    textureTableCPUPtr[i] = texture.gpuResourceID;
}
```

### Encode the sampler tables. - 4:33
```objc
// Encode the sampler tables outside of the rendering loop.

id<MTLBuffer> samplerTable  = [device newBufferWithLength:sizeof(MTLResourceID) * samplersCount
                                                  options:MTLResourceStorageModeShared];

MTLResourceID* samplerTableCPUPtr = (MTLResourceID*)samplerTable.contents;
for (uint32_t i = 0; i < samplersCount; ++i)
{
    // create sampler descriptor
    MTLSamplerDescriptor* desc  = [MTLSamplerDescriptor new];
    desc.supportArgumentBuffers = YES;
    ...

    // create a sampler
    id<MTLSamplerState> sampler = [device newSamplerStateWithDescriptor:desc];

    // encode the sampler in argument buffer
    samplerTableCPUPtr[i] = sampler.gpuResourceID;
}
```

### Encode the top level argument buffer. - 5:05
```objc
// Encode the top level argument buffer.

struct TopLevelAB
{
    MTLResourceID* textureTable;
    float*         myBuffer;
    uint32_t       myConstant;
    MTLResourceID* samplerTable;
};

id<MTLBuffer> topAB = [device newBufferWithLength:sizeof(TopLevelAB)
                                          options:MTLResourceStorageModeShared];


TopLevelAB* topABCPUPtr     = (TopLevelAB*)topAB.contents;
topABCPUPtr->textureTable   = (MTLResourceID*)textureTable.gpuAddress;
topABCPUPtr->myBuffer       = (float*)myBuffer.gpuAddress;
topABCPUPtr->myConstant     = 128;
topABCPUPtr->samplerTable   = (MTLResourceID*)samplerTable.gpuAddress;
```

### Allocate the read-only resources. - 6:49
```objc
// Allocate the read-only resources from a heap.

MTLHeapDescriptor* heapDesc = [MTLHeapDescriptor new];
heapDesc.size               = requiredSize;
heapDesc.type               = MTLHeapTypeAutomatic;

id<MTLHeap> heap = [device newHeapWithDescriptor:heapDesc];


// Allocate the textures and the buffers from the heap.

id<MTLTexture> texture = [heap newTextureWithDescriptor:desc];
id<MTLBuffer>  buffer = [heap newBufferWithLength:length options:options];
...

// Make the heap resident once for each encoder that uses it.

[encoder useHeap:heap];
```

### Allocate the writable resources. - 7:34
```objc
// Allocate the writable resources individually.

id<MTLTexture> textureRW = [device newTextureWithDescriptor:desc];
id<MTLBuffer>  bufferRW  = [device newBufferWithLength:length options:options];


// Mark these resources resident when they're needed in the current encoder.
// Specify the resource usage in the encoder using MTLResourceUsage.

[encoder useResource:textureRW usage:MTLResourceUsageWrite stages:stage];
[encoder useResource:bufferRW  usage:MTLResourceUsageRead  stages:stage];
```

### Encode the execute indirect - 19:31
```objc
// Encode the execute indirect command as a series of indirect draw calls.

for (uint32_t i = 0; i < maxDrawCount; ++i)
{
    // Encode the current indirect draw call.
    [renderEncoder drawIndexedPrimitives:MTLPrimitiveTypeTriangle
                             indexType:MTLIndexTypeUInt16
                         indexBuffer:indexBuffer
                   indexBufferOffset:indexBufferOffset
                      indirectBuffer:drawArgumentsBuffer
                indirectBufferOffset:drawArgumentsBufferOffset];
    
    // Advance the draw arguments buffer offset to the next indirect arguments.
    drawArgumentsBufferOffset += sizeof(MTLDrawIndexedPrimitivesIndirectArguments);
}
```

### Translate the indirect draw arguments to ICB. - 21:48
```cpp
// Kernel written in Metal Shading Language to translate the indirect draw arguments to an ICB. 

kernel void translateToICB(device const Command* indirectCommands [[ buffer(0) ]],
                           device const ICBContainerAB* icb [[ buffer(1) ]],
                           ...)
{
    ...

    device const Command* indirectCommand = &indirectCommands[commandIndex];
    device const MTLDrawIndexedPrimitivesIndirectArguments* args =
    &command->mdiBuffer[mdiIndex];

    render_command drawCall(icb->buffer, indirectCommand->mdiCmdStart + mdiIndex);

    if(args->indexCount > 0 && args->instanceCount > 0) {
        encodeCommand(indirectCommand, args, drawCall);
    }
    else {
        cmd.reset();
    }
}

// Encode a render command on the GPU.
void encodeCommand(device const Command* indirectCommand,
                   device const MTLDrawIndexedPrimitivesIndirectArguments* args,
                   thread render_command& drawCall)
{
    drawCall.set_render_pipeline_state(indirectCommand->pso);

    for(ushort i = 0; i < indirectCommand->vertexBuffersCount; ++i) {
        drawCall.set_vertex_buffer(indirectCommand->vertexBuffer[i].buffer,
                              indirectCommand->vertexBuffer[i].slot);
    }

    for(ushort i = 0; i < indirectCommand->fragmentBuffersCount; ++i) {
        drawCall.set_fragment_buffer(indirectCommand->fragmentBuffer[i].buffer,
                                indirectCommand->fragmentBuffer[i].slot);
    }

    drawCall.draw_indexed_primitives(primitive_type::triangle,
                                args->indexCount,
                                indirectCommand->indexBuffer + args->indexStart,
                                args->instanceCount,
                                args->baseVertex,
                                args->baseInstance);
}
```

## Resources
* https://developer.apple.com/documentation/metalfx/applying_temporal_antialiasing_and_upscaling_using_metalfx
* https://developer.apple.com/documentation/metal
* https://developer.apple.com/documentation/metalfx
* https://developer.apple.com/documentation/metal/metal_sample_code_library/modern_rendering_with_metal
