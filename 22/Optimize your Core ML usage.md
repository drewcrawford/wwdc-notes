# Performance tools
Standard workflow using coreml.  
1.  choose your model.  see   [[Get Models on Device Using Core ML Converters]] [[What's new in Create ML]]
2. Integrate into your app.  Bundle the model with your app, using coreml apis, etc.
3. Optimize the way you use coreml.

## Choosing a model
How do you decide between candidates of a single model?  Etc.
* functionality. 
* accuracy
* performance

Open it in xcode.  Model type, size, OS requirements.  Details captured in the model's metadata, comptue/storage precision, etc.

Preview tabs lets you test the model.  Predictions tab displays the model's inputs and outputs, as well as types and sizes to expect at runtime.  Utilites tab can help with omdeling encryption and deployment tasks.  Overall, these views give you a quick overview of your model's functionality and preview of its accuracy.  but what about performance?

* Load time
* prediction latency
* Hardware utilization

One way to get insight into the model's performance is to do an initial integration or prototype.  Since performance is hardware-dependent, you want to test on a variety of supported hardware.

## Performance reports
Xcode model viewer is open.  Now a performance tab.  Select the plus icon at the bottom left, select the device to run on, click next, select which compute units to use, etc.

Optimize for latency with all available compute units.  To ensure the test can run, make sure the selected device is unlocked.  Spinning icon while report is generated.  Model sent to device, several iterations of compile, load, and predictions are run with the model, once those are ocmplete, metrics are calculated.

Details about device where test was run, etc.  Statistics about the run.  Median prediction time was 22.19ms, median load time was 400ms.  also, if you plan to compile your model on device, this shows the comiplation time.  

prediction time tells me that this model can support about 45fps.  

Since the model contains a neural network, there's a layer view shows me where each item runs.  Filled-in checkmark, means it was executed there.  Unfilled checkmark means it was supported there, but coreml didn't choose to run it there.  Empty diamond => not supported.

* available in xcode 14
* Test multiple hardware and OS variants
* No code required

## Integration
Bundling the model with your app, making use of coreml apis to load model and make predictions.  In tihs case, I've built an app that uses CoreML style transfer models to perform style transfer on live camera session.

Framerate is slower than I'd expect, why?


## Optimize
Show the performance a model is capable of achieving.  But you need a way to profile performance for running live. 

**CoreML Instrument**.  Lets you visualize performance of your model live, helps identiy potential performance issues.

Tempalte includes core ml instrument, and several other useful instruments to profile my coreml usage.  To capture trace, I press record.  

Activity => top-level coreml events that ahve a 1:1 relationship with API.
Data => events in which coreml performs data checks or data transformations
Compute => coreml sends compute requests to specific compute units, like neural engine or GPU.  

Ungrouped view => individual lane for each event type.

bottom: model activity aggregation view.  Aggregate statistics for all events.  Can sort by duration.  

Reprofile the app to see if I'm no longer loading before each prediction.

So far I've only looked at views that show all activity.  Here there's one model per style.  So I can break down by model.  In main graph, click arrow at the top left, and it will make one subtrack for each model used in the trace.  Here we see all models.

Aggregation view offers similar functionality by breaking down statistics by model.

Combine coreML instrument with GPU instrument and neutral engine instrument.  I have 3 instruments pinned here.
CoreML => entire region where the model ran.
neural engine instrument => compute first running on the neural engine.  Then GPU instrument shows handed off.  Better idea of how my model is actually being executed.

## Recap
* available in xcode 14
* profile your app's core ml usage
* identify issues and opportunities
* measure impact of your changes
* Combine with GPU and neural engine instruments

# Enhanced APIs
When creating a model, that model has input/output features, each with a type and size.  At runtiem, you use APIs to provide inputs.

## image and multiarrya scalar types

| image                        | multiarray |
| ---------------------------- | ---------- |
| oneComponent8 // Grayscale 8 | Int32      |
| 32BGRA //color               | Double     |
| 32ARGB //color               | Float           |

## Sharpening filter
To do this, I used moremltools to convert an image-sharpening torch model to coreml format as shown here.

Model uses float16 precision.  Takes image inputs and produces image outputs.

I had to downcast input from onecomponent16half=>onecomonenthalf, and then reverse for output.

Since model performs computation ant float16 precision, at some point it converts back to float16.  Notice the data steps coreml is doing before and after computation.  When zooming in on data lane, it shows coreml is copying data to prepare it for computation on the neural engine, which converts it to float16.  This is unfortunate, since it was already in float16.

Ideally, these transformations can be avoided by making the model work directly with float16 inputs and outputs.  Starting in this reelase, we are adding supprot for `OneComponent16Half` and `Float16` arrays.

Just specify new color layout `GRAYSCALE_FLOAT16`.  Since float16 support is available in iOS16/macos ventura, these features are only available when a minimumd eployment target is specified.

How reconverted model looks.  Inputs andoutputs are marked as Grayscale16Half.  My app can directly feed float16 images to coreml, avoiding the needs for downcasting inputs, etc.

Send the pixel buffer directly to coreml.  No data copy or transformation.  

## Output backing
You can have coreml fill yuor preallocated buffer.  Do this by allocating a backing buffer and settig it in prediction options.  Better control over buffer management.

Summary
* native support for float16 buffers
* provide preallocated output buffers
* recommend using IOSurface

# new capabilities
## Weight compression
in iOS 12, we introduced post-training weight compression.  We are extending this to 16 and 8-bit support.  And new option to store weights in sparse representation.  coreml tools can quantize, palettize, and sparisfy weights.

| neural network                  | ml programs                             |
| ------------------------------- | --------------------------------------- |
| last year                       | this year's release                     |
| Linear, palettized/lookup table | Linear, palettized/lookup table, Sparse |

## compute units
Set using `MLMOdelConfiguration.computeUnits` on model init/load

| mlcomputeunits      | cpu | gpu | neural engine |
| ------------------- | --- | --- | ------------- |
| .all                | yes | yes | yes           |
| .cpuAndGPU          | yes | yes | no            |
| .cpuOnly            | yes | no  | no            |
| .cpuAndNeuralEngine | yes | no  | yes              |

In addition to existing options, we have .cpuAndNeuralengine.  This avoids GPU.  Can be helpful when the app uses GPU for other computation, and we want to limit focus to cpu + neural.

## MLModel from data
In-memory model compilation and instantiation
Custom serialization, encryption, and runtime model construction

no requirement for model to be on disk.

## Swift package support
With xcode 14, you can include coreml models in your packages.  When someone imports your package, it will just work. xcode compiles your bundles automatically, etc.  Excited about this change as it will make it easier to distribute your models

# Wrap up
* performance reports and coreML instrument
* float16 support and output backings
* weight compression
* more integration options



* https://developer.apple.com/documentation/coreml