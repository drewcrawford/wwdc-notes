#metal 

4-cores.

# A14 GPU performance
GPU-driven pipelines.
Indirect command buffers enable you to encode commands on GPU timeline.  This frees up CPU time to do other tasks.
A14 makes this more efficient.

Faster threadgroup memory.  Better support parallel computing.
Threadgroup-scoped atomics are now faster.

Ability to conserve memory bandwidth via lossless compression.  Framebuffer compression improves by 15% or more on average.  depth buffer 40%

AppleGPUFamily7.

* Barycentric coordinates and primitive ID
* New texture addressing modes
* SIMD reduction instructions
* SIMD matrix multiply

# Barycentric coordinates and primitive ID
* fragment location P(x,y) within a primitive
* Accsible within the fragment shader

"weight of vertices that form the triangle".  Sum of weights is 1.
In msl, can get via `[[barycentric_coords]]` in a fragment shader.

## Procedural generation
Can render or generate models algorithmically.
Draw custom lines procedurally.

## primitive ID
Tells you which primtiive the current fragment corresponds to in te input geometry.
Accessible within the fragment shader.
If the triangle gets clipped by hardware, child inherits from parent.  For tesselation, simpliy corresponds to patch ID.
primitive id is `uint [[primitive_id]]`

### temporal anti-aliasing 
* temporal AA reduces motion artifacts
* Pixel history used for sample accumulation (prior frame)
* Primitive ID can help validate history data.

## deferred rendering
1.  Generate G-buffer
2.  Consume G-buffer to light

At high resolution or multisampling, g-buffer can be quite large and reading/writing may be a lot of memory bandwidth.

Instead, use a visibility buffer.  Minimizes the work in the geometry stage by simplifying the output from this stage.  

Complexity of geometry stage is reduced, allowing for higher fill rate.
New reconstruction step during the lighting pass ot reconstruct material inputs from minimal vbuffer.

### vbuffer
* primitive ID ot reconstruct vertex data
* Barycentric coordinates to interpolate vertex data

1.  Geometry stage (produces vbuffer)
2.  lighting stage (consumes)

Geometry stage -> vertexstage transsforms poisition only
fragment stage generates primitive ID and barycentric coordinates

In the lighting stage, a new reconstruction step uses a dferred vertex fetch based on primitive id and interpolate with barycentric coordinates.
Material and lighting identical to normal deferred rendering.

Vertex shader only needs position.
Fragment shader generates surface_id and barycentric coordinates
only afterward do we have "very large" buffer/structure (?)

```cpp
// Vertex Shader - Geometry stage

vertex float4 geoVertex(float3* pos   [[buffer(0)]], 
                        float4x4* vpm [[buffer(1)]],
                        Uint vid      [[vertex_id]]) {
   float3 p = pos[vid];          // Position fetch
   return doTransform(p, vpm);   // Transform step
}


// Fragment Shader - Geometry stage

fragment VisBufferFragment geoFrag(uint primid  [[primitive_id]],
                                   float3 baryc [[barycentric_coords]],
                                   uint drawid  [[buffer(0)]]) { 
   VisBufferFragment out;    out.surface_id  = makeID(primid, drawid);   // Combine draw ID and primitive ID
   out.barycentric = baryc.xy; 
   return out;
}
```

lighting stage

```cpp
// Lighting Compute Kernel

kernel lightingCompute(Draws* draws          [[buffer(0)]],
                       texture2d<float4> lit [[texture(0)]]) {

  /* Reconstruction using Visibility Buffer */
  MaterialInput mi  = doReconstruct(surface_id, baryc, draws);
  
  /* Apply material model */
  MaterialOutput mo = doMaterial(mi, mat);
  
  /* Apply lighting function */
  lit[thread_id]    = doLighting(mo, lights);

}
```

How does reconstruction work?  Since other 2 are normal defferred data.

Note that a barycentric coordinate contains 2 values.  The third value can be made from these 2, since the sum of all 3 is unity.

```cpp
MaterialInput doReconstruct (uint32 surface_id, short2 baryc, Draws* draws, Mesh* meshes)
{
    MaterialInput mi;

    /* Retrieve primitive ID and Draw ID */
    uint primid   = surface_id & 0x0000FFFF; uint surface_id = ids >> 16;
    /* Retrieve the 3rd barycentric coordinate */
    float3 bc     = float3(baryc.xy, 1.0 - baryc.x - baryc.y);
    /* Retrieve (mesh ID, vertex ID) using primitive ID and draw ID */
    uint meshid   = draws[drawid].meshid;
    uint3 vertid  = uint3(draws[meshid].ib[primid*3+0], 
                          draws[meshid].ib[primid*3+1], 
                          draws[meshid].ib[primid*3+2]);
    /* Interpolate vertex buffer data cross barycentric coordinates */
    mi.uv  = meshes[meshid].vb[vertid.x].uv.xy * bc.x +
             meshes[meshid].vb[vertid.y].uv.xy * bc.y +
             meshes[meshid].vb[vertid.z].uv.xy * bc.z;
    /* Other draw state such as normals */
    mi.normal     = normalize(draws[drawid].vpm * float4(normal, 0)).xyz;
    
    return mi;
}
```

