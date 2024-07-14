Get started with an overview of machine learning frameworks on Apple platforms. Whether you're implementing your first ML model, or an ML expert, we'll offer guidance to help you select the right framework for your app's needs.

Underlying many features in our os/apps are advanced ML/AI models.

gesture recognition, portrait mode, ECG/heart rate, etc.
Models run entirely on device!  Doing so allows them to be interactive and advances privacy.

Possible due to apple silicon.  Unified memory combined iwth ML acclerators in cpu, gpu, neural engine, allows for efficient and low-latency.

# Apple intelligence

Exciting advancements in intelligence.  Many of these will be available within your apps such as writing tools.  Helps users communicate even more effectively by rewriting text for tone/clarity.

Integrates system text and webviews you're already using.

[[Get started with Writing Tools]]

* proofreading and summarization
* processed on device
## image playground

Integrate image creation features into your apps.  Don't train a model or design safety guardrails.

Prebuilt UI for users to create and embed images.  Plus, since the model runs locally on device, users can create as many images as they want.

## siri

[[Bring your app to Siri]]

# ML-powered APIs

If you want to offer your own intelligent features, we have a number of APIs/frameworks to helpw ith that.  Dont' deal with model statically.

## vision

Vision framework provides a range of capabilities for visual intelligence. Text extraciton, face detection, body pose, and more.

Swift 6 support.
Body pose with hands
aesthetic score

[[Discover Swift enhancements in the Vision framework]]

## Natural language
[[Explore natural language multilingual models]]

## speach
[[Customize on-device speech recognition]]
## sound
[[Discover built-in sound classification in SoundAnalysis]]

## translation

Direct language-to-language translation with simple translation presentation Ui which can be launched programmatically.

* simple UI
* flexible API
* efficient batching
[[Meet the Translation API]]

Apple's ML-powered apis offer tons of capabilities.  When you need some model customization for particular usecase, CreateML is a great tool to begin with.

Customize models powering our frameworks with your own data.  

The underlying createML and createML components framework offer you capabilities to train on all platforms.

New this year, we have an object tracking template.  Train reference objects to anchor spatial experiences on visionOS.  
Data source exploration improvements
Time series models

[[What’s new in Create ML]]

# Running models on device

You can run a wide-array of models across our devices.  Whisper, SD, Mistral, etc.

1.  Train
2. Prepare (convert).  Optimize parameters, etc.
3. Integrate (write code).

## train

Take full advantage of apple silicon and unified memory architecture to architect/train high performance models.  PyTorch, TensorFlow, JAX, MLX.

[[Train your machine learning and AI models on Apple GPUs]]

* scaled dot product attention
* Custom metal operations
* Mixed precision JAX

## prepare

Core ML tools.  Start with any pytorch model, use tools and convert it into coreml format.  also optimize the model for apple hardware using a number of compression techniques.  

New model compression techniques.  
States and transformers
Multifunction models

[[Bring your machine learning and AI models to Apple silicon]]

## integrate

CoreML.  

[[Deploy machine learning and AI models on-device with Core ML]].  New MLTensor type.  Key-value caches.  Adapters.   Performance reports.

If your app has demanding graphics workloads, MPS graph lets you sequence with other workloads, optimizing GPU utilization.  

alternatively, BNNS Graph, Accelerate offers strict memory management, etc.

Form part of CoreML's foundation.  And are directly accessible for you.

* MPSGraph.  Built on top of MPS.  [[24/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]  Fast fourier transforms, MPS graph model viewer.
* BNNS Graph.  New API from accelerate for CPU.  Graph-based API.  Real-time guarantees.  Audio processing.  [[Support real-time ML inference on the CPU]]

Providing you the best ML foundation.  built into the OS and developer SDK.  
# Research

Hundreds of papers with novel approaches to AI models and device optimization.  We have made sample code, data source, MLX etc. open source.

## MLX
* familiar and flexible API
* Designed for apple silicon
* Multi-language support
https://github.com/ml-explore/mlx

## CoreNet

OpenELM.  Efficient language model family with open training and inference framework.  

# Wrap up
* apple intelligence
* Customize with ML-powered APIs
* Train models on apple gpus
* Optimize models for apple silicon
* Run models on device
* Explore open source frameworks and models


