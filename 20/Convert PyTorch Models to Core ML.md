#coreml 

# background

At WWDC 2020, we announced an overhaul to CoreML converters.  Expanded support for common libraries.  Redesigned architecture.  In-memory representation.  Unified API so a single call for any model source.

[[Get Models on Device Using Core ML Converters]]

How do you convert the PyTorch model to a coreml model?
Old coverter required you to export to onnx.  But this has limitations.  Onnx moves slowly so it complicates conversion on both ends.  Removing this dependency is good.


# CoreML from PyTorch
Use PyTorch's 'JIT' module to convert to TorchScript.  Then with TorchScript model, invoke the CoreML converter to generate mlmodel.

```python
import coremltools as ct
model = ct.convert(
    source_model, #TorchScript model / path to saved model
	inputs = [ct.ImageType(name="input", shape=(3,224,224))],
)
```

## under the hoold
* iterate over torchscript graph
* convert to coreml equivalent

Sometimes a TS op might convert to multiple ops.  Other times they might be fused.

## unimplemented operations
Converter is designed to be extensible so you can add definitions for new ops.  Express as a combination of existing ops, which we call a composite ops.

If not sufficient, can write a custom swift implementation and target that during conversion.  See online resources.

## how to get torchscript?
JIT trace (torch.jit.trace)
JIT script (torch.jit.script)

# Tracing
By invoking the trace method of pytorch's jit module

```python
import torch
traced_model = torch.jit.trace(mode,example_input)
```

Runs the example input through a forward pass, captures the operations that are invoked through the model layers.  The collection of operations becomes the TorchScript implementation.

Best thing to use is data similar to what the model will see during normal fuse.  
Make sure that range of input values is consistent with what the model expects.

## demo
### segmentation model
takes an image and assigns a class probability score to each pixel of the image.
PyTorch model: torch.jit.trace

TorchScript model: coremltools.convert

```python
# # Converting a Segmentation Model via CoreML

# ### Imports

import urllib
import torch
import torch.nn as nn
import torchvision
import json

from torchvision import transforms
import coremltools as ct
from PIL import Image


# ### Load Sample Model and Image

# Load model
model = torch.hub.load('pytorch/vision:v0.6.0', 'deeplabv3_resnet101', pretrained=True).eval()
# Load sample image
input_image = Image.open("dog_and_cat.jpg")
display(input_image)


# ### Image Preprocessing

to_tensor = transforms.ToTensor()
input_tensor = to_tensor(input_image)
input_batch = input_tensor.unsqueeze(0) #need batch size because model expects it


# ### Trace the Model with PyTorch

# First attempt at tracing
trace = torch.jit.trace(model, input_batch)
```

At this point, tracing throws an exception.  `Only tensors or tuples of tensors can be output from traced functions`

Model is returning dictionary.  So we need to wrap and extract the tensor value only

```
# ### Wrap the Model to Allow Tracing

class WrappedDeeplabv3Resnet101(nn.Module):
    
    def __init__(self):
        super(WrappedDeeplabv3Resnet101, self).__init__()
        self.model = torch.hub.load('pytorch/vision:v0.6.0', 'deeplabv3_resnet101', pretrained=True).eval()
    
    def forward(self, x):
        res = self.model(x)
        x = res["out"]
        return x


# ### Trace the Wrapped Model

traceable_model = WrappedDeeplabv3Resnet101().eval()
trace = torch.jit.trace(traceable_model, input_batch)


# ### Convert to Core ML 

# Define input
_input = ct.ImageType(
    name="input_1", 
    shape=input_batch.shape, 
	#resnet 101 expects this for some reason
    bias=[-0.485/0.229,-0.456/0.224,-0.406/0.225], 
    scale= 1./(255*0.226)
)

# Convert model
mlmodel = ct.convert(
    trace,
    inputs=[_input],
)


# ### Set the Model Metadata

labels_json = {"labels": ["background", "aeroplane", "bicycle", "bird", "board", "bottle", "bus", "car", "cat", "chair", "cow", "diningTable", "dog", "horse", "motorbike", "person", "pottedPlant", "sheep", "sofa", "train", "tvOrMonitor"]}

mlmodel.type = 'imageSegmenter'
mlmodel.user_defined_metadata['com.apple.coreml.model.preview.params'] = json.dumps(labels_json)


# ### Save the Model for Visualization

mlmodel.save("SegmentationModel.mlmodel")
```

Now click on saved model in finder and it will be opened by xcode.  View metadata, types, etc.

Some models cannot just be traced.  to explain how to convert other models...
# Scripting
Get TorchScript with JIT scripting
Directly compile PyTorch into TorchScript operations.

Like tracing, scripting a model is also easy.

```python
import torch
model = MyModel()
scripted_model = torch.jit.script(model)
```

when to use one vs the other?

* use scripting for control flow.  If we trace the model, we get only the path for the given input, which is not the whole module.
* trace where possible, script only when necessary.  Tracing usually produces a simpler representation than scripting.

## tracing vs scripting
```python
class MyModel(nn.Module):
    def __init__(self):
	   ...
	   self.loop_body = torch.jit.trace(...) #isolate the loop to trace on its own
	   
    def forward(self,x):
	    for _ in range(5):
		    x = self.loop_body(x)
		return x

model = torch.jit.script(MyModel()) # script the model as a whole
```

