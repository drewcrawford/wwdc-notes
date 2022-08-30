Use ML to solve ap roblem that would usualyl require an expert to perform some very specialized work.  How to add open source ML models to your apps and create fantastic new experiences.

Highlight a few of the many tools, fraemworks, and APIs availsble to build apps using ML.

Go through a series of decisions taht hopefully will bring the best experience to your users.  When adding ML to applications.

* Should I use ML?
* How can I get an ML model?
* How do I make a model copmatible with apple platforms?
* Will the model work for the specific use case?
* Does the model run on the ANE?

ex: colorize b&w photos

tremendous amount of frameworks and tools.  Data processing, model trailing, inference, etc.  For this journey I will use several.

# Development process
1.  Model search.  e.g. Colorizer.  

The colorizer expects b& w image as input.  python sorucecode converts any rgb image to a LAB space.  This colorspace has 3 channels.  Lightness (L), and 2 color components.

Lightness becomes the input of the colorizer model.  Model estimates two new color components.

```python
from skimage import color

in_lab = color.rgb2lab(in_rgb)
in_l = in_lab[:,:,0]
```

```python
from skimage import color
import numpy as np
import torch

out_lab = torch.cat((in_l, out_ab), dim=1)
out_rgb = color.lab2rgb(out_lab.data.numpy()[0,…].transpose((1,2,0)))
```

Now we make this compatible with the app.  Convert the pytorch model to coreml format using coremltools.
```python
import coremltools as ct
import torch
import Colorizer

torch_model = Colorizer().eval()

example_input = torch.rand([1, 1, 256, 256])
traced_model = torch.jit.trace(torch_model, example_input)

coreml_model = ct.convert(traced_model, 
                          inputs=[ct.TensorType(name="input", shape=example_input.shape)])

coreml_model.save("Colorizer.mlpackage")
```

Once the model is in coreml format, I need to verify that conversion worked correctly.  Directly in python using coremltools.  

```python
import coremltools as ct
from PIL import Image
from skimage import color

in_img = Image.open(“image.png").convert("RGB")
in_rgb = np.array(in_img)
in_lab = color.rgb2lab(in_rgb, channel_axis=2)

lab_components = np.split(in_lab, indices_or_sections=3, axis=-1)
(in_l, _, _) = [
    np.expand_dims(array.transpose((2, 0, 1)).astype(np.float32), 0)
    for array in lab_components
]
out_ab = coreml_model.predict({"input": in_l})[0]

out_lab = np.squeeze(np.concatenate([in_l, out_ab], axis=1), axis=0).transpose((1, 2, 0))
out_rgb = color.lab2rgb(out_lab, channel_axis=2).astype(np.uint8)
out_img = Image.fromarray(out_rgb)
```

Verify the functionality of converted model matyches functionality of original model.  Model verification.

One more important check to be done.  Need to understand if this can run fast enough on my device.  New CoreML performance report in xcode 14, #coreml, performs a time-based analysis of the model.  Drag and drop into xcode.

I can see that estimated prediction time is 90ms.  This is perfect.

[[Optimize your Core ML usage]]

Model integration.  Identical to what we did in python, but now in swift.  

Recall the model expects a single channel iamge representing its lightness.  Like what I did in python, must convert RGB input image to image using Lab.    Could write it in many ways.  vImage, using Metal.  But let's use Core Image.

```swift
import CoreImage
import CoreML

func colorize(image inputImage: CIImage) throws -> CIImage {

    let lightness: CIImage = extractLightness(from: inputImage)

    let modelInput = try ColorizerInput(inputWith: lightness.cgImage!)
    
    let modelOutput: ColorizerOutput = try colorizer.prediction(input: modelInput)

    let (aChannel, bChannel): (CIImage, CIImage) = extractColorChannels(from: modelOutput)

    let colorizedImage = reconstructRGBImage(l: lightness, a: aChannel, b: bChannel)
    return colorizedImage
}
```

