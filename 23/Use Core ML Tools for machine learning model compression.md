Discover how to reduce the footprint of machine learning models in your app with Core ML Tools. Learn how to use techniques like palettization, pruning, and quantization to dramatically reduce model size while still achieving great accuracy. Explore comparisons between compression during the training stages and on fully trained models, and learn how compressed models can run even faster when your app takes full advantage of the Apple Neural Engine. For more on optimizing Core ML, check out “Improve Core ML integration with async prediction" from WWDC23.

Number of models per app is increasing.  Models themselves get bigger.  Critical to keep model size in check.

# Benefits

* more models
* Larger models
* Faster inference

# why are models large
50 layers in total with various sizes.
ending linear layer with 2.1M parameters.
2.5M parameters x 2 bytes per weight = 50mb

# Paths to smaller models
* efficient architecture
* compression techniques

# Compression techniques
* pack weights more efficiently, with sparse matrix representation, pruning
* Use fewer bits per weight (quantization, palletization)
	* Lossy compression

## pruning
setting some weight values to 0.  To prune we can set smallest magnitude weights to 0.  Now only need to store non-zero values.  and an index of 0s.

Model size decreases linearly with sparsity.  50% -> 28MB, approximately half size

## quantization

Store weights in 8-bit precision.  Take weight values, scale/shift/round them to INT8 range.

ex 2.35 * int8 + bias

## palletization
Non-uniform lowering of precision via clustering

| precision          | 2bit | 4bit | 6bit | 8bit |
| ------------------ | ---- | ---- | ---- | ---- |
| number of clusters | 4    | 16   | 64   | 256  |

Basically we average values that are 'close' and then we store the raw values in a table, and then an index for the overall matrix.

## summary

| x                                | pruning                        | quantization                      | palletization                |
| -------------------------------- | ------------------------------ | --------------------------------- | ---------------------------- |
| representation                   | non-zero values + zero indices | Int8 weights + float scale + bias | 2,4,6,8 bit per weight value |
| compression factor (wrt float16) | 1x-10x                         | 2x                                | 2x,2.67x, 4x,8x              |
| parameter                        | % sparcity (S)                 | scale only / scale + bias         | number of clusters           |

# Workflows

As you apply compression, model size reduces but this impacts accuracy.  Impact may become more prominent.

Maybe apply compression during training?  In this wya the model can react to constraints we enforce.

Training-time compression improves tradeoff between accuracy and compression.  

To apply during training, we need to use differentiable operators.

# Core ML tools

| x             | post-training compression     | training time compression |
| ------------- | ----------------------------- | ------------------------- |
| sub-module    | coremltools.compression_utils | n/a                       |
| pruning       | .sparsify_weights             | n/a                       |
| quantization  | .affine_quantize_weights      | n/a                       |
| palletization | .palettize_weights            | n/a                       |

`import coremltools.optimize` 

new:


| x             | post-training compression   | training time compression     |
| ------------- | --------------------------- | ----------------------------- |
| sub-module    | coremltools.optimize.coreml | coremltools.optimize.torch    |
| pruning       | .sparsify_weights           | .pruning.MagnitudePruner      |
| quantization  | .affine_quantize_weights    | .quantization.LinearQuantizer |
| palletization | .palettize_weights          | .parlletization.DKMPalettizer |


# Performance impact

Support and execution of compressed models.

| iOS                           | 16              | 17                  |
| ----------------------------- | --------------- | ------------------- |
| sparse weights                | yes             | yes                 |
| quantized weights             | yes             | yes                 |
| palettized weights            | yes             | yes                 |
| 8-bit activation quantization | no              | yes                 |
| inference latency             | same as float16 | faster than float16 |
see also macOS, tvOS, watchOS, etc.

In models where only weights are comperssed, since activations are in fp precision, before an operation such as a convolution or mmul can happen, weight values must be decompressed.  This decompression takes place AOT in the iOS 16 runtime, ex we convert to fully float prior to execution.

However in iOS 17, in certain scenarios, weights are decompressed JIT, advantage of loading smaller bit weights from the memory at the cost of doing decompression in every instance call.  For certain compute units such as neural engine, and certain models, this could lead to inference gains.

typically, 1.1x or so.  5-30%.  

what's the strategy for best latency performance?

1.  Start with float model, use optimized apis to explore various representations.
2. Then profile latency on device of interest.  CoreML performance reports will give you a lot of visibility into inference.
3. shortlist based on best gains
4. Evaluate accuracy and try to improve, ex. training time compression.

# Wrap up
* consider reducing size of your models
* Easier than before with new CoreML tools APIs
* Checkout documentation

[[Improve Core ML integration with async prediction]]

# Resources
* https://developer.apple.com/documentation/coreml
* https://coremltools.readme.io/docs
* https://coremltools.readme.io/docs/pytorch-conversion