Basically limiting the scripting to just the bits of controlflow that need it, tracing everything else.

## demo (scripting)
Suppose I have a sentence completion model that I want to convert to coreml to run on device.
Sentence completion is a task that involves taking a sentence fragment and using a model to predict the following words.

Format: use tokens for words.  Special token for end-of-sentence.

```python
def forward(fragment):
    sentence = fragment
	token = 0
	while token is not EOS:
	    token = GPT2(sentence)
 	    sentence = stence + [token]
	return sentence
```

Wrap controlflow around the predictor until I see EOS token.
 
```python
# # Converting a Language Model via Core ML

# ### Imports
import torch
import numpy as np
from transformers import GPT2LMHeadModel, GPT2Tokenizer
import coremltools as ct


# ### Model
class FinishMySentence(torch.nn.Module):
    def __init__(self, model=None, eos=198):
        super(FinishMySentence, self).__init__()
        self.eos = torch.tensor([eos])
        self.next_token_predictor = model
        self.default_token = torch.tensor([0])
    
    def forward(self, x):
        sentence = x
        token = self.default_token
        while token != self.eos:
            predictions, _ = self.next_token_predictor(sentence)
            token = torch.argmax(predictions[-1, :], dim=0, keepdim=True)
            sentence = torch.cat((sentence, token), 0)
        
        return sentence


# ### Initialize the Token Predictor

token_predictor = GPT2LMHeadModel.from_pretrained("gpt2", torchscript=True).eval()


# ### Trace the Token Predictor
# 

random_tokens = torch.randint(10000, (5,))

traced_token_predictor = torch.jit.trace(token_predictor, random_tokens)


# ### Script the Outer Loop
# 

model = FinishMySentence(model=traced_token_predictor)
scripted_model = torch.jit.script(model)


# ### Convert to Core ML
# 

mlmodel = ct.convert(
    scripted_model,
    # Range for the sequence dimension to be between [1, 64]
    inputs=[ct.TensorType(name="context", shape=(ct.RangeDim(1, 64),), dtype=np.int32)],
)


# ### Encode the Sentence Fragment
# 

sentence_fragment = "The Manhattan bridge is"

tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
context = torch.tensor(tokenizer.encode(sentence_fragment))


# ### Run the Model

coreml_inputs = {"context": context.to(torch.int32).numpy()}
prediction_dict = mlmodel.predict(coreml_inputs)
generated_tensor = prediction_dict["sentence:2"]
generated_text = tokenizer.decode(generated_tensor)

print("Fragment: {}".format(sentence_fragment))
print("Completed: {}".format(generated_text))
```

# Troubleshooting and best practices
## Return value of traced model

```python
traced_model = torch.jit.trace(model,example_input)
RuntimeError: Only tensors or tuples of tensors can be output from traced functions (getOutput at ../torch/csrc/jit/tracer.cpp:212)
```

Wrap model to resolve error that unpacks the native outputs.

## Tracing python values
```python
traced_model = torch.jit.trace(model,example_input)
TracerWarning: Converting a tensor to a Python inex might cause the trace to be incorrect.  We can't record the data flow of Python values, so this value will be treated as a constant in the future.  This means that the trace might not generaliaze to other inputs!
```

```
nd,ns = w.size(-2), w.size(-1)
b = self.bias[:,:,ns-nd:ns,:ns]
```

Tracer is warning that it can't trace the math operations on bare python values.  However in this case the tracer is agressive and there isn't really a problem

Only use Python built-in operations on Python values.

```
slice = tensor1[:,tensor2.size(0) + 1] //ok
pad = tensor.size(0) % tensor2.size(0) //ok
scale = math.sqrt(tensor.size(1)) //won't trace correctly, trace doesn't understand math.sqrt and will collapse this to a constant value
```

but we can use instead
```python
scale = tensor.size(1) ** 0.5
```

```python
class TestNet(nn.Module):
	def forward(self,x):
		_list = []
		for i in range(10):
			_list.append(i)
		return _list
model = torch.jit.script(TestNet())
RuntimeError: arguments for call are not valid.
Expected a value of type 'Tensor' ... but instead found type 'int'
```

JIT scripter needs type information.  If the scriptor can't figure out the object's type, it assumes it is a tensor.  `_list` is not a list of tensors, it's a list of integers.

Have meaningful initialization, or use type annotations.

```python
class TestNet(nn.Module):
	def forward(self,x):
		_list = [0] #initialization
		for i in range(10):
			_list.append(i)
		return _list[1:] #type annotation
model = torch.jit.script(TestNet())
RuntimeError: arguments for call are not valid.
Expected a value of type 'Tensor' ... but instead found type 'int'
```

Make sure your model is in evaluation mode before tracing.
```python
import torchvision

model = torchvision.models.inception_v3()
model.eval() # evaluation mode
traced_model = torch.jit.trace(model,example_input)
```

This ensures that all layers are configured for inference rather than training.  But e.g. if you have dropout mode, evaluation mode will ensure it's disabled.

Converter ignores ops that are disabled by evaluation mode.  

# wrap up
Enable broader support for PyTorch models
Empower efficient model execution
Provide maximum support by unifying converters

