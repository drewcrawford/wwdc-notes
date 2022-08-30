#metal 

Explore new features and enhancements in metal.

# Metal for ML
most computationally expensive process of the pipeline.  GPUs excel at ML workloads.  Metal Performance Shaders.  

High-performance GPU primitives
* iamge processing
* linear algebra
* ray tracing
* machine learning

Edge map for an input image.  Common operation in image segmenetation applications.  This year, canny filter is able to process 4k images up to 8x faster.

MPS graph - general purpose compute graph.  Extends the compute functionality of MPS to multidimensional tensors.

[[Build customized ML models with the Metal Performance Shaders Graph]]

High level ML frameworks built on MPSGraph, such as coreml and tensorflow.

Accelerate tensorflow networks using metal plugin

[[Accelerate machine learning with Metal Performance Shaders Graph]]

3 topics to cover.  First, I'll introduce the newest ML framework coming to apple gpus.  

# PyTorch acceleration
The number one most requested feature was supprot for GPU accel on apple silicon.  We are bringing the power of metal to pytorch using an MPS device backend.  Supported on macOS.
Releasing with PyTorth 1.12.  This backend implements PyTorch operation kernels, ans well as a runtime framework.  Ops call into MPS graph and MPS.  And runtime component uses Metal.

This enables PyTorch to use the highly efficient kernels from MPS, with metal's command queues, etc.  Operation kernels and pytorch MPS eruntime components are part of the open source code and merged into PyTorch repo.
```bash
# v1.12+
python -m pip install torch
```

Please refer to metal developer resources.

```python
import torch

mpsDevice = torch.device("mps" if torch.backends.mps.is_available() else “cpu”)
```

Convert your models and inputs to MPS device.

```python
import torchvision

model = torchvision.models.resnet50()

model_mps = model.to(device=mpsDevice)
```

By default, the model will run on the cpu.  Use the `to` method to convert the model to the MPS device.  Ensures that intermediate tensors will use the accelerated MPS backend.  This example passes a random input tensor to the MPS model.

```python
sample_input = torch.randn((32, 3, 254, 254), device=mpsDevice)

prediction = model_mps(sample_input)
```

To use the backend, you need to provide the device in the arg above.  Finally, pass the sample input to the MPS model to get a prediction.

## demo
StyleTransfer network.  Allows yotu to apply a different artistic style to an image.  

We ahve seen amazing speedups on all these pytorch benchmarks.  On m1 ultra, up to 20x faster, with an average of 8x faster.  PyTorch makes it easy to develop ML models.  

# Enhancements to TensorFlow
Introduced metal plugin in 2.5, through the TF metal plugin.  Since then, additional features have been added.  Improved training with bigger batches (2.6), new ops/custom op support (2.7), RNN improvements (2.8), distributed training (2.9).  Make sure you update your tensorflow packages to get the latest features and improvements.

Leverage the unique benefits of the apple silicon architecture.  Shows speedups with various batch sizes.  Performance improves with bigger batch sizes, because each gradient update corresponds lcosely to the true gradient.  Allows you to run larger networks or batch sizes.  Run your workload on a single mac studio.

High performance per watt, meaning your network srun more efficiently than ever.

## new ops and custom ops
GPU acceleration for a variety of new operations. a rgMIn, all, pack, adaDelta, etc.  But what if you want gpu accel for a new op?

To do this, you will need to createa  custom operation.  

1.  convolution
2. maxpool
3. cross entropy

These are GPU Accelerated through MPS graph.  But maybe you want a custom loss function.  Without MPS GPU acceleration, that work will need to be performed on the CPU timeline, which introduces synchronization overhead and stalls.

Do this custom loss on the GPU.  To implement, must understand TensorFlow Metal Stream protocol.

```objc
@protocol TF_MetalStream

- (id <MTLCommandBuffer>)currentCommandBuffer;
- (dispatch_queue_t)queue;
- (void)commit;
- (void)commitAndWait;

@end
```

