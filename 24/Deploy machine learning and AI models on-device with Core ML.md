Learn new ways to optimize speed and memory performance when you convert and run machine learning and AI models through Core ML. We'll cover new options for model representations, performance insights, execution, and model stitching which can be used together to create compelling and private on-device experiences.

Create new forms of interaction, powerful professional tools, insightful analysis.  Keep data private and secure.

Thousands of apps create amazing experiences.  So can you!

# Integration
Focus on integrating/running on device.

[[Train your machine learning and AI models on Apple GPUs]]
[[Bring your machine learning and AI models to Apple silicon]]

Models are executing with apple silicon's stack.  Dispatching work across CPU/GPU/neural engine.  MPSGraph, BNNS Graph.  

[[24/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]
[[Support real-time ML inference on the CPU]]

iOS 18 is much faster, etc.  Doesn't require recompiling your models.  

# MLTensor

Can be as simple as passing in the required input.  Complexity can grow for more advanced ucasecases.  Generate AI is often iterative and can load multiple models.  In these usecases, there exist computation outside the model.

Implementing operations from scratch or various low-level APIs.  

MLTensor.  A new type in CoreML that provides convenient efficient way.

* compute for multidimensional arrays
* Optimized for Apple silicon
* Familiar API

Elementwise-addition, multiplication, etc.  

Slice tensor by indexing into each dimension.  

top-k sampling, most likely token, etc.

# Models with state

Most models you've interacted with are stateless.  Process each input independently with no history, ex image classifier is stateless as each input is processed in isolation.  In contrast with stateful models, which retain history from previous inputs.  

Stateful models can be supported by managing state manually.  However, loading/unloading the data used for state incurs some overhead.  Becomes noticeable as size of state increases.

This year we improved our support for stateful models, coreml does it for you reducing some overhead.

Key-value cache as I/O.  Score indicates confidence of word coming next.  Model outputs k/v vectors for given word.  Used by the attention mechanism to improve ability to generate outputs.  To avoid computing prevous word vectors, often stored/reused.  KV cache.  This cache sounds like an ideal candidate for our new feature.

Support for states must be explicitly added during prep [[Bring your machine learning and AI models to Apple silicon]]

States appear in xcode when viewing the ML package.  

Instead of manually preallocating each state, we preallocate at the top of the loop.  Can simply pass in via model instance.  Since update is performed in-place, omit last step.

1.6x speedup on mistral 7B, varies by hardware, etc.

# Multifunction models

When we think of an ML model, we usually think of input->output, like a function.  Functions are how NN are represented.  Single function containing our block of operations.  An extension is to support multiple functions, now supported in CoreML.

Adapters: small modules embedded into network.  Extend functionality of large pretrained model without adjusting weights.  Single base model shared across multiple adapters.

[[Bring your machine learning and AI models to Apple silicon]]

Many other scenarios!

# Performance tools

Can be generated with any connected device.  Simply open the model in xcode, select the performance tab, click the plus button, select the device to profile, and compute unit.

The report displays a summary of load, prediction times, compute unit usage.  This year we now offer even more information, such as estimated time, etc.  Time spent on each operation.  Calculated using median prediction time mulitplied by estimated relative cost.  Sort ops based on estimated time.

Can find out why layers are running on each device.  

MLComputePlan -> model debugging and profiling info for coreML.  
* model structure
* Supported and preferred compute device
* Operation compute device support status
* estimated relative cost.
# Wrap up
* MLTensor
* Stateful predictions
* Multifunction models
* Enhanced performance tools



# Resources
* https://developer.apple.com/documentation/CoreML
* https://machinelearning.apple.com/research/stable-diffusion-coreml-apple-silicon