## New texture addressing modes
Coordinates outside the sampling range, for texture atlas.

1.  Mirror clamp to edge.
	1.  \[-1,1\] mirrored across the axis
	2.  outside of \[-1,1\]: clamped.
2.  Clamp to border color
	1.  transparent black
	2.  opaque black
	3.  opaque white

```swift
/* * rAddressMode : The address mode for the texture depth (r) coordinate.
   * sAddressMode : The address mode for the texture width (s) coordinate.
   * tAddressMode : The address mode for the texture height (t) coordinate.
   * borderColor : The border color for clamped texture values.
 */

let device = MTLCreateSystemDefaultDevice()!

let samplerDesc = MTLSamplerDescriptor()

/* … Program other sample state… */
samplerDesc.magFilter = .linear;

samplerDesc.sAddressMode = .mirrorClampToEdge
samplerDesc.tAddressMode = .mirrorClampToEdge

samplerDesc.rAddressMode = .clampToBorderColor
samplerDesc.borderColor  = .transparentBlack

let samplerState = device.makeSamplerState(descriptor: samplerDesc)!
```

# Compute features
## SIMD reduction
Reduce an array to a single result in a data paralllel way
* compute average luminance
* Apply tone mapping using min/max values

Reduction instructions.  SIMD-scope.

`simd_sum` and `simd_product` give the sum/product across all threads in a simd group.
`simd_minimum` and `simd_maximum` 
floating point and integer types.
now supports reductions with `and` `or` and `xor`.  Only on integer types.

### Talking about thread organization.

Compute dispatches launch threads into a grid.  Divided into threadgroups.  then SIMD groups (32 threads).  SIMDgroups run in lockstep.

SIMD group functions share data between threads using registers.

### SIMD group functions

32 lanes.

simd_sum lets you add all values in a simd group and broadcast the result back to all lanes.  All threads can inspect their copy of the result.
"Inactive threads ar eskipped in the computation correctly, they do not contribute to the final sum."

```cpp
void reduce_sum(device float const* input_array,
                device float* total_sum)
{
     threadgroup float SSA[32];

     float a = input_array[read_offset];

     float simdgroup_sum = simd_sum(a);

     SSA[sg_index] = simdgroup_sum;

	//"note that we need a barrier before we can access the array in threadgroup memory"
     threadgroup_barrier(mem_flags::mem_threadgroup);

     if (simdgroup_index_in_threadgroup == 0)
     {
         *total_sum  = simd_sum(SSA[sg_index]);
     }
}
```

## SIMD matrix multiply
* SIMD group scope
* Buliding blocks for larger operations

`simdgroup_float8x8` and `simdgroup_float4x4`
can then use `simdgroup_multiply` and `simdgroup_multiply_accumulate`.

```cpp
// SIMD 16x16 matrix multiplication

void matmul(threadgroup float const* A, threadgroup float const* B, threadgroup float* C)
{

    simdgroup_float8x8 matA, matB, matC(0.0f);


	//this is address arithmetic necessary to do a row of one and a column of the other.
    A += A_increment(sg_index, A_row_stride);
    B += B_increment(sg_index, B_row_stride);
    C += C_increment(sg_index, C_row_stride);

    for (ushort k = 0; k < 16; k += 8) {
        simdgroup_load(matA, A + k, A_row_stride);
        simdgroup_load(matB, B + k * B_row_stride, B_row_stride);
        simdgroup_multiply_accumulate(matC, matA, matB, matC);
    }
 
    simdgroup_store(matC, C, C_row_stride);

}
```

### MPS
If you're using MPS, 

```swift
// General matrix multiplication
let matMulKernel = MPSMatrixMultiplication(device: device!, resultRows: M, 
    resultColumns: N, interiorColumns: K)
matMulKernel.encode(commandBuffer: commandBuffer, leftMatrix: A, rightMatrix: B,
    resultMatrix: C)

// CNN convolution
let convKernel = MPSCNNConvolution(device: device!, weights: convDataSource)
convKernel.encodeBatch(commandBuffer: commandBuffer, 
    sourceImages: sources, destinationImages: results)

// MPS Graph
let graph = MPSGraph()
let A = graph.placeholder(shape:[M, K], dataType: .float32)
let B = graph.placeholder(shape:[K, N], dataType: .float32)
let C = graph.matrixMultiplication(primaryTensor: A, secondaryTensor: B)
graph.run(feeds: [A, B] targetTensors: C targetOperations: nil)
```

[[Build customized ML models with the Metal Performance Shaders Graph]]

# Wrap up
* barycentric coordinates and primitive id
* New texture addressing modes
* SIMD reduction instructions
* SIMD matrix multiply
* architectural improvements (lossless compression, faster memory, improved indirect dispatch)

