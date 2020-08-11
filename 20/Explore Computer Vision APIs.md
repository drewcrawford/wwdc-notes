# Why
Computer vision can enhance your application even if it is not at the core of your business.

Camera can save your users a lot of steps

# APIs for computer vision
* VisionKit
	* High level, computer vision API
* Core Image
	* Image processing
* Vision
	* Image analysis
* CoreML
	* Machine learning inference

Today we will focus on CI and Vision.



[[Advances in Core Image: Filters, Metal, Visiona nd More - 17]]

# why use CI
* make your image more amenable for analysis
* visualize the results of analysis
* Augmentation for ML training

# make your image more amenable for analysis
* Scaling
	* `CILanczosScale`

```swift
import CoreImage.CIFilterBuiltins
let f = CIFilter.lanczosScaleTrasnformFilter()
f.inputImage = inImage
f.scale = 0.5
newImage = f.outputImage
```

* `CIAffineTrasnform`
* `CIBicubicScaleTransform`

* Morphology Operations
	* Dilate
		* `CIMorphologyRectangleMaximum`
	* Erode -> `CIMorphologyRectangleMinimum`
	* Close -> Minimum then maximum
		* Removing small areas of noise

Some algorithms only need monochrome.  `CIColorMatrix`.  `CIMaximumComponent` -> channel with greatest signal will be used

Noise reduction
* `CIMedianFilter`
* `CIGaussianBlur`
* `CIBoxBlur`
* `CINoiseReduction`

Edge detection
* `CIConvolution3x3` (Sobel)
* `CIGaborGradients` -> 2D gradient vector

Constrast enhance
* `CIColorPolynomial`
* `CIColorControls`

Binarization (convert to black and white)
* `CIColorThreshold`
* `CIColorThresholdOtsu` -> automatically based on histogram

Comparing images
* `CIColorAbsoluteDifference`
* `CILabDeltaE` -> formula designed to match human perception of color

develoepr.apple.com/documentation/coreimage/cifilter

[[Build Metal-based Core Image Kernels with Xcode]]

# Images come in a variety of colorspaces
* BT.709 / sRGB
* P3
* BT.2020 (HDR)

CoreImage automatically converts inputs to working space, which is unclamped, linear, bt.709 primaries
Your algorithm may want images in a different colorpspace.  In that case,

```swift
let colorspace = CGColorSpace(name: CGColorSpace.sRGB)!
image = image.matchedFromWorkingSpace(to: colorspace)
//apply your algorithm
image = image.matchedToWorkingSpace(from: colorspace)
```

# Using Core Image with Vision results
* re-generating barcodes

```swift
let f = CIFilter.barcodeGeneratorFilter()
f.barcodeDescriptor = vnBarcodeObservation.barcodeDescriptor
barcodeImage = f.outputImage
```
Apply `CIFilters` based on `VNFaceObservations`
* Example fasce-based vignette

```swift
let f = CIFilter.radialGradientFilter()
//need to convert between vision's coordinate system to CI's coordinate system
f.color0 = CIColor.clearColor
f.color1 = CIColor.blackColor
f.center = ciCenter(vnFaceObservation.boundingBox)
f.radius0 = ciRadius(vnFaceObservation.boundingBox)
f.radius1 = f.radius0 * 2.0

let vignette = f.outputImage.cropped(to: image.extent)
newImage = vignette.composited(over: image)
```

# Fundamentals of Vision
* the task -> what you want to do
	* `VNRequest`s, e.g. `VNDetectRectanglesRequest`, etc.
* the machinery -> what performs the work
	* `VNRequestHandler`, e.g. `VNImageRequestHandler`, etc.
* the results -> what you want to get back
	* `VNObservation`, e.g. `VNRectangleObservation`, etc.

# What's new
* Hand and body pose -> [[Detect Body and Hand Pose with Vision]]
* Trajectory Detection -> [[Explore the Action & Vision app]]
* Contour Detection
* Optical Flow

## Contour Detection
Find edges in my image.

```swift
let contourRequest = VNDetectContoursRequest.init()
contourRequest.contrastAdjustment = 1.0
contourRequest.detectDarkOnLight = YES
contourRequest.maximumImageDimension = 512 //tradeoff performance vs accuracy
```

### observation
`VNCountoursObservation`.  `topLevelContours` are the index path.  Inside, we have child contours.
`contourCount` can use to work through each contour
`contourAtIndexPath`.
`normalizedPath` -> CGPath.

