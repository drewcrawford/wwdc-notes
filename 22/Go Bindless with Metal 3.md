#metal 

# Bindless rendering
raytracing shaders can access all info.  App makes 3d textures etc. avialable to raytracing shaders.  All data into argument buffers.  When bindless rendering is paired with .e.g heaps, apps can enjoy better performance, less pressure on the CPU.

# enhancements in Metal3
* writing argument buffers
* unbounded arrays
* MTLHeap allocated accleeration structures
* Shader validation

## Writing argument buffers
encode scene into buffers
traditionally accomplished with argument encoder:
1.  describe struct members to emtal
2. write data into the buffer
3. [[Explore bindless rendering in Metal]]

Enocder objects can be tough to manage.  Multithreading.
C struct.

Write directly into argument buffers.  You now have access to a virtual GPU address and resource ID of the resources.  Metal now understands what resources.  No longer needed encoders.

All devices with argument buffers tier 2.
* 2016 mac or newer
* a13bionic or newer
* feature query in `MTLDevice`.

Define CPU struct.  Use a 64-bit type for buffer addresses.
Allocate from MTLDevice
Sub-allocate from MTLHeap
Get buffer contexts and cast to struct type.
Write addresses and IDs to the struct.

```objc
// Write argument buffers in Metal 3

struct Mesh
{
   uint64_t normals; // 64-bit uint for constant packed_float3*
};

NSUInteger meshArgumentSize = sizeof(struct Mesh);

id<MTLBuffer> meshArgumentBuffer = [device newBufferWithLength:meshArgumentSize
                                                       options:storageMode];

struct Mesh* meshes = (struct Mesh *)(meshArgumentBuffer.contents); 
 
meshes->normals = normalBuffer.gpuAddress + normalBufferOffset;
```

Metal guarantees that sizes and alignment match across clang and metal compilers.

Note that struct declaration changes between metal and C.
You can use a single declaration in header and ocnditional compilation to choose which one.

```cpp
// Shared struct:

#if __METAL_VERSION__
#define CONSTANT_PTR(x) constant x*
#else
#define CONSTANT_PTR(x) uint64_t
#endif

struct Mesh
{
    CONSTANT_PTR(packed_float3) normals;
};
```

Can use templates in C++.  Check out argument buffer sample code for best practices.

## unbounded arrays.
Must allocate enough storage for all structs you want to store
Iterate over buffer contents.

```objc
// Write unbounded arrays of resources in Metal 3

struct Mesh
{
   uint64_t normals; // 64-bit uint for constant packed_float3*
};

NSUInteger meshArgumentSize = sizeof(struct Mesh) * meshes.count;

id<MTLBuffer> meshArgumentBuffer = [device newBufferWithLength:meshArgumentSize
                                                        options:storageMode];

struct Mesh* meshes = (struct Mesh *)(meshArgumentBuffer.contents); 

for ( NSUInteger i = 0; i < meshes.count; ++i )
{
   meshes[i].normals = normalBuffers[i].gpuAddress + normalBufferOffsets[i];
}
```

free to index into any position.  bindless is flexible because your shaders have no constraints.

GPU side
```cpp
// Metal shading language:

struct Mesh
{
   constant packed_float3* normals;
};


fragment half4 fragmentShader(ColorInOut v          [[stage_in]],
                              constant Mesh* meshes [[buffer(0)]] )
{
    /* determine mesh to read, e.g. geometry_id */

    packed_float3 n0 = meshes[ geometry_id ].normals[0];
    packed_float3 n1 = meshes[ geometry_id ].normals[1];
    packed_float3 n2 = meshes[ geometry_id ].normals[2];

    /* interpolate normals and calculate shading */
}
```

another option, pull everything into a struct
```cpp
// Metal shading language:
struct Mesh
{
   constant packed_float3* normals;
};

struct Scene
{
   constant Mesh*     meshes;     // mesh array
   constant Material* materials;  // material array
};

fragment half4 fragmentShader(ColorInOut v          [[stage_in]],
                              constant Scene& scene [[buffer(0)]] )
{
    /* determine mesh to read, e.g. geometry_id */
    packed_float3 n0 = scene.meshes[ geometry_id ].normals[0];
    packed_float3 n1 = scene.meshes[ geometry_id ].normals[1];
    packed_float3 n2 = scene.meshes[ geometry_id ].normals[2];
    /* interpolate normals and calculate shading */
}
```

## MTLHeap allocated acceleration structures
Flag residency in a single call.
Reduce CPU usage

Per-device alignment and size.  New query to check this.
`heapAccelerationStructureSizeAndAlignWithDescriptor:`
different than `accelerationStructure...`
For compute: `useHeap`
for render: `useHeap:stages:`

Manual hazard tracking.  Synchronize acceleration structure builds between each other.  

