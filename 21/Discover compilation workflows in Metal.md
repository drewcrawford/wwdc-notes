#metal 

# Metal shading language evolution
Resembles the CPU compilation model
Share and reuse utility code
Change pipeline behavior at runtime
Reuse GPU binaries

# ex
Fragment shader that is either `foo` or `bar`.
If this function is multiple pipelines, might want to compile once.  Might need to link a different implementation.  Or fragment shader is extensible to handle new statements.
Might want to call user-provided function.

Many different requirements.  Metal offers various APIs to support different implementations.  

# Dynamic libraries in render
* Shared object file
* Reusable across pipelines
* Dynamically link, load, and share GPU binaries

[[Build GPU binaries with metal]]

Added support to share utility libraries acrosss compute and render workflows.

High volume of shared code.  Can now manage large utility code and share across workloads.  Pipeline updates and recompilation.  Can even swap out functions at runtime by changing libraries that are loaded in the pipeline.
Allow users to inject code.

How to buidl and work with them?

Implementation exists in metallib.  

1.  Build source to AIR.  Xcode or `newLibraryWithSource`.
2.  Create `newDynamicLibrary` API from AIR.  
3.  Serialize and reuse `serializeToURL`
4.  `newDynamicLibraryWithURL`

Shading language
Declare external functions

```cpp
// Declare external functions

extern float4 foo(FragmentInput input);
extern float4 bar(FragmentInput input);

// Use functions in shader

fragment float4 main(FragmentInput input [[stage_in]])
{
    switch(condition(input)) 
    {
        case 0: 
            return foo(input);
        case 1:
            return bar(input);
    }
}
```

Possible to replace this with something different at runtime. 
`descriptor.preloadedFragmentLibraries = @[dylibB]`

Resolved in array order.

Platform support

|  | macOS | iOS |
|-|-|-|
| Compute | `MTLDevice.supportsDynamicLibrarires` or Apple7+ | Apple6+ |
| Render | Apple7+ | Apple6+ |
| Tile | Apple7+ | Apple6+ |

In practice, iOS seems to be A13+.

On my intel mbp,


```swift
        for device in MTLCopyAllDevices() {
            if #available(macOS 11.0, *) {
                print("\(device.name) \(device.supportsDynamicLibraries)")
            } else {
                // Fallback on earlier versions
            }
        }
```
```
**AMD Radeon Pro 5500M false**

**Intel(R) UHD Graphics 630 true**
```


# Function pointers in render
[[Get to know Metal function pointers]]

Allowing you to call functions that we haven't seen before.  

* Render and tile pipelines
* Call to unknown code
* Function tables – change behavior dynamically, either by binding a different table, or indexing into table
* Binary or AIR.  

Setup
* Declare and instantiate visible functions
* Configure pipeline descriptor
* Create and populate visible function table

Usage
* Encoding and calling function pointers

## Declare and instantiate
```objc
// Declare a descriptor and set CompileToBinary options

MTLFunctionDescriptor* functionDescriptor = [MTLFunctionDescriptor new];
functionDescriptor.options = MTLFunctionOptionCompileToBinary;

// Backend compile the function

functionDescriptor.name = @"foo";
id<MTLFunction> foo = [library newFunctionWithDescriptor:functionDescriptor
```

Function compiled by GPU Backend copmiler

Provide list of functions that pipeline stage may call

```objc
// Provide a list of functions that the pipeline stage may call

// AIR functions
//compiler statically links visible functions, allowing backend compiler to optimize the code.
renderPipeDesc.fragmentLinkedFunctions.functions = @[foo, bar, baz];

// Binary functions
// inform the driver which externally compiled functions are callable
renderPipeDesc.fragmentLinkedFunctions.binaryFunctions = @[foo, bar, baz];
```

### avoiding stack overflow
If the code you are calling has a complex callchain, important to specify maximum stack depth.  The compiler cannot do static analysis to determine the depth.

