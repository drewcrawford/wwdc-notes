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

see first session, etc.

# New metal compiler tools
* avoid on-device generation of metal IR.  
* Metal provides you the tools necessary to generate metal IR aot.
* stored as a part of a metal library.  Aim to generate metal libraries AOT.

however, when coming from other API, need a way to get them into metal.  if you're a newcomer to the mac, you have Metal Shader Converter.
simplify shader pipelines
package metal libraries

allow converted shaders to natively integrate
# Convert your shaders

supports translation of DXIL to metal IR
fast compilation through binary translation
reduced shader asset build times
enables the advanced features of apple GPUs

supports all traditional and modern shader stages

compute stage
ray generation stage, amplification, mesh shaders, etc.

how to use.

after you set up DXC and shader converter, start by compiling to dxil.  Use dxc for this.
call `metal-shaderconverter` that transpiles from dxil to metallib
by default, we emit latest version of macOS DT, and json for reflection.

some game engines ahve custom asset build programs that compile/package shaders into game-specific formats.

you may also want to see how shaders are working just-in-time, like for evaluation purposes.

dynamic library itnerface is also available.  Exposes all the same functionality as the CLI tool.

pure C interface.  available on macOS and windows.  

game integration.  
* declare resources (global variables, register declarations, etc.)
* bind resources - directly bind to each stage/slot?
* define an explicit memory layout?

metal is flexible.  lay out resources into argument buffers.  Bind top--level argument buffer
different layout modes to suit your game

top-level argument buffer is a shared resource.  Race condition of CPU writes while GPU reads.
do not serialize Cpu/gpu work
use a "bump' allocator.  Can be a alarge mtl buffer where we sub-allocate resources
shadow for each flame in flight
see sample code

[[Go Bindless with Metal 3]]

can also support more complex pipelines.

Geometry and tesselation.  When you rely on these for traditional effects, converting them by hand is costly.  Metal shader converter helps you bring these pipelines to metal by adapting them to mesh shaders.  The tooling does the heavy lifting to move things over to metal behaviors.

includes the tesselator.  To support this, we are adding `objectLinkedFunctions` and `meshLinkedFunctions`.  After you compile your shaders, use them to build a render pipeline descriptor and compile it into PSO.  compile/link emtal IR, making aall functions into a single pipeline, avoiding function call overhead, etc.


shader converter runtime helps you build pipeilnes, emulate draw calls.  For more information, see documentation.

Performance tips.

Avoid calling `useResource` more than necessary.  This is an expensive call.  Favor `useREsources` or `useHeap`.  Provide several resources at once.  

Take advantage of GPU binary caching.

* compatability flags
* optimize for gpu family
* customize vertex fetch
* rename entry point
* reflection, etc.

can mix shading language.  it's all metal IR.

Textuer array sexplicitly required
load resources as texture arrays
use texture views
MTKTextureLoaderOptionLoadAsArray
samplers require `supprotsARgumentBuffers`.  

get metal shader converter from the downloads section.  macOS, or developer tools for windows.  

cost of JIT compilation.

Reduce load times and runtime stutters, by generating shader binary at game buildtime.  

gpu binary toolchain.  this happens when you create `newPipelineStateWithDescriptor`.  Not only metal library, but other information such as color format of attachments, vertex layout descriptor, etc.  These are generated JIT during PSO creation.

Binary archives allow you to take control of this compliation.  Provide both existing metal library as well as pipeline configuration script.  Provide to `metal-tt`.  

# Finalize GPU binaries



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

you now have a metal script taht you can use.  see the json schema in docs.

can record binary archives with `serializeToURL`.  If you ahrvest these from device, can use `metal-source` to extract the piepelinscripts

be sure to update the paths to the metal libraries.

[[Build GPU binaries with metal]]
[[Discover compilation workflows in Metal]]

binary archives contain gpu binaries.
metal produces different versions based on their device.  metal-tt helps manage teh complexity by encapsulating variants for different gpu targets.  metal loads the papoprirate binary at runtime.

can encapsulate multiple sets of binaries into a single binary archives.

best practices
* metal searches the archive at runtime
* user does not benefit if no match is found
* test this with `FailOnBinaryArchiveMiss`.  

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

deployment and os compatibility
* not all players may be on latest os
* generate an archive per major oOS Update
* include all of them in your app

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

metal upgrades on-device binary archives
for any unpackaged binary archive in the app bundle
automatic after os update or game install

| tool                  | description                                     | macOS | windows |
| --------------------- | ----------------------------------------------- | ----- | ------- |
| metal                 | compile msl into metal ir                       | yes   | yes     |
| metal-shaderconverter | comiple other ir to metal ir                    | new   | new     |
| metal-tt              | comiple metal ir into gpu binaries              | yes   | new     |
| metal-source          | extract a pipeline script from a binary archive | yes   | new     |

## Wrap up
* metal shader converter
* gpu binary compiler
* complete set of tools


## Resources
* https://developer.apple.com/download/all/?q=game%20porting%20toolkit
* https://developer.apple.com/download/all/?q=shader%20converter
* https://developer.apple.com/metal/shader-converter/
* https://developer.apple.com/documentation/metal

# Part 3: Render with Metal

