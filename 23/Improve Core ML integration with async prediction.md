Learn how to speed up machine learning features in your app with the latest Core ML execution engine improvements and find out how aggressive asset caching can help with inference and faster model loads. We'll show you some of the latest options for async prediction and discuss considerations for balancing performance with overall memory usage to help you create a highly responsive app. Discover APIs to help you understand and maximize hardware utilization for your models. For more on optimizing Core ML model usage, check out "Use Core ML Tools for machine learning model compression" from WWDC23.

Engineer on the core ml team.  I'm going to integrate coreml into your app.  Building experiences into your app has never been easier.

Set of domain-specific frameworks give you access to built-in intelligence through simple apis.
trained and optimized by apple.  These models are executed via coreML.  Provides framework for running models on device.  Easily deploy models customized for your app.  Extracts away the hardware details while leveraging high-performance compute capabilities.

Help you integrate machine learning models into your app.

Our focus was performance and flexibility.  Improvements to workflow, api surface, and inference engine.

Performance benefits.

Relative prediction time between iOS 16 and 17, 17 is just faster.

speedup comes with the os and doesnt' require recompilation or any code changes.  Same is true for other platforms as well.  Speedup is model/hardware dependent.

# Workflow overview

## Developing your model
Use CreateML.  Provides various templates for common ML tasks, can leverage things built into the OS.  Interactively evaluate results.  To learn more, see the [[Discover machine learning enhancements in Create ML]]

can also use python.  

Input and output types.
Optimize converted model's precision, footprint, computation costs.  Input/output that best match app's dataflow.  If your input shape can vary, specify variation rather than just choosing one shape or switching among multiple specific models.

Compute precision can be explicitly set between one model and whole model operations.
Weight quantization and compression

Utilities can help you improve footprint of your model and improve performance on device.  Some tradeoff in accuracy.  There are some new tools to help

* post training quantization utilities
* Quantization aware training
* weight palettization and pruning
* Activation quantization

[[Use Core ML Tools for machine learning model compression]]

## evaluation
Python with CoreML tools?
xcode provides helpful tools when it comes to evaluation/exploration of your models.
Previews are available for many common model types.  Sample inputs, preview predicted output.
Performance reports -> breakdown of model computation performance for load, prediction, and compilation times on attached device.  Even before training!


## using your model
Model integration.  Carefully manage/optimize how you use the model.

1.  Write application code.  
2. Build code and model
3. Run, test, profile
# Compute availability

New additions for optimizing model integration.

CoreML is supported on all apple platforms.  By default we consider all compute.  CPU, GPU, neural engine.  However performance may vary.

Some experiences may require models running on neural engine to meet performance requirements.  New API for runtime inspection of availability.

`MTLComputeDevice` enum.  

Inspect what devices are available on coreML.  For example, this code checks if a neural engine is available.  Checks if collection of all available compute devices contains one that's neuralengine.

# Model lifecycle

Two asset types.  Source models, compiled models.  Source models have a `mlmodel` or `mlmpackage` format.  

Compiled model has `mlmodelc`.  Runtime access.  In most cases, you add a source model to app target, then we compile and place in app's resources.

Instantiation takes a URL to its compiled form and optional configuration.  Loaded all necessary resources based on specified configuration and device-specific hardware capabilities

First, coreml checks the cache to see if we've already specialized model based on configuration and device.  If so, we load the required resources.

If configuration is not found in cache, we trigger device specialized compilation.  Process adds output to cache and finishes load from there.  Uncached load.

for some models, uncache dload can take a significant amount of time.  Focused on optimizing model for device and making subsequent loads as fast as possible.  

1.  Model parsing
2. general optimization passes
3. segmentation -> cached
4. Segment optimize and compilation.  -> also cached
Cache policy: per bundle, model, and configuration
Persists assets across
* model reload
* app relaunch
* reboots
Invalidates and purges asset when
* storage pressure
* system update
* model modification or deletion

Model load visibility in the core ml instrument

If we have "prepare and cache" in coreml instrument, it was an uncached load.

If load event has the label cached, it was a cached load.  New specifically for MLProgram models.

Performance report also has visibility.  By default, it shows the median cached load.  But you can also specify uncached load times. Since loading a model can be expensive in terms of latency and memory

## best practices

avoid blocking main thread
async loading
Keep loaded for multiple predictions
Unload when no longer needed
Subsequent loads will be faster.
# Async prediction
New async options!

Since synchronous prediction isn't threadspace, we have to ensure predictions are run serially.  We ensure this by making ColorizingService an actor.  

When looking at the instruments trace, we see that predictions are run serially.  Note that actor isolation is not only around inference but also preparation.  

To take advantage of coreML prediction concurrency itself, consider batch processing API.  Batch version is pretty straightforward, however the challenging part is forming the batch.  Multiple aspects of this usecase that make it difficult.

best used
* fixed quantity of work (but we're a function of screen size, scrolling)
* partial batches
* Different UI experience (colorized in batches)
* Lack of cancellation support

New async prediction API is useful.  Threadsafe and works well for coreml alongside swift concurrency.  To switch to async design, I change colorized method to async.  Add await keyword, required to use the new async version of the api.  Change colorizing service to be a class instead of an actor.

Lastly, I add a cancellation check to start of method.  Best to include an extra check at the start, so we also avoid preparing the inputs.

I'll make these changes and re-run the app.  Just as before, I'll set to colorized.  

Trace now shows multiple things running concurrently.  

With initial implementation, colorizing images took about 2 seconds.  After switching to async implementation, we cut time in half.  Speed up 2x.

However, it's important to note that the amount that a given model and usecase heavily depends on several factor

* model operations
* compute units and hardware
* system workload
* ML program and pipeline model types

Overall, when adding concurrency to your app, carefully profile the workload to make sure it's actually benefiting your usecase.

Keep in mind, concurrent memory can be an issue.  Use allocations.  Trace is showing that memory usage is rising quickly for many overlapping inputs.  A potential issue is that the colorize method has no flow control, so the amount of images concurrently has no fixed limit.  May not be an issue, but having many sets of IO can increase peak memory usage.

A way to improve this is to limit 2 in-flight, queue later.  

Best strategy depends on your usecase.  ex, when streaming data from a camera, you may simply want to drop work instead of dferring it.  Avoid accumulating frames that are no longer temporally relevant.

1.  synchronous context, and time betwee is large, use sync prediction.
2. inputs in batches, batch API
3. async context, and have large amounts of inputs individually available over time, use async api

# wrap up

* Optimization steps in your workflow
* Compute availability api
* Cached vs uncached model loads
* Integrating async prediction
* 



# Resources
* https://developer.apple.com/documentation/coreml
