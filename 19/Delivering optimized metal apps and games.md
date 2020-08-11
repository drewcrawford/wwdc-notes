#metal 

# Metal Best Practices
## General Performance

### Choose the right resolution

Each effect may require a different resolution

* Consider image quality and performance tradeoff
* Composite the game UI at native resolution

Note that they use different resolutions for different techniques.

### Minimize non-opaque overdraw

Processing multiple fragments per pixel causes overdraw

* Render meshes in the right order
* Don't render invisible (fully transparent) meshes

`FS Invocations / Pixels Stored`

### Submit GPU work early

Scheduling all offscreen GPU work early
* Improves latency and responsiveness
* Allows system to adapt to the workload
* In particular, do this **before getting the drawables**!  


1.  make cb for offscreen work
2.  `.commit()`
3.  wait for drawable
4.  make cb for onscreen work
5.  present drawable
6.  `.commit()`

#### Demo

### Stream resources efficiently

Allocating resources takes time
Streaming resources from the render thread may cause stalls

best practice
* Consider the memory and perf tradeoff
* Allocate and load GPU resources at launch time
* Allocate and stream new resources from a dedicated thread

### Design for sustained performance

Test your game under `serious` thermal state
Consider tuning for `serious` thermal state

[[Designing for Adverse Network and Temperature Conditions]]

Xcode energy gauge

## Memory Bandwidth

Memory transfers are costly

iOS devices have
* shared memory between CPU and GPU
* Dedicated memory for the GPU (tile memory)
* Metal is designed to help you leverage both

### Textures

#### Compress texture assets

Sampling large textures may be inefficient

* Compress all texture assets – ASTC, PVRTC, and more
* Generate mipmaps for textures that may be minified

If your game targets newer devices, use ASTC over PVRTC.



#### Optimize for faster GPU access

Some textures can't be compressed ahead of time
* Render targets
* Runtime generated

A12 supports lossless compression

* Use `private` storage mode
* Don't set `unknown` usage flag
* Don't set unnecessary usage flags (`shaderWrite` `pixelView`)
* For `shared` textures, explicitly optimize after any CPU updates

Calling `optimizeContentsForGPUAccess` on shared textures may help performance

Note that memory viewer can show storage mode and usage flags per texture

#### Choosing the right texture format

Avoid using pixel formats with unnecessary channels
Lower precision wherever possible.

Sampling rate (A12 and later)

* Conventional 32-bit formats -> 1.0x
* 64-bit formats: YCbCr (GBRGR422 and BGRG422) RGB11B10Float RGB9E5Float -> 0.5x
* 128-bit formats (RGBA32Float) -> 0.25x

### Render targets
#### Optimize load/store actions

Loading or storing render targets costs bandwidth
Suboptimal configuration may create false dependencies
Don't load/store unless necessary


#### Optimize MSAA
iOS devices have very fast MSAA
* Resolve from tile memory
* Explicit color coverage control, custom resolves, and more

* Consider MSAA over native resolution
* Don't load/store the multisample texture
* Set the storage mode of the multisample texture to `.memoryless`

#### Explicitly leverage tile memory
* Programmable blending
* image blocks
* tile shaders

[[Modern rendering with Metal]]

### Demo


## Memory Footprint

iOS enforces an app memory limit

iOS 12 has an accounting change for memory that particularly applies to metal
xcode memory gauge shows the right memory number
Will also display the memory limit

### Metal memory viewer
Added into frame debugger

Graph and detail UI format

### Metal Resource Allocations Instrument

1.  Track for allocation size
2.  Track for allocation events
3.  Detail

### C-based API to query available memory at runtime

`os_proc_available_memory`

### Use memoryless render targets

Transient render targets
* Not loaded/stored on system memory
* Don't require allocation

Use `memoryless` storage mode
Set for all multisampled attachments!

Check "Allocated size" in dependency viewer

### Avoid loading unused assets

Loading all the assets into memory will increase memory footprint

* Consider the memory and perf tradeoff
* Load only the assets taht will be used
* Free temporary resources such as splash screen or tutorial UI


### Use smaller assets

* Consider image quality/memory tradeoff
* Compress textures
* Compress meshes (vertex data)
* Consider smaller mipmap levels
* Consider lower-fidelity 3D models

### Simplify memory-intensive effects

Some FX require large offscreen buffers
Shadowmaps, SSAO

Consider lowering resolution, or disabling effects when memory-constrained

## Advanced memory footprint

### Metal resource heaps

Frames may involve intermediate memory
Resource heaps allow
* Creation of multiple resources with 1 allocation
* Resource aliasing

Consider aliasing intermediate resources that have no dependencies

### Purgeable memory

* Non-volatile
* Volatile
* empty

**Volatile and empty allocations don't count toward the appliation memory footprint**

* Explicitly manage the purgeable state of resources
* Mark all cached resources as volatile

`.setPurgeableState(.volatile)`

Potentially get a handle when the command buffer is completed, and flag as volatile again.

### Pipeline State Objects (PSO)

Load memory state upfront

Consider memory tradeoff of doing that though.  Don't hold PSO references that are no longer needed
Don't hold metal function references after PSO creation.  They're not needed to render, only to create new PSOs.

### Memory viewer demo

