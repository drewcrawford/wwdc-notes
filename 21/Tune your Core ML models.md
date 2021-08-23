#coreml 

# API improvements
`MLMultiArray` Multidimensional data.
Problems: type at runtime, NSNumber, etc.

`MLShapedArray` is a new swift-only type.  Strongly-typed.  COW value types, rich slicing syntax with fast indexing.

Interoperable.  Use the initializer that takes an instance of the other type.  Convert datatypes using the converting initializer.


# ML packages
coreml model file:
* Metadata
* Interface - inputs, outputs
* Architecture - internal structure.  ex, model's layers, connections, etc.
* Learned parameters - 

Break these into separate files with `.mlpackage`.  Container that stores each component in its own file.  By separating these components, easily edit metadata and changes with sourcecontrol.  Compile more efficiently.

## Demo
Now that xcode supports model packages, you can edit metadata in xcode.  

I can add, modify, and remove additional metadata fields as well.  

In addition to UI support, all this information is accessible with MLModel description API.  

Code that xcode automatically updates to each model.  

MLPackages support all existing stuff, plus a new type called "ML Program".

# ML programs
Mathematical expressions are abstracted, and presented in the form of a computation graph or network.  Dataflow diagrams.

In ML software library, model is expressed as operations in code.  New MLProgram model type aligns with code.

Human-readable text format *although the intention is you don't write it yourself*.  

Weights are typically serialized into a binary file.

|                          | Neural network                       | ML program                      |
|--------------------------|--------------------------------------|---------------------------------|
| Composed of              | Layers                               | Operations                      |
| Weights                  | Embedded in layer descriptions       | Decoupled serialized separately |
| Intermediate tensor type | Implicit, determined by compute unit | Explicit, specified in program  |

You can now use the same converter API to convert to an MLProgram by specifying iOS 15 as minimum deployment target

[[Get Models on Device Using Core ML Converters]]

Moving forward, MLProgram will be the favored format.  MLModel and MLPackage are still supported, but we won't be improving them.



# Typed execution

Neural network intermediate types - implicit, not specified in models.

* Automatically partitions layer graph
* Precision tied to compute unit (implicit min precision is Float16)
* Tuned for performance

* CPU - float32
* GPU - float16
* NE - float16

You have some control by specifying `.all`, `.cpuAndGPU` or `.cpuOnly`.  Defaults to all.   Global setting for the model, leaves performance on the table.

ML Programs have specific types.  

* Automatically partitions operations
* Respects programs explicit types as min precision - will not downcast
* Tuned for performance

* Float32 and Flaot16 support for all operations on GPU
* Float16 CPU support for some selected ops

## Demo
FP16 precision pass.  Casts every float to 16-bit. 

For images, we prefer SNR.  SNR>20 is pretty good.

`compute_precision=ct.precision.FLOAT32` to force float32.  Compare this to original tensorflow.  

Compare two ML programs across dataset.  Most deep learning models work just fine with FP16 precision.  That's why it's on by default.

FP16 will be eligible for neural engine, making it have better performance/battery.

Unlike NN models, if your model needs higher precision, don't have to change app code, just the model.  This demo notebook will be available.

* `minimum_deployment_target = target.iOS15`
* `compute_precision = precision.FLOAT32` if required

# Wrap up
* MLShapedArray
* ML Package
* ML Program
* Typed execution



