#mps
GPU-accelerated primitives
* image processing
* linear algebra
* raytracing
* machine learning

Supported on iOS, iPadOS, tvOS, and macOS.

# Improved performance
Improved `MPSImageEuclidianDistanceTransform`
15x faster features in Final Cut Pro on iMac Pro

Added `MPSImageEDLines` edge detection kernel
10x faster line detection in the Measure app on iPad Pro
Better line segment detection in a higher resolution

# Float32 support for traning
Full FP32 support in `MPSCNNConvolution` and `MPSCNNFullyConnected` layers
* no need to change hyper-paramaters to train

Use `kernelWieghtsDataType()` and set to Float32

[[Metal for Accelerating Machine Learning - 18]]
# New `MPSNDArray` primitive
Supports up to 16 dimensions
Dimension sizes can go up to ~1b with no alignment restrictions

Use an `MPSNDArrayDescriptor` to create an `MPSNDArray`

```swift
let descriptor = MPSNDArrayDescriptor(dataType:.float32, shape: [1,3])
let ndarray = MPSNDArray(device: device, descriptor: descriptor)
```

Can convert between metal buffers.

# New Metal Performance Shaders Graph framework

* general purpose compute graph
* Supported on macOS, iOS, iPadOS, tvOS
* New framework, link with

## basic building blocks
Symbolically represeents a DAG of operations and tesnor inputs/outputs
Operationsa re nodes, representing a unit of computation.
Tensors are the edges of the graph.  
### `MPSGraph`
Symbolic representations of all operations, inputs, and outputs.
Retains references to all tensors and operations.
### `MPSGraphTEnsor`
Symbolically represent data
* shape
* datatype
* defining operation


```swift
let constantTensor = graph.constant(0.2, shape: [2,4], dataType:.float32)
```
| 0.2 | 0.2 | 0.2 | 0.2 |
|-----|-----|-----|-----|
| 0.2 | 0.2 | 0.2 | 0.2 |

### `MPSGraphOperation`
A unit of computation
* input tensors
* Output tensors
* Control dependencies (special edge).  Graph executes before the current operation, even if op itself does not depend

```swift
let addTensor = graph.addition(tensor1, tensor2, name: nil)
```

#### Reshape
```swift
let reshapedTensor = graph.reshapeTensor(placeholderTEnsor, shape: [2,4], name: nil)
```

#### slice
```swift
let slicedTensor = graph.sliceTensor(reshapedTensor, dimension: -1, start: 2, length: 1, name: nil)
```

#### concat
```swift
let concatenatedTensor = graph.concatTensors([slicedTensor, reshapedTensor], dimension: 1, name: nil)
```

#### transpose
#### zero-length slice
#### many more



## writing a custom compute function
### GeLU -> Gaussian Error Linear Unit
```
GeLU(x) = (1.0 + erf(x * sqrt(0.5))) * 0.5 * x
```
1.  Define inputs
2.  create operations
3.  execute

### placeholder
Placeholder ops represent inputs to the graph during runtime
```swift
let placeholderTensor = graph.placeholder(shape: [2,3], dataType:.float32, name: nil)
```

### create operations
```swift
class MyMPSGraph: MPSGraph {
	func GeLU(input: MPSGraphTensor) -> MPSGraphTensor {
		let ones = constant(1.0, shape: [1], dataType:.float32)
		let half = constant(0.5, shape: [1], dataType:.float32)
		let sqrt = squareRoot(with: half, name: nil)
		let multiply = multiplication(sqrt, input, name: nil)
		let multiply2 = multiplication(half,input,name:nil)
		let erf = erf(with: multiply, name: nil)
		let add = addition(erf, ones,, name: nil)
		return multplication(multiply2, add, name: nil)
	}
}
```

## execute
Use the run method to execute the graph

### MPSGraphTensorData
Abstracts MPS types and Metal resources
```swift
let tensorData = MPSGraphTensorData(_ : MTLBuffer, shape: [NSNumber], dataType:MPSDataType)
```
Can abstract over MPSVector / MPSMatrix, MPSImageBatch, and MPSNDArray


```swift
let graph = MyMPSGraph()
let inputTensor = graph.placeholder(shape:nil,dataType:.float32, name: nil)
let geLU = graph.GeLU(input: inputTensor)
let inputs = MPSGraphTensorData(inputData)
let results = graph.run(feeds: [inputTensor: inputs], targetTensors: [geLU], targetOperations: nil)
```

###  Stitching
MPSGraphCompiler fuses operations, avoiding unnecessary memory traffic?

Set of element-wise math operations
MPSGraph compiler creates a region of operations to fuse
Metal Compiler fuses the operations together to create a single fused kernel

#### optimization
Hand-tuned compute kernels
Adjacent element-wise math operations

### inference

## LSTM Text Generation Network
Works on a single character input.
Let's say we want to generate ROMEO.
Feed O into character embedding layer.  This generates vector
Feed vector into LSTM layer.  
Feed that output to character un-embedding layer.
Update LSTM layer for next iteration
Multiply the un-embedding output by inverse temperature
Feed that to Softmax
Sample output
Get index of the next character.

## Training neural networks with `MPSGraph`s
Repeatedly attempting to execute the network on known data.
[[Using Metal 2 for Compute - 17]]
[[Metal for Accelerating Machine Learning - 18]]

