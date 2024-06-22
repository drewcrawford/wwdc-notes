Discover how you can use BNNSGraph to accelerate the execution of your machine learning model on the CPU. We will show you how to use BNNSGraph to compile and execute a machine learning model on the CPU and share how it provides real-time guarantees such as no runtime memory allocation and single-threaded running for audio or signal processing models.
### Create the graph - 0:01
```swift
// Get the path to the mlmodelc. 
NSBundle *main = [NSBundle mainBundle];
NSString *mlmodelc_path = [main pathForResource:@"bitcrusher" ofType:@"mlmodelc"];

// Specify single-threaded execution. 
bnns_graph_compile_options_t options = BNNSGraphCompileOptionsMakeDefault();
BNNSGraphCompileOptionsSetTargetSingleThread(options, true);

// Compile the BNNSGraph. 
bnns_graph_t graph = BNNSGraphCompileFromFile(mlmodelc_path.UTF8String, NULL, options);
assert(graph.data);
BNNSGraphCompileOptionsDestroy(options);
```

### Create context and workspace - 0:02
```swift
// Create the context. 
context = BNNSGraphContextMake(graph);
assert(context.data);

// Set the argument type. 
BNNSGraphContextSetArgumentType(context, BNNSGraphArgumentTypePointer);

// Specify the dynamic shape. 
uint64_t shape[] = {mMaxFramesToRender, 1, 1};
bnns_graph_shape_t shapes[] = { 
    (bnns_graph_shape_t) {.rank = 3, .shape = shape}, 
    (bnns_graph_shape_t) {.rank = 3, .shape = shape} 
};
BNNSGraphContextSetDynamicShapes(context, NULL, 2, shapes);

// Create the workspace. 
workspace_size = BNNSGraphContextGetWorkspaceSize(context, NULL) + NSPageSize();
workspace = (char *)aligned_alloc(NSPageSize(), workspace_size);
```

### Calculate indices - 0:03
```swift
// Calculate indices into the arguments array. 
dst_index = BNNSGraphGetArgumentPosition(graph, NULL, "dst");
src_index = BNNSGraphGetArgumentPosition(graph, NULL, "src");
resolution_index = BNNSGraphGetArgumentPosition(graph, NULL, "resolution");
saturationGain_index = BNNSGraphGetArgumentPosition(graph, NULL, "saturationGain");
dryWet_index = BNNSGraphGetArgumentPosition(graph, NULL, "dryWet");
```

### Execute graph - 0:04
```swift
// Set the size of the first dimension. 
BNNSGraphContextSetBatchSize(context, NULL, frameCount);

// Specify the direct pointer to the output buffer. 
arguments[dst_index] = { 
    .data_ptr = outputBuffers[channel], 
    .data_ptr_size = frameCount * sizeof(outputBuffers[channel][0]) 
};

// Specify the direct pointer to the input buffer. 
arguments[src_index] = { 
    .data_ptr = (float *)inputBuffers[channel], 
    .data_ptr_size = frameCount * sizeof(inputBuffers[channel][0]) 
};

// Specify the direct pointer to the resolution scalar parameter. 
arguments[resolution_index] = { 
    .data_ptr = &mResolution, 
    .data_ptr_size = sizeof(float) 
};

// Specify the direct pointer to the saturation gain scalar parameter. 
arguments[saturationGain_index] = { 
    .data_ptr = &mSaturationGain, 
    .data_ptr_size = sizeof(float) 
};

// Specify the direct pointer to the mix scalar parameter. 
arguments[dryWet_index] = { 
    .data_ptr = &mMix, 
    .data_ptr_size = sizeof(float) 
};

// Execute the function. 
BNNSGraphContextExecute(context, NULL, 5, arguments, workspace_size, workspace);
```

### Declare buffers - 0:05
```swift
// Create source buffer that represents a pure sine wave. 
let srcChartData: UnsafeMutableBufferPointer<Float> = { 
    let buffer = UnsafeMutableBufferPointer<Float>.allocate(capacity: sampleCount) 
    for i in 0 ..< sampleCount { 
        buffer[i] = sin(Float(i) / ( Float(sampleCount) / .pi) * 4) 
    } 
    return buffer 
}()

// Create destination buffer. 
let dstChartData = UnsafeMutableBufferPointer<Float>.allocate(capacity: sampleCount)

// Create scalar parameter buffer for resolution. 
let resolutionValue = UnsafeMutableBufferPointer<Float>.allocate(capacity: 1)

// Create scalar parameter buffer for resolution. 
let saturationGainValue = UnsafeMutableBufferPointer<Float>.allocate(capacity: 1)

// Create scalar parameter buffer for resolution. 
let mixValue = UnsafeMutableBufferPointer<Float>.allocate(capacity: 1)
```

### Declare indices - 0:06
```swift
// Declare BNNSGraph objects. 
let graph: bnns_graph_t 
let context: bnns_graph_context_t 

// Declare workspace. 
let workspace: UnsafeMutableRawBufferPointer 

// Create the indices into the arguments array. 
let dstIndex: Int 
let srcIndex: Int 
let resolutionIndex: Int 
let saturationGainIndex: Int 
let dryWetIndex: Int
```

### Create graph and context - 0:07
```swift
// Get the path to the mlmodelc. 
guard let fileName = Bundle.main.url(forResource: "bitcrusher", withExtension: "mlmodelc")?.path() else {  
    fatalError("Unable to load model.") 
} 

// Compile the BNNSGraph. 
graph = BNNSGraphCompileFromFile(fileName, nil, BNNSGraphCompileOptionsMakeDefault()) 

// Create the context. 
context = BNNSGraphContextMake(graph) 

// Verify graph and context. 
guard graph.data != nil && context.data != nil else { 
    fatalError()
}
```

### Finish initialization - 0:08
```swift
// Insert code snippet.
```

### Create arguments array - 0:09
```swift
// Copy slider values to scalar parameter buffers. 
resolutionValue.initialize(repeating: resolution.value) 
saturationGainValue.initialize(repeating: saturationGain.value) 
mixValue.initialize(repeating: mix.value) 

// Specify output and input arguments. 
var arguments = [(dstChartData, dstIndex), (srcChartData, srcIndex), (resolutionValue, resolutionIndex), (saturationGainValue, saturationGainIndex), (mixValue, dryWetIndex)] 
    .sorted { a, b in a.1 < b.1 } 
    .map { 
        var argument = bnns_graph_argument_t() 
        argument.data_ptr = UnsafeMutableRawPointer(mutating: $0.0.baseAddress!) 
        argument.data_ptr_size = $0.0.count * MemoryLayout<Float>.stride 
        return argument 
    }
```

### Execute graph - 0:10
```swift
// Execute the function. 
BNNSGraphContextExecute(context, nil, arguments.count, &arguments,
```
# Resources
