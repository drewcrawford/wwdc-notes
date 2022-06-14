#metal 

* new flexible pipeline for GPU-drive geometry creation and processing
* applications
	* fine-grained geometry culling
	* procedural geometry
	* custom geometry inputs - compressed vertex trees, etc.


# Metal mesh shaders
traditionally, device memory, command encoder, draw call, etc.  Vertex shader.  Rasterizer.  Fragment shader.

This has certain limitations.  ex, procedural fur.Traditionally, you need a compute kernel, generate geometry, and output it back into device memory.  Then RCE to take the procedural geometry as an input.  This requires 2 command encoders, as well as memory buffers.  Barrier.  Etc.

New geometry pipeline that replaces vertex shader stage with 2: object shader, and mesh shader.

1.  object shader => produces payload
2. mesh shader => accepts payload.

procedural geometry only exists inside the draw call, so it requires no device memory.
Draw calls are performed

* similar to compute kernels
* launched in grids of thread groups
* object thread group can launch a mesh grid

each object grid TG can launch a mesh grid.
Each object grid TG passes payload data.

objects can be whatever you want.  region of space, etc.  Mesh is designed to build meshes and send geometry data to rasterizer.
# Hair rendering
Divide target surface into tiles.  Calculate LOD per each tile.  Generate hair strands.

* object grid
* object threadgroup
* payload
* mesh grid
* mesh threadgroup
* mesh output

`object_data` payload.
metal output `metal::mesh`.

For hair rendering, payload is a strand of hair as curve points.
mesh output is triangles I guess.  Then we send to rasterizer as normal.

First, we split into tiles.  Then do object shader.Each threadgroup can generate unique mesh threadgroup sizes, e.g. 4 for one, 3 for the next, etc.

```cpp
[[object]]  //object shader
void objectShader(object_data CurvePayload *payloadOutput [[payload]], 
                             const device void *inputData [[buffer(0)]], 
                             uint hairID [[thread_index_in_threadgroup]],
                             uint triangleID [[threadgroup_position_in_grid]],
                             mesh_grid_properties mgp) //encodes mesh grid size
{
    if (hairID < kHairsPerBlock)
       payloadOutput[hairID] = generateCurveData(inputData, hairID, triangleID);
    if (hairID == 0)
       mgp.set_threadgroups_per_grid(uint3(kHairPerBlockX, kHairPerBlockY, 1));
}
```

encoding
```swift
let meshPipelineDesc = MTLMeshRenderPipelineDescriptor()
meshPipelineDesc.objectFunction = objectFunc
meshPipelineDesc.payloadMemoryLength = kPayloadLength
meshPipelineDesc.maxTotalThreadsPerObjectThreadgroup = kHairsPerBlock
```
## limits
* 16kb payload size limit
* mesh grid size limit is 1024 threadgroups
* user-defined payload.  Payload is the set of curve control points.  Each mesh threadgroup is a single strand of hair.

## mesh shader
```cpp
struct VertexData    { float4 position [[position]]; };
struct PrimitiveData { float4 color; };

using triangle_mesh_t = metal::mesh<
                                    VertexData,               // Vertex type
                                    PrimitiveData,            // Primitive type
                                    10,                       // Maximum vertices
                                    6,                        // Maximum primitives
                                    metal::topology::triangle // Topology
>;

[[mesh]] 
void myMeshShader(triangle_mesh_t outputMesh, ...);
```

This is some kind of crazy thread strategy based on thread ID.  Basically, if your thread ID is low, you do some of the work.  e.g., one thread encodes the vertex count, because there is only 1 count of all vertexes.  etc.

```cpp
[[mesh]] void myMeshShader(triangle_mesh_t outputMesh,
                         uint tid [[thread_index_in_threadgroup]])
{

    if (tid < kVertexCount) 
        outputMesh.set_vertex(tid, calculateVertex(tid));

    if (tid < kIndexCount) 
        outputMesh.set_index(tid, calculateIndex(tid));

    if (tid < kPrimitiveCount)
        outputMesh.set_primitive(tid, calculatePrimitive(tid));

if (tid == 0)
    outputMesh.set_primitive_count(kPrimitiveCount);

}
```


```swift
meshPipelineDesc.meshFunction = meshFunc
meshPipelineDesc.maxTotalThreadsPerMeshThreadgroup = kVertexCountPerHair
```

## limits
* 256 vertices
* 512 primitives
* 16KB size for total mesh

## fragment stage
```swift
meshPipelineDesc.fragmentFunction = fragmentFunc
```

## pipeline state

```swift
// initialize pipeline state object
var meshPipeline: MTLRenderPipelineState! 
do {
    meshPipeline = try device.makeRenderPipelineState(descriptor: meshPipelineDescriptor)
} catch {
    print(“Error when creating pipeline state: \(error)\”) }
```

## encode
```swift
var encoder = commandBuffer.makeRenderCommandEncoder(descriptor: desc)!

encoder.setRenderPipelineState(meshPipeline)

encoder.setObjectBuffer(objectBuffer, offset: 0, atIndex: 0)
encoder.setMeshTexture(meshTexture, atIndex: 2)
encoder.setFragmentBuffer(fragmentBuffer, offset: 0, atIndex: 0)

let oGroups   = MTLSize(width: kTrianglesPerModel, height: 1, depth: 1)
let oThreads  = MTLSize(width: kHairsPerBlock, height: 1, depth: 1)
let mThreads  = MTLSize(width: kThreadsPerHair, height: 1, depth: 1)
encoder.drawMeshThreadgroups(oGroups, threadsPerObjectThreadgroup: oThreads, threadsPerMeshThreadgroup: mThreads)

encoder.endEncoding()
```

# Meshlet culling

Splitting scene meshes into smaller pieces called meshlets.
Splitting scene geometry improves granularity, fine-grained culling etc.

* GPU-driven rendering
* fine-grained meshlet culling

culling done within a single encoder, no intermediate buffer to encode draw commands.

we generate meshlets, fragments of geometry.

up to you to map your scene into an object grid.  Object shader will determine your visibility.  Special workload for what is being presented in the final image.

mesh shader processes meshlets and constructs metal meshes.  Mesh grid size is flexible etc.

# Wrap up
* modern, flexible geometry pipeline
* procedural rendering
* meshlet culling

**AVAILABLE IN**: mac2 and apple7.

apple7: a14+, m1+

Note that iOS 16 deploys back to a11 / apple4.

# coda on "mac2"

Did some r&d on "mac2".  It seems to be most macs.  By GPU:
intel HD graphics 5xx
intel iris plus graphics 6xx
AMD radeon, radeon pro, firepro

By model:
iMac 2014+
iMac pro 2017+
Mac pro (late 2013)
macbook (2016+)
mbp (2016+)

Looking at ventura requirements...
all macbooks ok
mba - I thnk so?
mbp - yes
mac mini - I think so?
iMac - yes
iMac pro - yes
mac pro - yes

So mac2 seems to just be any supported mac.