### Digit classification
1.  Define inputs
2.  Create variables
3.  Create inference
4.  Create loss
5.  Create gradients
6.  Create updates
7.  execute

#### Defining inputs
Sizes are determined through type inference at runtime
```swift
//placeholder created with unknown batchSize (-1)
let inputTensor = graph.placeholder(shape: [-1,28,28,1],
dataType: .float32, name: nil)
let labelTensor = graph.placeholder(shape: [-1,10], dataType:.float32, name: nil)
```

#### Create trainable parameters (variables)
Here we have weights and baises.

##### `MPSGraphVariableOp`
Retains values across graph runs for you
```swift
let variableTensor = graph.variable(with: Data(bytes: floatValues, count: 4), shape: [2,2], dataType:.float32)
```
#### Inference or forward pass
This involves creating a "convolution" layer
First, we create a `MPSGraphConvolution2DOpDescriptor`.
Next we pass this to a `MPSGraph.convolution2D`

Padding style `TF_SAME` and `TF_VALID`, as well as custom padding.

##### `MPSGraphTensorNamedDataLayout`
attach names, maybe for debugging?
Image formats
* NCHW (BatchSize x channels x height x width)
* NHWC

Weight formats
* OIHW (OutputChannels x inputChannels x kernelHeight x kernelWidth)
* HWIO

##### `MPSGraphVariableOp`
Takes a snapshot of the variable value during graph execution
```swift
//op to read a value tensor from a variable
let readValue = graph.read(_ variable: MPSGraphTensor, name: String?)
```
##### Example
```swift
let convDesc = MPSGraphConvolution2DOpDescriptor(strideInX: 1, strideInY: 1, dilationRateInX: 1, dilationRateInY: 1, groups: 1, paddingStyle:.TF_VALID, dataLayout: .NCHW, weightsLayout: .HWIO)

let weightsReadVariabe = graph.read(weightsVariable, name: nil)
let convTensor = graph.convolution2D(inputTensor, weight: weightsReadVariable, descriptor: convDesc, name: "convolutionLayer")
```
Note that graph can implicitly add read variable ops

##### bias
For bias add,
```swift
let biasTensor = graph.addition(convTensor, biasVariable, name: "biasAdd")
```
##### ReLU
```swift
let reLUTensor = graph.reLU(with: biasTensor, name: "reLU")
```
##### reshape
```swift
let reshapedTensor = graph.reshapeTensor(reLUTensor, shape: [-1, 10], name: "reshape")
```
-1 -> Unknown batch size, 10 -> number of classes to classify (digits)
##### softmax
```swift
let softMaxTensor = graph.softMax(with: reshapeTensor, axis: -1, name: "softMax")
```
axis -1 -> indicates fastest-moving axis

#### loss
```swift
let lossTensor = graph.softMaxCrossEntropyLoss(reshapeTensor, labels: labelTensor, axis: -1, reductionType: .sum, name: nil)
```
Accumulates loss across batch into single scalar loss value

#### gradient
Backpropagation across the chain rule.
delta L / delta L = 1

Automatic differentiation creastes the backward pass automatically according to the chain rule

```swift
let gradients = graph.gradient(of: loss, with: [weightsVariable, biasesVariable], name: nil)
```
#### updates
Stochastic gradient descent opttimizer

```swift
let weightsUpdate = graph.stochasticGradientDescent(learningRate: learningRate, values: weightsVariable, gradient: gradients[weightsVariable], name: nil)
```

#### execute
```swift
for i in 0..<NumEpochs {
	//get inputs and labels for current iteration
	let inputs = MPSGraphTensorData(inputData)
	let labels = MPSGraphTensorData(labelsData)
	
	//execute graph to update variables and return loss for the sample
	let results = graph.run(feeds: [inputTensor: inputs,
	labelTensor: labels],
	targetTensors: [lossTensor],
	targetOperations: [assignWieghts, assignBiases])
}
```

Can also be run asycnhronously with `runAsync`.  Useful for overlapping CPU encoding with GPU execution.  Completion handler.

Can encode on a command queue or command buffer

#### demo
### digit generation
Generative adversarial network (GAN)
1.  Generator
2.  Discriminator

Generator network is "quite simple".  Proceeds to display a long list of stages.  Random layers out-of-order include

* convolution transpose.  This somehow relates a gradient and a convolution.  
* BatchNorm.  "Normalization helps us ensure our values don't vanish or explode".  Somehow based on descriptive statistics, like mean or variance.

For the Discriminator, we have a similar mess of layers.  This is 1 class, because we only determine if it's a real digit or not.

* Dropout.  Randomly dropout values in a tensor at a provided rate.  "Boosts the other values to regain the energy"

#### training
Not sure I followed this at all

Execute discriminator and generator together.  MPSGraph optimizes across them.

#### demo

# Summary
Introduced MPSGraph
Writing and executing a custom function
Graph optimizations
Training neural networks

# Resources
https://developer.apple.com/documentation/metalperformanceshadersgraph/adding_custom_functions_to_a_shader_graph
https://developer.apple.com/documentation/metalperformanceshaders
https://developer.apple.com/documentation/metal

