#mps 

* GPU-accelerated primitives
* Image processing
* Linear algebra
* Ray tracng
* Machine learning
* Optimized for iOS, iPadOS, tvOS, macOS

# Graph

* General purpose compute graph
* Supported on macOS, iOS, iPadOS, tvOS

[[Build customized ML models with the Metal Performance Shaders Graph]]

# Training and Inference acceleration
* Adopted by high level frameworks
* Improved performance of MPS kernels and MPSGraph
* Translated to performance gains for ML frameworks

## Tensorflow acceleration
We developed a new metal plugin to release in TensorFlow 2.5.

Brings the power of Metal to tensorflow using MPS
Supported on macOS
Existing TensorFlow scripts work out of the box

`pip install tensorflow-macos
pip install tensorflow-metal`

Will be available on Python package repo.  

## Core ML

* Apple ML Inference framework

[[Tune your Core ML models]]


# New operations
## Control dependencies
* Operations that must be explicitly ordered in the graph
* Also prevents operations from being optimized away
* Commonly used in batch normalization

Batch nroamlziation
1.  compute mean and variance
2.  Update running_mean and running_variance for inference
3.  Training graph does not require so might be optimized out

Can explicitly make the final step depend on these.

```swift
// Execute the graph
let results = graph.run(feeds: [inputTensor: inputs],
                        targetTensors: [exp],
                        targetOperations: [assign])
```

This requires the developer to track dependencies globally.  Instead,

```
// Create control dependency

let exp = graph.controlDependency(with: [assign],
                                  dependentBlock: { 
                                      return [graph.exponent(with: input, 
                                                             name: nil)]
                                  },
                                  name: nil)

// Execute the graph

let results = graph.run(feeds: [inputTensor: inputs],
                        targetTensors: [exp],
                        targetOperations: nil)
```


## Stencil
* Sliding iwndow operations
* Fnite element methods
* Certain NN layers
* Custom local window operations
* Image convolution
* Laplace operator

Sort of like an alpha mask, you mark the pixels you use?

* Weighted reduction of input tensor over window
* Reduction and padding modes
* Supports stitching in MPSGraph

* Used to normalize in the channel dimension


## Gather
* Efficient access to arbitrary non-contiguous memory
* Embedding lookup
* Dynamic matrix copy

* Copy data from anywhere in an N-dimensional input
* Input coordinates are vectors
* Unpecified dimensions result in slice copies
* Result tensor is a 2D matrix

## Embedding lookup
Common operation used to find embedding vectors for input objects.  Embedding matrix associates each word in vocabulary to an embedding vector.

Words in input can be indices to an embedding operation.  Get corresponding rows for each row ID.

We specify 1 coordinate to copy entire row for each input word.
# Compilation flexibility
`MPSGraphExecutable`.  Improves performance
1.  Gives developer control on when to compile
2.  Reduce graph compilation calls

## Control

When you run MPS graph, we do
1.  Compile
2.  MPSGraphExecutable
3.  Execute

Seamlessly cache the executable.  Users now have flexibility to invoke compilation AOT.  Run directly on `MPSGraphExecutable`.

```swift
// Create the graph

let placeholder0 = graph.placeholder(shape: [1, 3], 
                                     dataType: .float32, 
                                     name: nil)

let placeholder1 = graph.placeholder(shape: [2, 1], 
                                     dataType: .float32, 
                                     name: nil)

let addTensor = graph.addition(placeholder0, 
                               placeholder1, 
                               name: nil)

// Compile the graph into an executable

let executable = graph.compile(with: nil,
                               feeds: [placeholder0: MPSGraphShapedType(shape: [1, 3], 
                                                                        dataType: .float32),
                                       placeholder1: MPSGraphShapedType(shape: [2, 1], 
                                                                        dataType: .float32)],
                               targetTensors: [addTensor],
                               targetOperations: nil,
                               compilationDescriptor: nil)

// Execute the graph into an executable

let fetch = executable.run(with: commandQueue,
                           inputs: [MPSGraphTensorData(input0),        
                                    MPSGraphTensorData(input1)],
                           results: nil,
                           executionDescriptor: nil)
```
## Type inference
* INfer tensor shapes when not specified by user
* By type inference we can figure out e.g. the size of output matrix based on sizes of input matrix
* Changing sequence length of image sizes might be going into the network (dynamically known)
* Previously, for every new image, compilation for type inference is invoked.

Now with control, you can invoke compilation with type inference *turned off*.  We infer types just-in-time.  Tradeoff between compilation time vs most optimal graph.

