Learn how to optimize your machine learning and AI models to leverage the power of Apple silicon. Review model conversion workflows to prepare your models for on-device deployment. Understand model compression techniques that are compatible with Apple silicon, and at what stages in your model deployment workflow you can apply them. We'll also explore the tradeoffs between storage size, latency, power usage and accuracy.

I assume that you already have a ML model.  Could be pretrained, fine-tuned, etc.

[[Train your machine learning and AI models on Apple GPUs]]

I will not be covering the code you write to integrate your model.  

[[Deploy machine learning and AI models on-device with Core ML]]
[[24/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]
[[Support real-time ML inference on the CPU]]

utilities to optimize and convert your models for use in apple frameworks.

Pytorch model and transform into mlmodel format.

You want to consider storage, memory, and compute available on the platform and devices you are targeting.  Need to be aligned with model size, accuracy, and latency requirements.

Model preparation is exploring your options and applying various optimizations to find the best alignment.

# Model compression

[[Use Core ML Tools for machine learning model compression]]

* palettization.  Weights with similar valuesare clustered together using hte various centroids
* Quantization -> take float weight values and linearly map them into the integer range.  Integer weights are stored, with scale and bias.  Later used to map integers back to float.
* Pruning.  Pack model weights with sparse representation.  

Example: stable diffusion XL.  5.16gb: too large for iPads

8bits: 2.58GB.
6 bits: 1.94GB.
4bits: 1.29GB.  Can't get a good image.

iOS 17 only supported per-tensor palettization.
All things clustered into a single table.  16 cluster centroids.  Low granularity.  Introduces more potential errors for large matrices.

* whole tensor
* each group of channels
* each channel individually
* blocks

in IOS 18 we now support per grouped channel lookup table.  Much better accuracy.

For linear quantization, where 17 allowed per-channel bias.  iOS 18 allows per block scale/bias.

For pruning, we now support sparse palletization and parse quantization.
Combine sparisty with other compression modes.

Workflows.  

Training-time compression can improve accuracy.  


Now we have a new workflow.  Calibration data.  It's inbetween compress post and training-time compression.

Try new compression representations.  New compression workflow that uses calibration data.  Many new features this year.  see docs.

# Stateful model

Usually we have inputs/outputs.  Input is processed by operations that model the input into th eoutput.

Dataflow inside the model only for 1 inference run.  However content may need a longer lifespan to store information across different runs.

State is used.  Model can read data from state and write back to state.  Information in the state is persistent and can be used across runs.

ex accumulator.  

This year we add supprot for stateful models.  

Transformers are one of the most popular models.  Several multi-head attention blcoks.  Usually the bottleneck during inference.

Composed of several MatMul and SoftMax, and large tensors which are Query, Key, Value.

Query vector of a current word will attend to Key vector of previous words.  Then fuse with Values.  

At each timestamp, the Key and Value vectors for previous tokens are already computed in prevoius steps.  To avoid duplication, we can use KV-cache.

KV-cache is a popular and effective technique in large language models to make decoding faster.  It's a perfect cas eto use a stateful model.  Without states, the KV cache can only be used with IO which is inefficient considering KV cache usually has large tensors.  With states, we can update in place more efficiently.

Fused representation knowns as SPDA.  Scaled dot product attention.  Previously when you converted a model to the top, CoreML tools broke the SPDA op including matmul, softmax, etc.  This year, with DT iOS 18, coreml tools will use an SPDA op in the converted model.

This SPDA op takes all inputs at once and calculates attention more efficiently.  

To learn more about SPDA op, see [[24/Accelerate machine learning with Metal|Accelerate machine learning with Metal]]

How to prepare mistral 7b.

In-place KV cache updates in the forward pass. 

In forward function, as we have registered buffers for keyCache and valueCache, we no longer need to have inputs.

`ct.StateType`.  Use the names that match the buffers in the torch model.


# Transformer optimization

# Multifunction model

During merging, we deduplicate shared weights as much as possible.  Shared feature extractor.  

Usually too expensive to fine-tune a whole model.  Instead use adapters.  Comparable performance to fine-tuning a whole model.  Different adapters for different tasks.  Usually not just at beginning or end of model, they interact with intermediate layers.

CoreML's new multifunction model to add adapters.  Weights for base model are shared as much as possible.

[[Deploy machine learning and AI models on-device with Core ML]]

# Wrap up

* new compression techniques and workflows
* State and multiple functions
* Transformer optimizations