Inside `VNContour`:
* Index path
* `normalizedPoints`
* `pointCount`
* `aspectRatio`
* `normalizedPath`

### working with VNContour
Vision uses a normalized coordinate space: our image is 1.0 high and 1.0 wide, with origin 0,0 in lower-left.  So this is different than your image space.

* mind the `aspectRatio` in your computations.

#### analyzing contours
`VNGeometryUtils` provides API for analysis
* `boundingCircle` -> smallest circle that completely encapsulates
* `calculateArea`
* `calculatePerimeter`

#### simplifying contours
* contours can be noisy.
* `polygonApproximationWithEpsilon:` -> filter the noise parts, so only strong points will stay.  

### Computer Vision Problem to Native Solution

Tasked to identify the dimples in a punch card.
We find a code in python, but how to run on our platform?

* Loading an image
	* `CGImageSource` or `UIImage` into `CIImage`
* Process the image
	* CIFilters

#### finding the dimples - image analysis
`VNImageRequestHandler` with `CIImage`
Analyze the image
* `VNRequests`
	* `VNDetectContoursRequest`
	* We might not even have to preprocess

#### visualizing results
* CoreImage
	* Composite with your image into the same context
	* `CIMeshGenerator`
	* `CITextGenerator`
* Use CoreGraphics or UIKit
	* Render into `CALayer`

#### demo
```swift
import UIKit
import CoreImage
import CoreImage.CIFilterBuiltins
import Vision


public func drawContours(contoursObservation: VNContoursObservation, sourceImage: CGImage) -> UIImage {
	let size = CGSize(width: sourceImage.width, height: sourceImage.height)
	let renderer = UIGraphicsImageRenderer(size: size)
	
	let renderedImage = renderer.image { (context) in 
		
		let renderingContext = context.cgContext
		
    // flip the context
    let flipVertical = CGAffineTransform(a: 1, b: 0, c: 0, d: -1, tx: 0, ty: size.height)
    renderingContext.concatenate(flipVertical)
        
		// draw the original image
		renderingContext.draw(sourceImage, in: CGRect(x: 0, y: 0, width: size.width, height: size.height))
		
		renderingContext.scaleBy(x: size.width, y: size.height)
		renderingContext.setLineWidth(3.0 / CGFloat(size.width))
		let redUIColor = UIColor.red
		renderingContext.setStrokeColor(redUIColor.cgColor)
		renderingContext.addPath(contoursObservation.normalizedPath)
		renderingContext.strokePath()
	}
	
	return renderedImage;
}

let context = CIContext()
if let sourceImage = UIImage.init(named: "punchCard.jpg")
{
	var inputImage = CIImage.init(cgImage: sourceImage.cgImage!)
	
	let contourRequest = VNDetectContoursRequest.init()
    
// Uncomment the follwing section to preprocess the image
//	do {
//			let noiseReductionFilter = CIFilter.gaussianBlur()
//			noiseReductionFilter.radius = 1.5
//			noiseReductionFilter.inputImage = inputImage
//
//			let monochromeFilter = CIFilter.colorControls()
//			monochromeFilter.inputImage = noiseReductionFilter.outputImage!
//			monochromeFilter.contrast = 20.0
//			monochromeFilter.brightness = 8
//			monochromeFilter.saturation = 50
//
//			let filteredImage = monochromeFilter.outputImage!
//
//			inputImage = filteredImage
//		}
	
	let requestHandler = VNImageRequestHandler.init(ciImage: inputImage, options: [:])

	try requestHandler.perform([contourRequest])
	let contoursObservation = contourRequest.results?.first as! VNContoursObservation
	print(contoursObservation.contourCount)
	_ = drawContours(contoursObservation: contoursObservation, sourceImage: sourceImage.cgImage!)
} else {
	print("could not load image")
}
```

#### What I didn't have to do
* No reliance on 3rd-party packages
* Never left the most optimal processing path
* Never had to convert between image types
	* Saves memory
	* Saves comptuational cost

## Optical flow
Analyze the movement between 2 frames
Traditionally, we might use registration
* alignment of the whole image

Optical flow
* Per pixel flow between X and Y
* e.g., imagine that we have 2 objects that move *apart*.

### results
* `VNPixelBufferObservation`
* Floating point image
* Interleaved X/Y movement

Use CI to visualize the result.

