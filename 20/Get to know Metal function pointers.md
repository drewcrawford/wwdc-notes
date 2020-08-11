#metal 

`void (*myCallback)(void *myData)`

# Function pointers in metal

Range of use cases
Ray tracing
 * Custom intersection functions

[[Discover ray tracing with metal]]

Seems to be recapping that talk.

Maybe want different functions for lights and for materials.

# visible functions

```cpp
[[visible]] Type Function(...) { ...}
```
We want to manipulate the function from the metal API

`device.supportsFunctionPointers` <- Not every device supports this.

First, we wrap our `Function` in `MTLFunction`

`library.makeFunction(name: "Function")`

```swift
let linkedFunctionsn = MTLLinkedFunctions()
linkedFunction.functions = [mtlFunction]
computeDescriptor.linkedFunctions = linkedFunctions
```

# single compilation pipeline

When we build our pipeline, we can take a copy of these visible functions.

Contains both a specialized version of our kernel, and specialized versions of all our functions
Similar to LTO, where kernel and visible funcions can be optimized based on usage.

But this can increase time to make pipeline, and increase codesize.

Maybe we want to have separate objects?

# Separately-compiled pipeline

Compile each function to standalone GPU binary form and re-use across multiple pipelines

```swift
let functionDescriptor = MTLFunctionDescriptor()
functionDescriptor.name = "Area"
functionDescriptor.options = .compileToBinary //magic
let areaBinaryFunction = try library.makeFunction(descriptor: functionDescriptor)
```

Then when we create the cpd, we do
```swift
linkedFunctions.binaryFunctions = [areaBinaryFunction]
```

This indicates we mean the binary function, not specialize the function inside the compute PSO.

We can mix and match precompiled functions with specialized versions.

#  Compare/contrast
|                        | Single           | separate                     |
|------------------------|------------------|------------------------------|
| Functions              | create as normal | precompile to binary         |
| Linked functions array | `functions`      | `binaryFunctions`            |
| Pipeline size          | Larger           | Smaller                      |
| Creation time          | Longer - LTO     | shorter                      |
| Runtime performance    | Maximum          | Lower - binary call overhead |

# Incremental compilation pipeline
```swift
computeDescriptor.supportAddingBinaryFunctions = true

...

pipeline.makecomputePipelineStateWithAdditionalBinaryFunctions(...)

```

# Visible function tables on GPU

This appears to be a kind of typedef
```cpp
using LightingFunction = Lighting(Light, TriangleIntersectionData);
```

Kernel parameter
```cpp
visible_function_table<LightingFunction> lightingFunctions [[buffer(1)]],
```

Access via index

```cpp
LightingFunction *lightingFunction = lightingFunction[light.index];
Lighting lighting = lightingFunction(light,triangleIntersection);
```

# on CPU
```swift
let vftDescriptor = MTLVisibleFunctionTableDescriptor()
vftDescriptor.functionCount = 3
let lft = pipeline.makeVisibleFunctionTable(descriptor: vftDescriptor)
```

Find and set function by handle:

```swift
let fHandle = pipeline.functionhandle(function: spot)!
lft.setFunction(functionhandle,index: 0)
```

Find and set functions by handle:

```swift
computeComamndEncoder.setVisibleFunctionTable(lft,bufferIndex:1)
argumentEncoder.setVisibleFunctionTable(lft, index:1)
```

# Tables for incremntal pipelines

* reuse vft from ancestor plieplines
* Existing handles are ok
* New handles can be added to existing tables
* Only access new handles from new pipeline


# Performance
## Function groups

Group related functions.  Apply `[[function_groups]]` To give names the the group of possible functions that are called from this location

```cpp
LightingFunction *lightingFunction = lightingFunctions[light.index];
[[function_groups("lighting")]] Lighting lighting = lightingFunction(light, triangleIntersection)
```
Then on CPU, define groups with
```swift
linkedFunctions.groups = ["lighting": [a,b,c],"material: [d,e,f]]"
```


**Can now implement recursive functions**


`computeDescriptor.maxCallStackDepth = 3`

The default is  so that the typical use cases of visible functions work out of the box.
This value is expected to be used for any change of ...
Consider writing iteratively to reduce stack usage.

# Divergence in Ray Tracing
* Ray coherency
* Ray compaction
* Interleaved tiling

[[Ray tracing with metal - 19]]

Note that funciton pointers are another case that lead to divergence.  Consider how divergent the set of fps will be.  Worst case, each thread executes a novel function.

May want to reorder function calls to introduce coherence.

1.  Write all params, thread and function index to threadgroup memory
2.  Sort function calls by function index
3.  Invoke functions in sorted order
4.  Write result to threadgroup memory