```swift
// Create the graph compilation descriptor

let descriptor = MPSGraphCompilationDescriptor()

// Disable type inference

descriptor.disableTypeInference()

// Compile the graph into an executable

let executable = graph.compile(with: nil,
                               feeds: /* feeds */,
                               targetTensors: /* target tensors */,
                               targetOperations: nil,
                               compilationDescriptor: descriptor)

// execute the graph
```


# Control flow

* Dispatch operations based on evaluated tensors
	* Batch norrmalization
	* RNNs

## While loop
1.  Predicate graph
2.  Evaluated on CPU with explicit memory sync
3.  If predicate is true, graph is re-executed
4.  If false, loop ends and second graph is created to consume result

Now these can be launched as part of an MPSGraph.  You don't have to introduce explicit memory synchronization primitives.

With synchronization, CPU has to wait for GPU.  And GPU has to wait for CPU sync.  Happens in each iteration.

Now we do this all on GPU, and kernel can be launched iwthout bubbles.

## New APIs
* If/else
* For loop
* while loop

### if/else
* Executes branches based on a predicate.

Maybe want different behavior vs training and inference?
```swift
// Different behavior during inference and training

let results = graph.if(isTraining,
                       then: { ... },    // compute mean and variance
                       else: { ... },    // use running_mean and running_variance
                       name: nil)
```

ex
```swift
let predicate = graph.lessThan(a, 
                               b, 
                               name: nil)

let results = graph.if(predicate,
    then: {[
        graph.addition(a, 
                       b, 
                       name: nil)
    ]},
    else: {[
        graph.subtraction(a, 
                          b, 
                          name: nil)
    ]},
    name: nil)
```

## For loop
* Executes a set of operations a fixed number of times
* Example - RNNs

```swift
var result = input0

for i in 0..<4 {
    result *= input1
}
```

```swift
// Initialize inputs

let input0 = graph.placeholder(shape: [], 
                               dataType: .int32, 
                               name: nil)

let input1 = graph.placeholder(shape: [], 
                               dataType: .int32, 
                               name: nil)
        
let numberOfIterations = graph.constant(4, 
                                        shape: [], 
                                        dataType: .int32)
// Create for loop operation

let result = graph.for(numberOfIterations: numberOfIterations,
                       initialIterationArguments: [input0],
                       body: body)
```

## While loop
* Executes a set of operatons while a condition is met
* Can do do-while by swapping body and predicate

```swift
var result = initialValue

while result < threshold {
    result *= multiplier
}
```

```swift
// Evaluate condition

let condition = {
    (inputs: [MPSGraphTensor], returnTensors: NSMutableArray) -> MPSGraphTensor in
        let predicate = graph.lessThan(inputs[0], threshold, name: nil)
        returnTensors.add(inputs[0])
        return predicate
}
// Define body

let body = {
    (inputs: [MPSGraphTensor]) -> [MPSGraphTensor] in
        let iterationResult = graph.multiplication(inputs[0], multiplier, name: nil)
        return [iterationResult]
}
// Create while loop operation

let results = graph.while(initialInputs: [initialValue],
                          before: condition,
                          after: body,
                          name: nil)
```

# Image composition ex
Laplacian edge filter with linear solver
Inputs
1.  Background iamge
2.  source image
3.  mask of source

Solution
* Laplacian edge detector
* Linear solver

Outputs: composite image

## Laplacian edge detector
* Apply the laplacian edge filter on the source image

```swift
// Apply the laplacian edge filter on the source image

let edges = graph.stencil(with: source, 
                          weights: laplacianWeights, 
                          descriptor: desc, 
                          name: nil)
```

Weights like 

| 0  | -1 | 0  |
|----|----|----|
| -1 | 4  | -1 |
| 0  | -1 | 0  |

Pretty much, this pulls out edges in the source image.
## Linear solver

This is some iterative process where we keep processing the same thing over and over again.  Solution improves.

Terminates when error is below a tolerance.

## Demo
Consider initializing with cloned image rather than source image as this will converge faster.

# Wrap up
* Accelerating CoreML and TensorFlow
* Compute primitives
* compilation flexibility
* Control flow


* https://developer.apple.com/documentation/metalperformanceshaders/training_a_neural_network_with_metal_performance_shaders
* https://developer.apple.com/documentation/metalperformanceshaders
* https://developer.apple.com/documentation/metal
* https://developer.apple.com/metal/metal-shading-language-specification.pdf

