# People analysis Technology
Face analysis.  
* Face detection
	* VNDetectFaceREctanglesRequest
	* High precision/recall
	* Arbitrary orientation
	* Different sizes
	* Partially occluded
		* Glasses
		* Hats
		* In v3
			* Masks
* Face landmarks detection
	* 76point constellation
	* Accurate pupil detection
* Face pose
* roll, pitch, yaw
* Face capture quality
	* expression, lighting, occlusion, blur, etc.
	* Comparative measure of the **same subject**.
	
Body analysis
* Human (body) detection
	* Upper body only
	* Full body (not default)
* Pose detection
* Hand pose detection
	* Returns collection of human hand joint names
	* chirality

[[Classify hand poses and actions with Create ML]]


# Person Segmentation
* Ability to separate people from the scene
* Use cases
	* Virtual background, sport analysis, autonomous driving, and more
	* Portrait mode
* Single frame API
* Streaming and offline processing
* macOS, iOS, iPadOS, tvOS
* Single matte of all people
* Stateful requst
	* Use the same request object across the entire sequence of frames


```swift
// Create request 
let request = VNGeneratePersonSegmentationRequest()

// Create request handler
let requestHandler = VNImageRequestHandler(url: imageURL, options: options)

// Process request
try requestHandler.perform([request])

// Review results
let mask = request.results!.first!
let maskBuffer = mask.pixelBuffer
```

1.  Revision.  We recommend to set explicitly.  Default will change in the future.
2.  qualityLevel.  
	1.  Accurate => default, computational photography
	2.  Balanced => Video
	3.  fast => Streaming
3.  outputPixelFormat
	1.  Onecomponent8
	2.  Onecomponent16Half
	3.  Onecomponent32Float

Result: Pixel buffer observation object.  

Fast: 1x, balanced: 4x, accurate: 10-40x.  See chart for memory etc.

## Applying a mask
Greenscreen type effect
```swift
let input = CIImage?(contentsOf: imageUrl)!
let mask = CIImage(cvPixelBuffer: maskBuffer)
let background = CIImage?(contentsOf: backgroundImageUrl)!

let maskScaleX = input.extent.width / mask.extent.width
let maskScaleY = input.extent.height / mask.extent.height
let maskScaled = mask.transformed(by: __CGAffineTransformMake(
                                  maskScaleX, 0, 0, maskScaleY, 0, 0))

let backgroundScaleX = input.extent.width / background.extent.width
let backgroundScaleY = input.extent.height / background.extent.height
let backgroundScaled = background.transformed(by: __CGAffineTransformMake(
                          backgroundScaleX, 0, 0, backgroundScaleY, 0, 0))

let blendFilter = CIFilter.blendWithRedMask()
blendFilter.inputImage = input
blendFilter.backgroundImage = backgroundScaled 
blendFilter.maskImage = maskScaled

let blendedImage = blendFilter.outputImage
```

## Other APIs
Several other fragments with similar functionality.
AVFoundation => available on newer generation device,s can be used when capturing a Photo with AVCaptureSession.

ARKit => Available on A12 Bionic and later devices, generates personal segmentation mask by processing the camera feed

CoreImage => Thin wrapper on top of vision API
```swift
let input = CIImage?(contentsOf: imageUrl)!

let segmentationFilter = CIFilter.personSegmentation()
segmentationFilter.inputImage = input

let mask = segmentationFilter.outputImage
```

AVFoundation => Selected iOS devices, with AVCaptureSession.
ARKit => Selected iOS devices with ARKit session
Vision => Multiplatform, online and offline processing
Core Image => Thin wrapper around vision

## Best practices
* Up to four people
* Person height is at least half of image height
* Avoid ambiguities
	* Statues
	* Pictures of people
	* Far distance

# Wrap up
People analysis overview and upgrades
* Masked face detection
* Adding pitch and continuous pose metrics
* Hand chirality

Person segmentation
* Vision and other APIs

https://developer.apple.com/documentation/vision/applying_matte_effects_to_people_in_images_and_video
https://developer.apple.com/documentation/vision



