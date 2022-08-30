#vision 

Let's recap some vision highlights.
* first introduced in 2017
* Growing collection of computer vision algorithms
* Easy-to-use, consistent API
* Takes full advantage of Apple Silicon
* tvOS, iOS, macOS.  

* person segmentation
* hand pose estimation
* trajectory detection

# New revisions
## Text recognition
* text recognizer used in Live Text
* VNREcognizeTestREquestRevision3
* Discover which languages are supporetd by calling the API.  
Korean
Japanese

## Automatic language identification.
Specify the recognition languages using `.recognitionLanguages`
Accurate mode now supports `automaticallyDetectsLanguage`
Best used when language is not known upfront

## Barcode detection
* new ML-based barcode detector and decoder
* VNDetectBarcodesRequestRevision3
Symbologies supported
* `supportedSymbologies` API.

Faster for multiple codes - "constant time"
Detects more ocdes per image.
Improved bounding boxes
Handles more corner cases

[[Capture machine-readable codes and text with VisionKit]]

## Optical flow
* new ML-based optical flow generator
* VNGenerateOpticalFlowRequestRevision2

You might not be aware of what it does compared to other things.

* analyzes two time-adjacent images, typically video frames
* Provides a local per-pixel estimate of motion from one image to the next
* Each estimate is a 2D vector given by a `VNPixelBufferObservation`
	* one channel X, other channel Y

Demo.  Better noise, etc.

Use cases
* local motion in a video
* Security video
* Tracker initialization
* Precursor to further video processing
* Video interpolation
* video action analysis

### output options
v1 => original image resolution
v2 => bilinear upscampled to original image resolution

Keep in mind that the flow image will be stretched.  When workign with network output directly, account for network/aspect ratio etc.

You can skip upsampling by using `keepNetworkOutput = true`.  Then, by accuracy,
* low => 64x96
* medium => 80x112
* high => 128x160
* very high => 160x224

When to use network output?
* default behavior best
	* backwards compatibility
	* when bilinear upsampling is acceptable for upsampling
* network otuput is best
	* don't eneed full resolution
	* need control over upsampling quality
# Cleaning house
Removal of older technology
rectanglesrequest revision 1
landmarksrequestrevision 1

We'll continue to support for old binaries.  For this release, we're satisfying version 1 requests with v2 output.   We're removing the old v1 detector completely.  This saves space on disk.

Behaviors to expect
* v1 behavior is preserved
* won't return upsidedown faces
* same landmarks constellation
* execution time is on par
* accuracy is improved
* no code change is required

call to action
* the most recent revisions are recommended
* use v3
* Better accuracy and performance
* Explicitly specify your desired revision
* don't rely on default behavior
# Debugging
## Quick look preview support
* mose-over vision objects in the debugger
* visualize the result on the input image
* available in xcode playgrounds

Keep in mind that the image request handler must be around for quicklook preview support to use the original images for display.  Things will continue to work but you won't see the image.

* which observation is which
* keep iamge request handler in scope (to use it with the input image)
* useful for live tuning

# Wrap up
* new revisions
	* text recognition
	* barcode detection
	* optical flow
* removal of older revisions
* Quick look preview support


* https://developer.apple.com/forums/tags/wwdc2022-10024
* https://developer.apple.com/forums/create/question?&tag1=254&t&tag2=354030
* https://developer.apple.com/documentation/vision