[[Maximize your Metal ray tracing performance]]
## shader validation
residency of indirect resources
`useResource:` and `useHeap:` flag residency
Common source of GPU restarts
Command buffer failures
image corruption
These problems are *common in  bindless workflows*.

This year we have new functionality to detect missing residency.

### ex
app adds buffers, textures, etc.  At rendering time, before the app starts, it indicates to metal that it uses resources in the set by calling `useResource:`.  metal makes these resident.

```objc
// Argument buffer loading
for (NSUInteger i = 0; i < mesh.submeshes.count; ++i) {

    Submesh*      submesh     = mesh.submeshes[i];
    id<MTLBuffer> indexBuffer = submesh.indexBuffer;
    NSArray*      textures    = submesh.textures;

    // Copy index buffer into argument buffer
   submeshAB[i].indices = indexBuffer.gpuAddress;

    // Copy material textures into argument buffer
   for (NSUInteger m = 0; m < textures.count; ++m) {
        submeshAB[i].textures[m] = textures[m].gpuResourceID;
    }

    // Remember indirect resources
    [sceneResources addObject:indexBuffer];
    [sceneResources addObjectsFromArray:textures];
}
```

There is a subtle bug here.  The app would run thecommand buffer and in some cases reflectiosn would be missing.

Now in metal 3 shader validation later finds this.  Now produce an error during buffer execution.  Error message indicates function name, pass name, metal file and line of code, etc, and even the label of the buffer, its size, and the fact that it was not resident.

Make sure you label metal objects!

# performance
how to maximize game's performance when going bindless
* Unretained resources
* untracked resources

## unretained resources
achieved via reference counting
start with `retainCount=1`
deallocation happens when 0

Because cpu and gpu operate in parallel, it's a problem if the cpu deallocated a resource while the gpu is still using it.  So metal creates strong references to all resources in use.

* directly bound resources
* MTLHeaps (`useHeap`)
* Indirect resources (`useResource:`)
* [[Program Metal in C++ with metal-cpp]]

No need to keep strong CPU references
small CPU cost

In some cases it's unnecessary because you are keeping the thing alive for the whole app or whatever.  You can ask command buffers not to retain resources.  Granularity is the entire command buffer.

CPU usage reduction of 2%.  And we tried hard to find a large number.

## untracked resources
Multi-pass rendering.  Shadow mapping, etc.  
Read-after-write hazards.
Write-after-write hazards.
Tracked resources ensure synchronization
Consideration for aggregated resources

* opt out of hazard tracking
* manually express dependencies to metal
* `MTLHazardTrakcingModeUntracked`
* Default for MTLHeaps
### synchronization primitives
* fences
	* different render and compute passes, in a single command queue
	* commit or enqueue producing before the consuming buffer.
* events
	* `waitForEvent`: consumer can wait, and therefore could be encoded first.
* shared events
	* similar but work at larger scope beyond a single GPU.  Use this when you have different devices or with the CPU.  
* memory barriers
	* forces subsequent commands within a single render or compute pass to wait.
	* similar to the cost of the fence.  One exception: barriers after the fragment stage in a render pass.
		* very high cost, similar to splitting the render pass.  
		* we disable these on apple gpus
		* validation error
		* Recommended to use a fence for this

| primitive      | scope                                      | performance cost |
| -------------- | ------------------------------------------ | ---------------- |
| fence          | within a command q                         | low              |
| event          | across command queues, single metal device | medium           |
| shared event   | multiple command queues, multiple devices  | high             |
| memory barrier | within a pass                              | it varies        |

prefer fences.  Great for common cases.
when submisison order can't be guaranteed or you have multiple queues, use metal events
shared events for multiple devices
memory barriers for cases where it's desired to synchronize within a pass.  Usually fast such as concurrent compute passes or concurrent stages within draw calls.  But never after fragment stage.

using untracked resources you can have all advantages of data aggregation while maximizing gpu paralllelism.

# tools for bindless

New dependency viewer
analyze your workload with two kinds of dependencies: dataflow and synchronization
solid lines => dataflow
dotted lines => synchronization
more details in sidebar inspector
use menu at bottom to filter into one or the other

resource list.  This year the metal debugger lets you check which resources a draw call accesses.  Click on the "Accessed" filter at the top.
type of each access.
Useful for understanding what resuorces your shader has accessed.

If you see reosurces you're not expecting, use the shader debugger.
help identify issues where your shader accesses the wrong buffer elements.

# Wrap up
* simplifed argument buffer encoding
* MTLHeap-allocated acceleration structures
* Metal validation layer
* CPU and GPU performance

https://developer.apple.com/documentation/metal/metal_sample_code_library/rendering_reflections_in_real_time_using_ray_tracing