Exposes the queue to use for CPU-side synchronization.  Use comimt or commitAndWait to submit work to GPU (comimtAndWait for debug purposes).

## Custom operations.
1.  Register the op
2. Implement
3. Import into your training scripts

### Registering the operation
```cpp
// Register the operation
REGISTER_OP("ZeroOut")
    .Input("to_zero: int32")
    .Output("zeroed: int32")
    .SetShapeFn([](::tensorflow::shape_inference::InferenceContext* c) {
      c —> set_output(0, c —> input(0));
      return Status::OK();
    });
```

### Implement

```objc
// Define Compute function
void MetalZeroOut::Compute(TF_OpKernelContext *ctx) {
     // Get input and allocate outputs
     TF_Tensor* input = nullptr;
     TF_GetInput(ctx, 0, &input, status);
     TF_Tensor* output;
     OP_REQUIRES_OK(ctx, ctx->allocate_output(0, input.shape(), &output));

    // Use TF_MetalStream to encode the custom op, ensures submissions are serialized.
    id<TF_MetalStream> metalStream = (id<TF_MetalStream>)(TF_GetStream(ctx, status));
    dispatch_sync(metalStream.queue, ^() {
              id<MTLCommandBuffer> commandBuffer = metalStream.currentCommandBuffer;
              // Create encoder and encode GPU kernel
             [metalStream commit];
    }

    // Delete the TF_Tensors
    TF_DeleteTensor(input);
    TF_DeleteTensor(output);
}
```

Get tensor object, define output, which requires an allocation.

### Import
```python
# Import operation in python script for training
import tensorflow as tf
zero_out_module = tf.load_op_library('./zero_out.so')
print(zero_out_module.zero_out([[1, 2], [3, 4]]).numpy())
```

Refer to metal developer resources for info on how to build and import `.so` files.  This example imports by using tF load_op_library.  

Now this works like a pythonw rapper, and we can invoke from python.

## Neural Radiance Fields, nerf

Enabling GPU accel.

Synthesize 3d views of a model.  For traaining, nerf takes as input images of an object vfrom different angles.  Two stacked multi-layer perceptrons.  Output is a volumetric model of the object.

A key perf optimization uses a hashtable optimization.  Allows a smaller multi-layer perceptron.  TF does not support this natively, so we implement in a custom op.  GPU acceleration makes it possible to train NeRF much faster.

### Demo
takes 30 minutes to see if anything was seen.  So I'll show you ap rebaked version.  3d model is blurred and unclear, even after 30m.  Will require a much longer training time.

Original twostacked multilayer perceptron is too slow.

Now we kick off an optimized version with custom hashtables.  Already able to render a much clearer model, each epoch takes 10 seconds to learn.  For more information, check out sample code.

Just one of the many networks that shows how to implement custom ops.

## Distributed  training
You can run multiple instances of the training script in separate processes, where each process evaluates a single iteration of the model.  Each process reads data from a central store, and then run through the model and calculaet gradients.  Processes will average gradients and communicate to each other.  Model is updated and you can repeat this process until iterations are exhausted.

Horovod - ring-node approach.  Each node communicates with 2 peers multiple times.  Worker processes synchronize gradients before each iteration.

For a single amc studio, we have 200 images per sec.  2, doubles.  4, 800.  Almost linear scaling on compute-bound training workloads.

Leverage GPUs on multiple devices to speed up your training time and make the most of apple devices.  All the improvements and features culminate into this chart showing relative performance against CPU implementation, with more improvements to come in the future.

# What's new in MPSGraph
MPSGraph uses primitives from MPS to accelerate work on GPU.  Today, I'm going to talk about two features to use.

## Shared events
Running apps on same command queue ensures synchronization between workloads.  Here wwe guarantee to terminate before other workloads, such as post processing and display.  You leverage the parallelism *within each dispatch*.

Some apps benefit from more parallelism.  Could be achieved by submitting work on multiple command queues.  In this case, post processing pipeline may be dispatched before the compute has produced results, introducing a race.

Shared event api can solve this problem and introduce synchronization across queues.  Using shared events within your code is simple.

