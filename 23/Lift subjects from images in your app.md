Discover how you can easily pull the subject of an image from its background in your apps. Learn how to lift the primary subject or to access the subject at a given point with VisionKit. We'll also share how you can lift subjects using Vision and combine that with lower-level frameworks like Core Image to create fun image effects and more complex compositing pipelines. For more information about the latest updates to VisionKit, check out “What's new in VisionKit." And for more information about person segmentation in images, watch "Explore 3D body pose and person segmentation in Vision" from WWDC23.

# Intro to subject lifting
the foreground object/objects of a photo.  Not always a person.

anything from a building, a plate of food, or pairs of shoes.  Images can have multiple subjects.  Note that subjects are not always an individual object.  In this example, the man and his dog together are the focal point of the image.

VisionKit + Vision

VisionKit - easily adopt system-like behavior.  Easy UI integration.Basic subject information.Image size is limited.

Vision: no out-of-the-box UI.
Multiple input sources
Higher image resolution
Good for advanced pipelines.

# subject lifting in VisionKit
Initialize an image analysis interaction and add to a view contianing an dimage.  Can be a UIImageView, but does not need to be.

On macOS, we create an ImageAnalysisOverlayView.

Interaction types.  
* .automatic - mirrors system behavior.  Use if you want subject lifting, live text, data detectors.
* .imageSubject - only includes subject lifting, if you don't want interactive text.

Can generate an image analysis via creating an ImageAnalyzer and calling `analyze`.  Asynchronously access a list of all subjects using `subjects` property.  `Subject` struct contains `image` and `bounds`.

Highlighted sxubjects contains a `Set<Subject>`.  Users can highlight a subject with long-press, but they can change in code as well.

look up a subject by point, `.subject(at:)`.  returns nil if no subject.

How to add a shadow?  We just draw the shadow ourselves.

# Diving down into Vision

VisionKit's api is the easiest way to get started with subject lifting.  For more advanced features, vision has you covered.

`VNGeneratePersonInstanceMaskREquest` and `VNGenerateForegroundInstanceMaskRequest`

saliency - attention, objectness - coarse, region-based analysis.  Not e that the generated aliency maps are lowres and not suitable for segmentation.  Can use for tasks like cropping.

Person segmentation - detailed segmentation masks for people.  use this if you specifically want to focus on people.

Person instance segmentation - separate mask for each person

[[Explore 3D body pose and person segmentation in Vision]]

In contrast, subject lifting is class-agnostic.  Any foreground object, regardless of its semantic class, can be segmented.  Notice how it picks up the car in addition to the people in the image.

Key concepts.
1.  Input image
2. foreground mask
3. apply to source image, get masked image
each distinct segmented object is referred to as an instance.  Pixel-wise information about these isntances.
512x512 single channel image.  background pixels are set to 0.  Instance labels are unordered.

subject lifting API - `VNGeneraetForegroundInstanceMaskRequest`.  make a request handler, perform request.  This is a resource intensive-task and do it on a bg thread.

Perform this asynchronously on a separate dispatch queue.

can get tight crop or full output.

For some operations, like applying mask effects, it can be more convenient to work with just segmentation masks instead.

`.generateScaledMaskForImage`.  Theoutput is a single-channel fp pixel buffer with soft segmentation mask.  Perfectly suited for use with coreimage.

performing masking in CI, preserves HDR input.  [[Support HDR images in your app]]

Use CI BlendWithmask filter.  The new bg image...

output is an HDR-preserved masked and composited image.

Demo.

## design outline
source image
FX pipeline
* execute ivsion request
* extract instance masks
* mask input image
* apply effect and composite
output image

VNImagePointForNormalizedPoint - coordinate conversion

Use bytesPerRow to solve alignment.  CVPixelBufferLockBaseAddress / unlockbaseaddress.

# Wrap up
* visionkit for ease of use
* vision for advanced applications
* works great with core image
* 


# Resources
* https://developer.apple.com/documentation/vision
* https://developer.apple.com/documentation/visionkit
