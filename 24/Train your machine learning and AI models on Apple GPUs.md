

Learn how to train your models on Apple Silicon with Metal for PyTorch, JAX and TensorFlow. Take advantage of new attention operations and quantization support for improved transformer model performance on your devices.

Lots of amazing features.  GPU.  Combined with unified memory architecture.  
Running large models locally
Larger batch sizes
More quickly from training to deployment

* Train
* Prepare
* Integrate

[[Explore machine learning on Apple platforms]]

* TensorFlow
* PyTorch
* JAX
* MLX

Distributed training
Mixed precision

Enabling metal backend is easier than ever.  Just install TF using a package manager like pip and import to your project

[[22/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]

Pytorch
* custom operation
* Profiler

[[22/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]

JAX 
* JIT compilation
* Numpy-like interface
install jax-metal.  

[[Optimize machine learning for Metal apps]]

MLX
* designed and optimized for apple silicon
* numpy-like API, JIT compilation, unified memory, etc.
* Python, Swift, and C/C++ bindings
* Samples for transformer models, fine-tuning, image generation, audio transcription, and more

`import mlx.core as mx`

ml-explore.github.io/mlx
github.com/ml-explore/mlx

# PyTorch improvements

last year, we advanced to beta
since then we had profiling support, opcoverage, unified memory architecture, etc.

Pytorch MPS backend.  

Stable diffusion, llama, gemma 7b, etc.

8 and 4-bit integer quantization.
scaled dot product attention fused kernel
Unified memory improvements

## quantization

* smallermemory footprint
* improved throughput
* similar accuracy
scaled dot product attention

query, key, value tensors.  
multiplication, scaling, softmax.

Overhead from dispatching small computations to gpu can be avoided.

## fma

## unified memory

Tensor to exist in main memory and be accessible to both cpu/gpu
both cpu and gpu access the same memory.

[[Deploy machine learning and AI models on-device with Core ML]]

## executorch
deploy pytorch models to devices
MPS backend for acceleration



# JAX features
Introduced JAX-metal plugin at wwdc23.
plugin has continued to evolve and add more functionality and perf updates.

Improved advancing array indexing
enhanced contorl flow
fixes and plugin CI runner adoption
BLofat16 mixed precisision

Adoption of the JAX metal backend.
MuJoCo - advanced physics simulation
Open source physics engine for robotics, biomechanics, graphics
AXLearn - deep learning library
Built on top of jax/xla to support development of deep learning
https://github.com/apple/axlearn

## mixed precision
bloat16 is now supported.

represents a wide dynamic range of fp values.



## NDArray indexing and updates





## improved padding

Add padding using dilation

# Wrap up
* ML benefits from unified memory architecture
* Apple GPU acceleration available for Pytorch, jax, tensorflow, mlx
* Many performance improvements especially for transformer models



# Resources
* [Forum: Machine Learning and AI](https://developer.apple.com/forums/topics/machine-learning-and-ai?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10160/4/1EF78DEB-091E-49EE-93FE-D764F58D45C2/downloads/wwdc2024-10160_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10160/4/1EF78DEB-091E-49EE-93FE-D764F58D45C2/downloads/wwdc2024-10160_sd.mp4?dl=1)