maximum stack depth in stack frames per stage.
Keep as small as possible for maximum occupancy
`renderPipelineDesc.maxFragmentCallStackDepth = N`

Create visible function table
```objc
// Create visible function table

    [renderPipeline newVisibleFunctionTableWithDescriptor:stage:];

// Create function handles

    [renderPipeline functionHandleWithFunction:stage:];

// Insert handles into table

    [visibleFunctionTable setFunction:atIndex:];
```

Both handles and table are specific to pipeline and stage.

Encoding and calling

```objc
// Bind visible function table objects to each stage

    [renderCommandEncoder setFragmentVisibleFunctionTable:atBufferIndex:];

// Usage in shader

   fragment float4 shaderFunc(FragmentData vo[[stage_in]],
                              visible_function_table<float4(float3)>materials[[buffer(0)]])
   {
   		 //...
     
       return materials[materialSelector](coord);
   }
```

Not uncommon to create a pipeline to find out later that you need to access additional functions.  Could create a second pipeline... but that ? pipeline compliation.

Now you can create an extended pipeline.  This way a new pipeline can be created faster from an existing pipeline and use the original pipeline descriptors.

```objc
// Enable incrementally adding binary functions per stage

renderPipeDesc.supportAddingFragmentBinaryFunctions = YES;

// Create render pipeline functions descriptor

MTLRenderPipelineFunctionsDescriptor extraDesc;
extraDesc.fragmentAdditionalBinaryFunctions = @[bat];

// Instantiate render pipeline state

id<MTLRenderPipelineState> renderPipeline2 =
  [renderPipeline1 newRenderPipelineStateWithAdditionalBinaryFunctions:extraDesc
 ```
 
 Feature table
 

|  | macOS | iOS | Device query |
|-|-|-|-|
| Compute | Mac2/Apple7+ | Apple6+ | `supportsFunctionPointers` |
| Render | Apple7+ | Apple6+ | `supportsFunctionPointersFromRender` |
| Tile | Apple7+ | Apple6+ | `supportsFunctionPointersFromRender` |

As far as non-render pointers, both my mac GPUs are supported.

# Adding functions to archives
Compiling shaders can be time-intensive.  
Collect and store compiled executable functions
Saving compilation time on subsequent runs
Reducing memory calls

Adding `visible` and `intersection` functions to binary archives, reducing overhead.

[[Build GPU binaries with metal]]

## Storng binary function pointers
`.options = CompileToBinary
addFunctionWithDescriptor`

To load, use `.binaryArchives = @[..]; newFunctionWithDescriptor`.

If any archives has the compiled function pointer, it will be returned without compilation.

1.  `newFunctionWithDescriptor`
2.  Found in binary archives?  Return 
3.  Not found?  check `CompileToBinary`
4.  Return AIR-only if not compiletobinary
5.  FailOnBinaryArchiveMiss?
	1.  Compile
	2.  return nil

Use the same archive to store all GPU-compiled code.  

When using a pipeline, you might want to cache AIR function pointers.  But what should you cache when you have a pieline with various fp combinations?

e.g., suppose we have 3 fragment shaders, which are identical.  Except they differ in which fp is used.

If using AIR, need to cache all permutations of the pipeline.  However, when using binary fp, enough to cache a single variant because the pipeline doesn't change when a new function is added to it.  Can use that archive to add other variants, independent of which variants are used.

 Binary archives are available on all devices.  But caching fp is available on
 macOS - mac2/apple7+
 ios - apple6+
 device query: `supportsFunctionPointers`, `supportsFunctionPointersFromRender`
 
 That is, it depends on fp availability
 
# Private linked functions
Sometimes oyu want statically linked functions.  Last year, we added linked funtions with support for statically linking AIR functions.  However, this requires fp support.

This year, we're introducing `privateFunctions`.  Linked statically at the AIR level, but since they're private, no ? can be made for function pointers.  Fully optimize shader code.