```swift
import CoreImage.CIFilterBuiltins

func extractLightness(from inputImage: CIImage) -> CIImage {

    let rgbToLabFilter = CIFilter.convertRGBtoLab()
    rgbToLabFilter.inputImage = inputImage
    rgbToLabFilter.normalize = true
    let labImage = rgbToLabFilter.outputImage

    let matrixFilter = CIFilter.colorMatrix()
    matrixFilter.inputImage = labImage
    matrixFilter.rVector = CIVector(x: 1, y: 0, z: 0)
    matrixFilter.gVector = CIVector(x: 1, y: 0, z: 0)
    matrixFilter.bVector = CIVector(x: 1, y: 0, z: 0)
    let lightness = matrixFilter.outputImage!
    return lightness
}
```
Lets analyze the output of the model..  It returns MLShapedArrays.  So after the prediction we convert them to CIImages.  We need to reconstruct the image.
```swift
func extractColorChannels(from output: ColorizerOutput) -> (CIImage, CIImage) {

    let outA: [Float] = output.output_aShapedArray.scalars
    let outB: [Float] = output.output_bShapedArray.scalars
    let dataA = Data(bytes: outA, count: outA.count * MemoryLayout<Float>.stride)
    let dataB = Data(bytes: outB, count: outB.count * MemoryLayout<Float>.stride)

    let outImageA = CIImage(bitmapData: dataA,
        bytesPerRow: 4 * 256,
        size: CGSize(width: 256, height: 256),
        format: CIFormat.Lh,
        colorSpace: CGColorSpaceCreateDeviceGray())
    let outImageB = CIImage(bitmapData: dataB,
        bytesPerRow: 4 * 256,
        size: CGSize(width: 256, height: 256),
        format: CIFormat.Lh,
        colorSpace: CGColorSpaceCreateDeviceGray())
   return (outImageA, outImageB)
}
```
Use a custom CIKernel that takes the three channels as input and returns a CIImage.  use the new CIFilter convertLabToRGB to convert to RGB and return to caller.
```swift
func reconstructRGBImage(l lightness: CIImage,
                         a aChannel: CIImage,
                         b bChannel: CIImage) -> CIImage {
    guard
        let kernel = try? CIKernel.kernels(withMetalString: source)[0] as? CIColorKernel,
        let kernelOutputImage = kernel.apply(extent: lightness.extent,
                                             arguments: [lightness, aChannel, bChannel])
    else { fatalError() }

    let labToRGBFilter = CIFilter.convertLabToRGBFilter()
    labToRGBFilter.inputImage = kernelOutputImage
    labToRGBFilter.normalize = true
    let rgbImage = labToRGBFilter.outputImage!
    return rgbImage
}
```
Here is the sourcecode to combine lightness.
```swift
let source = """
#include <CoreImage/CoreImage.h>
[[stichable]] float4 labCombine(coreimage::sample_t imL, coreimage::sample_t imA, coreimage::sample_t imB)
{
   return float4(imL.r, imA.r, imB.r, imL.a);
}
"""
```

See [[Display EDR content with Core Image, Metal, and SwiftUI]]

# Live coloring
My model needs 90ms to process an iamge.  If i want to process a video, I need something faster.  For smoother UX, run at least 30fps.  Camera produces a frame every 30ms.  Since the model needs about 90ms to colorize a frame, I lose 2-3 frames during each colorization.

Total prediction time of amodel is a function of its architecture as well as the compute ops it is mapped to.  Looking at performance, I see my model is 61 ops between neural engine and CPU.  If I want a faster prediction time, I need to change the model.

Experiment with the model architecture and explore some alternatives that are faster.  But changing the architecture means I need to retrain the network.  Since the original model was developed in pytorch, I decided to use pytorch on metal to retrain.

[[Accelerate machine learning with Metal]]

After retraining, I need to re-convert and re-verify.  

Performance report tells me only one aspect.  AFter runin gmy app, colorization is not as smooth as I expect.  What happens at runtime?  Let's see core ml template in instruments. #instruments 

analyzing initial performance, I notice that the app accumulates predictions.  I'd expect o have a single prediction per frame.  NE is still working on the first request when the second is given.   Lag already 20ms.  need to amke sure that a new prediction starts only if the previous one is finished to avoid cascading these lags.

I accidentally set the camera frame rate to 60fps instead of 30fps.  

# Demo
Taking advantage of rectangle detection in visiomn framework.  VNDetectRectangleRequest to isolate photo in the scene.

# Recap
Enormous amount of frameworks, APIs, and tools Apple provides to prepare, integrate, and evaluate ML functionality for your apps.

* Solving a problem with ML
* Model search and conversion
* Model assessment directly on device
* In-app integration
* Optimization for a great UX


* https://coremltools.readme.io/docs
* https://coremltools.readme.io/docs/pytorch-conversion
* https://developer.apple.com/documentation/coreml/integrating_a_core_ml_model_into_your_app
* https://developer.apple.com/documentation/coreml