Let's use two queues, for compute and postprocess.  Assume the reuslt of the compute graph is used as inputs for post process graph.

New MPS graph track in metal system intrace indicates that command queues overlap, producing a data race.  Solve this with a shared event.

```swift
// Using shared events
let executionDescriptor = MPSGraphExecutionDescriptor()
let event = MTLCreateSystemDefaultDevice()!.makeSharedEvent()!
executionDescriptor.signal(event, atExecutionEvent: .completed, value: 1)

let fetch = computeGraph.runAsync(with: commandQueue1,
                                  feeds: [input0Tensor: input0),
                                          input1Tensor: input1],
                                  targetTensors: [finalTensor],
                                  targetOperations: nil,
                                  executionDescriptor: executionDescriptor)

let executionDescriptor2 = MPSGraphExecutionDescriptor()
executionDescriptor2.wait(for: event, value: 1)

let fetch2 = postProcessGraph.runAsync(with: commandQueue2,
                                       feeds: [input0Tensor: fetch[finalTensor]!,
                                               input1Tensor: input1],
                                       targetTensors: [finalTensor],
                                       targetOperations: nil,
                                       executionDescriptor: executionDescriptor2)
```


## New operations
Do even more with the framework.  

## RNNs
* recurrent neural network (RNN)
* Long-short term memory (LSTM)
* gated recurrent unit (GRU)

These ops all work similarly.  

LSTM operations for natural alnguage processing and other application.  Diagram shows how LSTM operation works.  [[Using Metal 2 for Compute]]

Implement the LSTM unit yourself, but to do so you'd have to build this custom subgraph.  Now use LSTM operation, which efficiently encodes all GPU work required by the recurrent unit.

Up to 8x faster

```swift
let descriptor = MPSGraphLSTMDescriptor()

descriptor.inputGateActivation = .sigmoid
descriptor.forgetGateActivation = .sigmoid
descriptor.cellGateActivation = .tanh
descriptor.outputGateActivation = .sigmoid
descriptor.activation = .tanh
descriptor.bidirectional = false
descriptor.training = true

let lstm = graph.LSTM(inputTensor,
                      recurrentWeight: recurrentWeightsTensor,
                      inputWeight: weightsTensor,
                      bias: nil,
                      initState: nil,
                      initCell: nil,
                      descriptor: descriptor,
                      name: nil)
```

Other RNN ops work similarly.  Try these ops aout and see what kind of speedups you can get.

## Max pooling with indices

An input tensor and window size.  For each application of the window, we compute the maximum value within that window.  Commonly used in computer vision to reduce the dimensionality of an image.  API extended to return indices of maximum value location extracted by a pooling operator.

Can use (reuse?) indices in the gradient pass where gradients must be propagated to locations where max value was extracted.

* Return indices (consistent with PyTorch)
* Supports and optimizes training
* Up to 6x faster



```swift
// Forward pass
let descriptor = MPSGraphPooling4DOpDescriptor(kernelSizes: @[1,1,3,3], 
                                               paddingStyle: .TF_SAME)
descriptor.returnIndicesMode = .globalFlatten4D

let [poolingTensor, indicesTensor] = graph.maxPooling4DReturnIndices(sourceTensor,
                                                                     descriptor: descriptor, 
                                                                     name: nil)

// Backward pass
let outputShape = graph.shapeOf(destination, name: nil)
let gradientTensor = graph.maxPooling4DGradient(gradient: gradientTensor,
                                                indices: indicesTensor, 
                                        outputShape: outputShape, 
                                        descriptor: descriptor, 
                                        name: nil)
```

First, the poollingTensor and second the indicesTensor.  you can cache the indices tensor for later use, ex on a training pipeline.

## Random
New RNG.  To initialize the weights of a training graph.
Uses the philox 4x32-10 algorithm
same results as tensorflow for given seed

REturns a random tensor.  Can be used as input.  To use,