```cpp
//
//  OpticalFlowVisualizer.cikernel
//  SampleVideoCompositionWithCIFilter
//


kernel vec4 flowView2(sampler image, float minLen, float maxLen, float size, float tipAngle)
{
	/// Determine the color by calculating the angle from the .xy vector
	///
	vec4 s = sample(image, samplerCoord(image));
	vec2 vector = s.rg - 0.5;
	float len = length(vector);
	float H = atan(vector.y,vector.x);
	// convert hue to a RGB color
	H *= 3.0/3.1415926; // now range [3,3)
	float i = floor(H);
	float f = H-i;
	float a = f;
	float d = 1.0 - a;
	vec4 c;
		 if (H<-3.0) c = vec4(0, 1, 1, 1);
	else if (H<-2.0) c = vec4(0, d, 1, 1);
	else if (H<-1.0) c = vec4(a, 0, 1, 1);
	else if (H<0.0)  c = vec4(1, 0, d, 1);
	else if (H<1.0)  c = vec4(1, a, 0, 1);
	else if (H<2.0)  c = vec4(d, 1, 0, 1);
	else if (H<3.0)  c = vec4(0, 1, a, 1);
	else             c = vec4(0, 1, 1, 1);
	// make the color darker if the .xy vector is shorter
	c.rgb *= clamp((len-minLen)/(maxLen-minLen), 0.0,1.0);
	/// Add arrow shapes based on the angle from the .xy vector
	///
	float tipAngleRadians = tipAngle * 3.1415/180.0;
	vec2 dc = destCoord(); // current coordinate
	vec2 dcm = floor((dc/size)+0.5)*size; // cell center coordinate
	vec2 delta = dcm - dc; // coordinate relative to center of cell
	// sample the .xy vector from the center of each cell
	vec4 sm = sample(image, samplerTransform(image, dcm));
	vector = sm.rg - 0.5;
	len = length(vector);
	H = atan(vector.y,vector.x);
	float rotx, k, sideOffset, sideAngle;
	// these are the three sides of the arrow
	rotx = delta.x*cos(H) - delta.y*sin(H);
	sideOffset = size*0.5*cos(tipAngleRadians);
	k = 1.0 - clamp(rotx-sideOffset, 0.0, 1.0);
	c.rgb *= k;
	sideAngle = (3.14159 - tipAngleRadians)/2.0;
	sideOffset = 0.5 * sin(tipAngleRadians / 2.0);
	rotx = delta.x*cos(H-sideAngle) - delta.y*sin(H-sideAngle);
	k = clamp(rotx+size*sideOffset, 0.0, 1.0);
	c.rgb *= k;
	rotx = delta.x*cos(H+sideAngle) - delta.y*sin(H+sideAngle);
	k = clamp(rotx+ size*sideOffset, 0.0, 1.0);
	c.rgb *= k;
	/// return the color premultiplied
	c *= s.a;
	return c;
}
```

```swift
class OpticalFlowVisualizerFilter: CIFilter {
	var inputImage: CIImage?
	
	let callback: CIKernelROICallback = {
			(index, rect) in
				return rect
			}
	
	static var kernel: CIKernel = { () -> CIKernel in
		let url = Bundle.main.url(forResource: "OpticalFlowVisualizer",
								  withExtension: "ci.metallib")!
		let data = try! Data(contentsOf: url)
		
		return try! CIKernel(functionName: "flowView2",
								  fromMetalLibraryData: data)
	}()

	override var outputImage : CIImage? {
		get {
			guard let input = inputImage else {return nil}
			return OpticalFlowVisualizerFilter.kernel.apply(extent: input.extent, roiCallback: callback, arguments: [input, 0.0, 100.0, 10.0, 30.0])
		}
	}
}
```

```swift
var requestHandler = VNSequenceRequestHandler()
            var previousImage:CIImage?
			if (self.previousImage == nil) 
			{
				self.previousImage = request.sourceImage
			}
			let visionRequest = VNGenerateOpticalFlowRequest(targetedCIImage: source, options: [:])
			
			do {
				try self.requestHandler.perform([visionRequest], on: self.previousImage!)
				if let pixelBufferObservation = visionRequest.results?.first as? VNPixelBufferObservation
				{
					source = CIImage(cvImageBuffer: pixelBufferObservation.pixelBuffer)
				}
			} catch {
				print(error)
			}
			// store the previous image
			self.previousImage = request.sourceImage
			
			let ciFilter = OpticalFlowVisualizerFilter()
			ciFilter.inputImage = source
			let output = ciFilter.outputImage
```