Discover how you can support Metal in your rendering code as we close out our three-part series on bringing your game to Mac. Once you've evaluated your existing Windows binary with the game porting toolkit and brought your HLSL shaders over to Metal, learn how you can optimally implement the features that high-end, modern games require. We'll show you how to manage GPU resource bindings, residency, and synchronization. Find out how to optimize GPU commands submission, render rich visuals with MetalFX Upscaling, and more. To get the most out of this session, we recommend first watching “Bring your game to Mac, Part 1: Make a game plan” and “Bring your game to Mac, Part 2: Compile your shaders" from WWDC23.

## Manage GPU resources
* resource bindings
* resource residency and synchronization

translate your shaders with Metal Shader Converter
choose your bindings model
* automatic layout - converter generates bindings information automatically
* explicit layout - pass to MSC.  

ex, root signature
descriptor -> points to textures
buffer ?
constants?
descriptor table -> points to samplers

each descriptor table is a resuorce array to things of homogeneous type.  In metal, we let things be heterogeneous, but you can encode this pattern with an argument buffer.

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

you can do this upfront, outside of your rendering loop.  

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

by storing the gpu address, it's fine.
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

encode arugment buffers using metal 3
impelment root signatures and descriptor tables

* resource bindings
* resource resdiency and synchronization
* bindless resources require explicit residency management on all gpu aarchitectures
* metal provides efficient ways to control residency
we recommend allocating read-only resources from a MTLHeap
* call useHeap once per encoder


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

allocate writable resources individually:
* call useResources with the right usage flags when necessary.
* metal will handle synchronization and optimize for performance

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

 recommendation table
 
| resources       | read-only                 | writable                  |
| --------------- | ------------------------- | ------------------------- |
| binding         | top-level argument buffer | top-level argument buffer |
| allocation      | heap                      | individual                |
| hazard tracking | untracked                 | default                   |
| residency       | useHeap                   | useResource                          |

[[Go Bindless with Metal 3]]


## Optimize rendering commands
tile-based deferred rendering (TBDR)
unified system memory architecture
fast on-chip tile memory

[[Bring your  metal app to apple silicon]]
[[Harness apple GPUs with metal]]

engine commands
* continuous commands

metal commands:
* recorded in command buffer
* grouped in passes
* written with command encoder
* command buffer submitted to a command queue

* move copies before rendering
* group commands of the same type
* avoid empty encoders to clear render targets
* optimize MTLLoadAction / MTLStoreAction

ex

if possible, batch copies at the start of the rendering pass, to avoid interrupting it

reorder draws to batch together.  that lets draw calls become merged intoa  single render pass, saving significant memory bandwidth.

avoid a pass for clearing.  in metal, there is a very efficient way, use load action clear for the first render pass.  

only have to store content that will be used in the next pass.  set store action to storeactiondontcare.  

move copies before rendering
group commands of same type
avoid empty encoders to clear
optimize load/store actions

visualize pass dependencies
full-featured debugging and profiling suite
demo

learn more about metal debugger in xcode
[[Gain insights into your Metal app with Xcode 12]]
[[Discover Metal debugging, profiling, and asset creation tools]]


## Handle indirect rendering
with indirect rendering, instead of encoding multiple draw commands, arguments are stored in memory.  Only once encoded, specifying how many draw calls, etc.

main idea is to populate indirect buffer via compute shader, scheduled for execution.  this wya, the gpu prepares work for itself and decides what to render.  execution of indirect arguments is a key feature for advanced techniques such as gpu-driven rendering.

draw indirect `drawPrimitives` and `drawIndexedPrimitives` with indirect ubffer

or use indirect buffer ICB.
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

draw indirect  
if you have scnees with thousands of draw calls, and you're limited by cpu encoding time, consider indirect command buffers.  superset of buffers with indirect draw arguments.

also set buffer bindings and PSOs from the gpu.  encode `executeComamndsInBuffer` command.

Usually, with executeindirect, all draw calls share the same PSO.  When PSO changes, have to encode a new `executeIndirect` command.

but if you're using ICBs, it's not required to split by state changes.  All PSOs and buffer bindings will be set from ICB.  don't have to encode.  Depending on the structure of the scene, this might significantly reduce the encoding time.

Not necessary to modify existing shaders, etc.  Can share tehsame shaders with other platforms and compile them with metal shader converter.  Translate draw arguments to ICB by adding small compute kernel, and before direct rendering pass.




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

[[Modern rendering with Metal]]

## Upscale with metalFX

save time to each frame by reudcing the amount of gpu work.  Turnkey solution.
* performs upscaling on reduced rendering reoslution
* enables high-performance upscaling
* supports two upscaling algorithms
	* spatial - best performance
	* temporal - quality approaching native rendering

this year:
* iOS
* up to 3x upscaling
* support in metal-cpp

requirements:
engine upscaling support
configurable mip LOD bias

temporal upscaling also requires
* jitters and motion vectors
* exposure (optional)
* reset history on camera cuts

see docs, [[Boost performance with MetalFX Upscaling]]


# Wrap up
Manage gpu resources
optimize rendering commands
handle indirect rendering
upscale with metalFX

[[Optimize metal performance for apple silicon macs]]













## Resources
* https://developer.apple.com/documentation/metalfx/applying_temporal_antialiasing_and_upscaling_using_metalfx
* https://developer.apple.com/documentation/metal
* https://developer.apple.com/documentation/metalfx
* https://developer.apple.com/documentation/metal/metal_sample_code_library/modern_rendering_with_metal
