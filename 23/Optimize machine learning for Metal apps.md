Discover the latest enhancements to accelerated ML training in Metal. Find out about updates to PyTorch and TensorFlow, and learn about Metal acceleration for JAX. We'll show you how MPS Graph can support faster ML inference when you use both the GPU and Apple Neural Engine, and share how the same API can rapidly integrate your Core ML and ONNX models. For more information on using Metal for machine learning, check out “Accelerate machine learning with Metal” from WWDC22.

GPU-accelerate dprimitives
General-purpose compute graph
Extends MPS with multi-dimensional vectors

Machine learning inference frameworsk

TesnorFlow
PyTorch

[[Build customized ML models with the Metal Performance Shaders Graph]]
[[Accelerate machine learning with Metal]]

# PyTorch and TensorFlow updates

what's new in PyTorch 2.0

Operator coverage
* grid sampler
* triangular solve
* topk
* many more!

test modules
Network coverage
whisperAI
YOLO, stable diffusion, etc.

nightly.
profiling support using OS signposts
operation executions
copeis between cpu/gpu
fallbacks to cpu

Metal System Trace, part of Instruments

[[Accelerate machine learning with Metal]]

discussion of how to accelerate custom ops.
1.  Implement in c++
2. bind to python
3. compile extension
4. import

mixed precision.
Train faster using less memory
same quality!

This seems to work layer-by-layer, detecting the right precision to use for that layer.

TensorFlow Metal plugin 1.0
* grappler pass optimizations
	* FusedConv/FFusedMatMul
	* optimizer operations
	* LSTMCell/GRUCell
* mixed precision
* simplified installation



# JAX acceleration

based on numpy.

auto differentiation (grad)
auto batching (vmap)
jit compilation (jit)


# ML inference with MPSGraph

* serialization
* model format conversion
* quantization

[[Build customized ML models with the Metal Performance Shaders Graph]]

initial compilation can lead to high launch times.  Now we have MPSGraphPackage for this.

Create MPSGraphExecutable ahead of time.  

Model format conversion with MPSGraphTool.
Cross-platform support for deploying existing models in MPSGraph.
Can also use onnx.

Quickly include your existing models to mpsgraph.

```bash
mpsgraphtool convert -coreml model.mlpackage -path ./ -packageName etc etc
```

## quantization

May be better to use reduced-precision or 8-bit integer numbers.
For 8-bit integer formats, there are two types of quantization
* symmetric
* asymmetric - biased

also
* complex types
* FFTs
* conv3d, etc.


# Resources
* https://developer.apple.com/documentation/metal/metal_sample_code_library/customizing_a_pytorch_operation
* https://developer.apple.com/documentation/metalperformanceshadersgraph/filtering_images_with_mpsgraph_fft_operations
* 