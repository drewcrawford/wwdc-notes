Today we'll give a detailed presentaiton for ProRAW images.  

What we'll discuss:
* What is Apple ProRAW
* How to capture
* How to store in the photo library
* How to edit and display

# What is Apple ProRAW
[[Advances in iOS Photography - 16]]

Cake vs ingredients

Lovingly baked by apple
Much faster to display
Multiple catures can be fused
Smaller filesize

Greater flexibilty for editing
No lossy compression
More bits of precision

Proraw: best of both

Has a similar look to HEIC
Lossless compression
Multiple exposures can be fused
Great dynamic range for editing

Reasonably fast to display

The ProRAW format is designed to maximze
* Compatibility
* Quality 
* Look

## Compatability
* Container is a standard Adobe DNG file
* Supported by Apple Photos, aDobe Lightroom, and many others
* Apps using ImageIO and Core Image get support automatically
* Supported by earlier iOS and macOS
* Contain full resolution, JPEG quality pre-rendered previews

## Apple ProRAW quality
The pixels in the container are
* Scene referred and linearizable
* Generated from multiple exposures and image fusion
* Lossless copmressed 12-bit RGB
* Up to 14 stops of dynamic range
* File sizes range from 10mb to 40mb
* Adapting based on photo content

## Look

* ProRAW images havea  default look at is consistent with HEIC
* Achieved using DNG tags
	* LinearizationTable
	* BaselineExposure
	* BaselineSharpness
	* ProfileGainTableMap (DNG 1.6)
	* ProfileToneCurve
* All tags are unique to each image can be overridden

Can store multiple semantic mattes (new in DNG 1.6).  

# How to capture
AVFoundation Capture APIs provide access to camera on iOS/macOS
* Streaming live preview
* capture photos
* record movies

Added support for capturing photo in new DNG format

[[Advances in iOS Photography - 16]]

Bayer RAW vs apple PRORAW

|                                     | Bayer RAW                           | Apple ProRAW                    |
|-------------------------------------|-------------------------------------|---------------------------------|
| Devices                             | iOS 10 and later                    | iOS 14.3 and later              |
| Apple Camera.app                    | No                                  | Yes                             |
| Quality prioritization              | `.speed`                            | `.speed` `.balanced` `.quality` |
| Cameras                             | Only single camera AVCaptureDevices | All AVCaptureDevices            |
| Zoom                                | No                                  | Yes                             |
| Depth                               | No                                  | No                              |
| Content aware distortion correction | No                                  | No                              |
| Live photo                          | Yes                                 | No                              |
| Segmentation mattes                 | No                                  | YES                             |

How can you capture apple proraw?
* Setting up device and session
* Setting up photo output
* Prepare for ProRAW capture
* Consuming captured ProRAW

## Setting up device and session
Use `.photo` preset
Or optionally find a format that supports highest quality photos
```swift
// Or optionally find a format that supports highest quality photos

guard let format = device.formats.first(where: { $0.isHighestPhotoQualitySupported }) else {
  // handle failure to find a format that supports highest quality stills
} 
do 
{	
  try device.lockForConfiguration()
  {
    // ...
  }
  device.unlockForConfiguration()
} 
catch 
{
  // handle the exception
}
```

In this example, using the first index from the list of supported items.  Use the selection that works best for your application.

How can you configure photo output?

## Setting up photo output

```swift
// Enable ProRAW on the photo output

private let photoOutput = AVCapturePhotoOutput()
private func configurePhotoOutput() {
  photoOutput.isHighResolutionCaptureEnabled = true
	photoOutput.isAppleProRAWEnabled = photoOutput.isAppleProRAWSupported
	//...
}
```

Do this before session as it will otherwise involve lengthy operation

```swift
// Select the desired photo quality prioritization

private let photoOutput = AVCapturePhotoOutput()
private func configurePhotoOutput() {
	photoOutput.isHighResolutionCaptureEnabled = true
	photoOutput.isAppleProRAWEnabled           = photoOutput.isAppleProRAWSupported
	photoOutput.maxPhotoQualityPrioritization  = .quality // or .speed .balanced
  //...
}
```

[[Capture high-quality photos using video formats]]

## Prepare for capture
* Find a supported ProRAW pixel format

```swift
// Find a supported ProRAW pixel format

guard let proRawPixelFormat = photoOutput.availableRawPhotoPixelFormatTypes.first(
  where: {
    AVCapturePhotoOutput.isAppleProRAWPixelFormat($0) 
  }) 
else {
	// Apple ProRAW is not supported with this device / format
}

// For Bayer RAW pixel format use

AVCapturePhotoOutput.isBayerRAWPixelFormat()
```

Proraw only vs proraw + processed types?
```swift
// Create photo settings for ProRAW only capture

let photoSettings = AVCapturePhotoSettings(rawPixelFormatType: proRawPixelFormat)

// Create photo settings for processed photo + ProRAW capture

guard let processedPhotoCodecType = photoOutput.availablePhotoCodecTypes.first 
else 
{
	// handle failure to find a processed photo codec type
}
let photoSettings = AVCapturePhotoSettings(rawPixelFormatType: proRawPixelFormat,
	processedFormat: [AVVideoCodecKey: processedPhotoCodecType])
```