all devices with new OS.
# Function stitching

Some apps need to render dynamic content with a runtime.  ee.g., complex compute kernel etc.  Fucntion stitching is a great tool.

Prior to function stitching, have to manipulate metal source strings.  This is inefficient, and Metal->AIR can be expensive.

* Computation graph
* computation graph is a DAG
	* Input nodes
	* Function nodes
	* Data edges (how data flows from one node to another)
	* Control edges (order in which function calls should be executed)

## Stitchable functions
Functions must have `[[stitchable]]` attribute.  Visible function, which can be used with function stitching API

Fetched from a precompiled library.  
Stitching process generates a visible function in AIR

```cpp
[[stitchable]] int FunctionA(device int*, int) {…}
[[stitchable]] int FunctionC(int, int) {…}

[[stitchable]]
int ResultFunction(device int* Input0,
                   int Input1, 
                   int Input2)
{
  int N0 = FunctionA(Input0, Input1);
  int N1 = FunctionA(Input0, Input2);
  int N2 = FunctionC(N0, N1);    
  return N2;
}
```

Stitcher infers arguments to functions.  

Stitcher than generates a function equivalent that puts together A and C.

Can generate a library containing such functions.  

How to use?

```objc
// Create input nodes

  inputs[0] = [[MTLFunctionStitchingInputNode alloc] initWithArgumentIndex:0];

// Create function nodes

  n0 = [[MTLFunctionStitchingFunctionNode alloc] initWithName:@"FunctionA"
                                                    arguments:@[inputs[0], inputs[1]]
                                          controlDependencies:@[]];
  n1 = [[MTLFunctionStitchingFunctionNode alloc] initWithName:@"FunctionA"
                                                    arguments:@[inputs[0], inputs[2]]
                                          controlDependencies:@[]];
  n2 = [[MTLFunctionStitchingFunctionNode alloc] initWithName:@"FunctionC"
                                                    arguments:@[n0, n1]
                                          controlDependencies:@[]];

// Create graph

  graph = [[MTLFunctionStitchingGraph alloc] initWithFunctionName:@"ResultFunction"
                                                            nodes:@[n0, n1]
                                                       outputNode:n2
                                                       attributes:@[]];
```

```objc
// Configure stitched library descriptor
  
  MTLStitchedLibraryDescriptor* descriptor = [MTLStitchedLibraryDescriptor new];

  descriptor.functions      = @[stitchableFunctions];
  descriptor.functionGraphs = @[graph];

// Create stitched function

  id<MTLLibrary> lib = [device newLibraryWithDescriptor:descriptor 
                                                  error:&error];

  id<MTLFunction> stitchedFunction = [lib newFunctionWithName:@"ResultFunction"];
```

Stitched fucntion is ready to be used, including as a fucntion in another stitching graph.

This API is available across all devices in new OS.

# Picking the right API
* Dynamic libraries
* Function pointers
* Private linked functions
* Function stitching "improve compilation time"

## Dynamic libraries
A fixed set of many utility functions
Functions don't need to change too frequently

## Function pointers
* Multiple functions that need to be dynamically selected
* Dynamic selection can happen on CPU or GPU
* Function pointers can be "added" to the PSO
* AIR or binary function
* Shader does not need to know anything about these functions, names, number, etc.

## Private functions
* Statically link functions to PSO 
* Internal to pipeline
* Canonnt be included in visible table
* Compiler can do max optimizations
* Supported across all GPU families
* Visible functions directly referenced in the main shader
* Don't need to use fp
* Allow more aggressive optimizations

## Function stitching
* Precompile snippets directly to AIR
* Perform compilation at runtime
* Generate functions dynamically
* Use other functions as building blocks
* Accelerate some compilation workflows

# Wrap up
* Dynamic lbiraries in render
* Function pointers in render
* Adding functions to archives
* Private linked functions
* Function stitching