```swift
// Declare Philox state tensor
let stateTensor = graph.randomPhiloxStateTensor(seed: 2022, name: nil)

// Declare RandomOp descriptor
let descriptor = MPSGraphRandomOpDescriptor(distribution: .truncatedNormal,
                                            dataType: .float32)
descriptor.mean = -1.0f
descriptor.standardDeviation = 2.5f
descriptor.min = descriptor.mean - 2 * descriptor.standardDeviation
descriptor.max = descriptor.mean + 2 * descriptor.standardDeviation

let [randomTensor, stateTensor] = graph.randomTensor(shapeTensor: shapeTensor
                                             descriptor: descriptor, 
                                             stateTensor: stateTensor, 
                                             name: nil)
```

The descriptor specifies a truncated normal distribution.  Can also use normal and uniform distributions.  Specify distribution standard characteristics such as mean, etc.

MPSGraph supports GPU accelerated operations to comptue the hamming distance between two bit vectors.  Number of distance between two sequences.  used on several applications.

```swift
// Code example remember 2D input tensor
let primaryTensor = graph.placeholder(shape: @[3,4], 
                                      dataType: .uint32, 
                                      name: nil)
let secondaryTensor = graph.placeholder(shape: @[1,4], 
                                        dataType: .uint32, 
                                        name: nil)

// The hamming distance shape will be 3x1
let distance = graph.HammingDistance(primary: primaryTensor,
                                     secondary: secondaryTensor,
                                     resultDataType: .uint16
                                     name: nil)
```

New kernel supports broadcasting over batch dimensions on the GPU.

## ExpandDims

ex, two-to-three dimensions.

```swift
// Expand the input tensor dimensions, 4x2 -> 4x1x2
let expandedTensor = graph.expandDims(inputTensor, 
                                      axis: 1, 
                                      name: nil)
```
or squeeze back

```swift
// Squeeze the input tensor dimensions, 4x1x2 -> 4x2
let squeezedTensor = graph.squeeze(expandedTensor, 
                                   axis: 1, 
                                   name: nil)
```

```swift
// Split the tensor in two, 4x2 -> (4x1, 4x1)
let [split1, split2] = graph.split(squeezedTensor, 
                                   numSplits: 2, 
                                   axis: 0, 
                                   name: nil)
```

```swift
// Stack the tensor back together, (4x1, 4x1) -> 2x4x1
let stackedTensor = graph.stack([split1, split2], 
                                axis: 0,
                                name: nil)
```

Generate coordinate values along tensor dimensions for a given shape.  ex, populate a tensor of shape 2x4 with corodinates along the 0 axis.  Can be used to implement a range1d operation.  ex, assume you want to generate the range of numbers 3-27 with increments of 4.
```swift
// Get coordinates along 0-axis, 2x4
let coord = graph.coordinateAlongAxis(axis: 0, 
                                      shape: @[2, 4], 
                                      name: nil)
```

```swift
// 1. Set coordTensor = [0,1,2,3,4,5] along 0 axis
let coordTensor   = graph.coordinate(alongAxis: 0, withShape: @[6], name: nil)

// 2. Multiply by a stride 4 and add an offset 3
let strideTensor  = graph.constant(4.0, dataType: .int32)
let offsetTensor  = graph.constant(3.0, dataType: .int32)
let stridedTensor = graph.multiplication(strideTensor, coordTensor, name: nil)
let rangeTensor   = graph.addition(offsetTensor, stridedTensor, name: nil)

// 3. Compute the result = [3, 7, 11, 15, 19, 23]
let fetch = graph.runAsync(feeds: [:],
                           targetTensors: [rangeTensor],
                           targetOperations: nil)
```

With all these neww  operations, do even more and get higher performance across apple ecosystem.  

## Performance improvements demo
magic mask, uses ML to identify a moving object on screen and selectively apply filters on top of it.

Framerate demo by adopting MPS graph.  Explore whta kind of performance to bring to your app.

# Wrap up
* PyTorch and TensorFlow acceleration
* Training on apple silicon
* Optimize computation with MPS graph.

* https://developer.apple.com/documentation/metal