Select a supported thumbnail codec type and dimensions

```swift
// Select a supported thumbnail codec type and thumbnail dimensions

guard let thumbnailPhotoCodecType = photoSettings.availableRawEmbeddedThumbnailPhotoCodecTypes.first 
else 
{
  // handle failure to find an available thumbnail photo codec type
}

let dimensions = device.activeFormat.highResolutionStillImageDimensions

photoSettings.rawEmbeddedThumbnailPhotoFormat = [
  AVVideoCodecKey: thumbnailPhotoCodecType,
  AVVideoWidthKey: dimensions.width,
  AVVideoHeightKey: dimensions.height]
```

Select the desired quality prioritization
```swift
// Select the desired quality prioritization for the capture

photoSettings.photoQualityPrioritization = .quality // or .speed .balanced

// Optionally, request a preview image

if let previewPixelFormat = photoSettings.availablePreviewPhotoPixelFormatTypes.first
{	
  photoSettings.previewPhotoFormat = [kCVPixelBufferPixelFormatTypeKey as String: previewPixelFormat]
}

// Capture!

photoOutput.capturePhoto(with: photoSettings, delegate: delegate)
```

## Consumign captured proRAW

```swift
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingPhoto photo: AVCapturePhoto,
                 error: Error?) 
{
  guard error == nil 
  else 
  {
    // handle failure from the photo capture
  }
	if let preview = photo.previewPixelBuffer 
  { 
    // photo.previewCGImageRepresentation()
    // display the preview
  }
	if photo.isRawPhoto 
  {
		guard let proRAWFileDataRepresentation = photo.fileDataRepresentation() 
    else 
    {
      // handle failure to get ProRAW DNG file data representation
    }
		guard let proRAWPixelBuffer = photo.pixelBuffer 
    else 
    {
      // handle failure to get ProRAW pixel data
    }
		// use the file or pixel data
	}
```

Semantic segmentation mattes with Apple ProRAW
* Automatically generated
* scene dependent
* available through coreimage etc.

To customize proraw, implement costumizer delegate methods

```swift
// Provide settings for lossless compression with less bits

class AppleProRAWCustomizer: NSObject, AVCapturePhotoFileDataRepresentationCustomizer 
{
	func replacementAppleProRAWCompressionSettings(for photo: AVCapturePhoto,
                                                 defaultSettings: [String : Any],
                                                 maximumBitDepth: Int) -> [String : Any] 
  {
    return [AVVideoAppleProRAWBitDepthKey: min(10, maximumBitDepth),
            AVVideoQualityKey: 1.00]
  }
}
```

Provide settings for lossless compression with less bits

Fro lossy compression, set quality level below 1.  Small filesize but impact on quality of photo.

Customizaing the compression settings

```swift
// Customizing the compression settings for the captured ProRAW photo

func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingPhoto photo: AVCapturePhoto,
                 error: Error?) 
{
  guard error == nil 
  else 
  {
    // handle failure from the photo capture
  }
    if photo.isRawPhoto 
  {
		let customizer = AppleProRAWCustomizer()
		guard let customizedFileData = photo.fileDataRepresentation(with: customizer) 
    else 
    {
      // handle failure to get customized ProRAW DNG file data representation
    }
		// use the file data
	}
```

# How to store in the photo library
## PhotoKit
Support for RAW image formats
Online developer documentation for more information

SAving a ProRAW asset with PhotoKit
Perform changes on the shared `PHPhotoLibrary`
Create an asset with `PHassetCreationRequest`
Add the Apple ProRAW file as the `.photo`

```swift
PHPhotoLibrary.shared().performChanges 
{
    let creationRequest = PHAssetCreationRequest.forAsset()
  
    creationRequest.addResource(with:.photo, 
                                fileURL:proRawFileURL, 
                                options:nil)
} 
completionHandler: 
{ 
  success, error in
  // handle the success and possible error
}
```

## Fetching RAW assets from the photo library
New enum `PHASsetCollectionSubtye.smartAlbumRAW`
Includes all image assets with a RAW resource

`.photo` vs `.alternatePhoto`.

Store the raw and jpeg as two distinct resources?  jpeg is the `.photo` and the .raw is the `.alternatePhoto`.  This model is not recommended.  Use capture settinsg to embed preview in the file.

Check the `uniformTypeIdentifier`.  Use `PHAssetResourceManager` to retrieve the RAW data.

```swift
let resources = PHAssetResource.assetResources(for: asset)
for resource in resources 
{
  if (resource.type == .photo || resource.type == .alternatePhoto) 
  {
    if let resourceUTType = UTType(resource.uniformTypeIdentifier) 
    {
      if resourceUTType.conforms(to: UTType.rawImage) 
      {
        let resourceManager = PHAssetResourceManager.default()
        resourceManager.requestData(for: resource, options: nil) 
        { 
          data in
          // use the data
        } 
        completionHandler: 
        { 
          error in
          // handle any error 
        }
      }
    }
  }
}
```

# How to edit and display
* Getting CIImages from a ProRAW
* Applying common user adjustments
* Getting linear scene-referred output
* Saving to other file formats
* Displaying on a Mac with Extended Dynamic Range

Possible using `CGImageSoruceCreateWithURL` and etc.

```swift
// Getting the preview image

let rawFilter = CIRAWFilter(imageURL: url)

return rawFilter.previewImage
```

Getting semantic segmentation mattes

```swift

return rawFilter.previewImage

// Getting segmentation mattes images

return CIImage(contentsOf: url,
               options: [.auxiliarySemanticSegmentationSkinMatte : true])
```
Getting the primary image.
```swift
// Getting the primary image

return CIImage(contentsOf: url, options:nil)

let rawFilter = CIFilter(imageURL: url, options:nil)

return rawFilter.outputImage
```

Applying adjustments

```swift
func get_adjusted_raw_image (url: URL) -> CIImage?
{
    // Load the image
    let rawFilter = CIFilter(imageURL: url, options:nil)
    
    // Change one or more filter inputs
    rawFilter.setValue(value, forKey: CIRAWFilterOption.keyName.rawValue)
   
    // Get the adjusted image
    return rawFilter.outputImage
}
```

new swift-friendly api

```swift
func get_adjusted_raw_image (url: URL) -> CIImage?
{
    // Load the image
    let rawFilter = CIRAWFilter(imageURL: url)
    
    // Change one or more filter inputs
    rawFilter.property = value
   
    // Get the adjusted image
    return rawFilter.outputImage
}
```

Applying common user adjusments
* exposure
* temperature, tint
* sharpness
* localToneMapAmount
	* Bring up darker regons and bring up brighter areas.

Getting access to 14 stops? ("linear scene-referred output")

```swift
// Turn off the filter inputs that apply the default look to the RAW

rawFilter.baselineExposure      = 0.0
rawFilter.shadowBias            = 0.0
rawFilter.boostAmount           = 0.0
rawFilter.localToneMapAmount    = 0.0
rawFilter.isGamutMappingEnabled = false

let linearRawImage = rawFilter.outputImage
```

```swift
// Use the linear image with other filters

let histogram = CIFilter.areaHistogram()

histogram.inputImage = linearRawImage
histogram.extent     = linearRawImage.extent

// Or render it to a RGBAh buffer

let rd = CIRenderDestination(bitmapData: data.mutableBytes,
                             width: 􀠪imageWidth, 
                             height: 􀠪imageHeight, 
                             bytesPerRow: 􀠪rowBytes,
                             format: .RGBAh)

rd.colorSpace = CGColorSpace(name: CGColorSpace.extendedLinearITUR_2020)

let task = context.startTask(toRender: rawFilter.outputImage, 
                             from: rect, 
                             to: rd, 
                             at: point)

task.waitUntilCompleted()
```

In the default look ("output-referred"), the max luma is the sunset.  But the sky area has 80% max.

In a linear scene-referred, sky is max radiance is still snset, but 12% is the min range.  This is more physically logical.

## Saving
```swift
// Saving to 8-bit HEIC

try ciContext.writeHEIFRepresentation(of: rawFilter!.outputImage!,
                                      to: theURL,
                                      format: .RGBA8,
                                      colorSpace: CGColorSpace(name: CGColorSpace.displayP3)!,
                                      options: [:])

```

```swift
// Saving to 10-bit HEIC

try ciContext.writeHEIF10Representation(of: rawFilter!.outputImage!,
                                        to: theURL,
                                        format: .RGBA8,
                                        colorSpace: CGColorSpace(name: CGColorSpace.displayP3)!, 
                                        options: [:])
```

By default, CIRAwFilterOutput is SDR.
By setting some options, it can output EDR.

Many mac displays can present EDR content using a Metal Kit View.

Displaying to a MetalKitView in EDR on Mac

```swift
class MyView : MTKView {
  var context: CIContext
  var commandQueue: MTLCommandQueue
  //...
}
```

[[Core image performance and optimization techniques]]

Init your Metal Kit View for EDR

```swift
// Create a Metal Kit View subclass

class MyView : MTKView {
  var context: CIContext
  var commandQueue: MTLCommandQueue
  //...
}

// Init your Metal Kit View for EDR

colorPixelFormat = MTLPixelFormat.rgba16Float

if let caml = layer as? CAMetalLayer {
  caml.wantsExtendedDynamicRangeContent = true
  //...
}

// Ask the filter for an image designed for EDR and render it

rawFilter.extendedDynamicRangeAmount = 1.0

context.startTask(toRender: rawFilter.outputImage, 
                  from: rect, 
                  to: rd, 
                  at: point)
```

You really have to see this in person.

# Wrap up
* What is apple proraw
* How to capture
* Store
* edit

* https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/capturing_photos_in_raw_and_apple_proraw_formats
* https://developer.apple.com/documentation/photokit
* https://developer.apple.com/documentation/coreimage
* https://developer.apple.com/documentation/photokit
* https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture

