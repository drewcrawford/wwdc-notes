# What is HDR?
High Dynamic Range video
Extends video dynamic range over SDR
Produce more vibrant videos with brighter whites and deeper blacks
SDR up to 100 cd/m (nits)
HDR up to 10k cd/m (nits)
Often associated with Wide Color
Increased bit depth (10-bit or higher)
Transfer functions – HLG, PQ
[[Export HDR media in your app with AVFoundation]]

# Video editing workflows
`AVPlayerItem` and `AVAssetExportSession`

asset(s) -> AVComposition and/or AVVideoComposition -> AVPlayerItem -> AVPlayer

AVComposition/AVVideoComposition -> AVAssetExportSession

Asset(s) -> AVAssetReader -> AVAssetReader (accepts VideoComposition Output) -> AVAssetWriter


[[Advanced Editing with AV Foundation - 13]]
[[Working with Media in AV Foundation - 11]]


# Enable HDR for video editing
* AVVideoComposition configurations.  Blending/transformations that must be HDR-aware.
* Previewing in HDR
* AVAssetEXportSession and AVAsssetWriter settings

[[Export HDR media in your app with AVFoundation]]

## AVVideoComposition
Blend multiple layers of video, apply frame-level transformations.
Transition effects between 2 clips of video, swipe, etc.
This method is *not* good if you want to apply filtering effects, such as color, blur, etc.

```swift
// Create AVVideoComposition with custom compositor class
let videoComposition = AVMutableVideoComposition()

videoComposition.instructions = [videoCompositionInstruction]
videoComposition.frameDuration = CMTimeMake(value: 1, timescale: 30)
videoComposition.renderSize = assetSize
```
Leave `videoComposition.customVideoCompositor` unset to use the built-in compositor.

Note that nothing here says anything about HDR.  How to make HDR-capable?

**Nothing.**

Chances are, you will get HDR video out with the same composition instructions properly applied!

## Use the applying CI filters
Useful when you want to apply filtering FX and are working with a single layer of view.
Does *not* support blending multiple layers.

```swift
// Create AVVideoComposition using “applyingCIFiltersWithHandler”

let videoComposition = 
 AVMutableVideoComposition(asset: asset, 
 applyingCIFiltersWithHandler: {
     (request: AVAsynchronousCIImageFilteringRequest) -> Void in
     let ciFilter = CIFilter(name: “CIZoomBlur”)
     ciFilter!.setValue(request.sourceImage, forKey: kCIInputImageKey)
         request.finish(with: ciFilter!.outputImage!, context: nil)
     })
```

No app-level changes to enable HDR
Built-in CIFilters support HDR
Custom filters need to be HDR aware.

```cpp
// HDRHighlight.metal

#include <metal_stdlib>
#include <CoreImage/CoreImage.h>
using namespace metal;

extern “C” float4 HDRHighlight(coreimage::sample_t s, coreimage::destination dest) {       
	
    if (s.r > 1.0 || s.g > 1.0 || s.b > 1.0)
		return float4(2.0, 0.0, 0.0, 1.0);
	else
		return s;
}
```

[[Build Metal-based Core Image Kernels with Xcode]]

```swift
// ColorInverter.metal - not HDR ready

#include <metal_stdlib>
#include <CoreImage/CoreImage.h>
using namespace metal;

extern “C” float4 ColorInverter(coreimage::sample_t s, coreimage::destination dest) {       
	
	return float4(1.0 - s.r, 1.0 - s.g, 1.0 - s.b, 1.0);
}
```

## AVVideoComposition - custom compositor
* blending layers
* frame-level geometry transformation
* filter effects

```swift
// Custom compositor class
class SampleCustomCompositor: NSObject, AVVideoCompositing {
…
}


// Create AVVideoComposition with custom compositor class
let videoComposition = AVMutableVideoComposition()

videoComposition.instructions = [videoCompositionInstruction]
videoComposition.frameDuration = CMTimeMake(value: 1, timescale: 30)
videoComposition.renderSize = assetSize

videoComposition.customVideoCompositorClass = SampleCustomCompositor.self
```

* Work in custom compositor.
* Operate in 10-bit or higher bit depth pixel formats
* Aware of HDR (intensity > 1.0) when filtering
* Indicating support of 10-bit or higher bit depth pixel formats for both input and outputs
* Indicating support of HDR source frames

```swift
// Setting custom compositor to support HDR

class SampleCustomCompositor: NSObject, AVVideoCompositing {
	//note 10-bit formats here.  Framework will make sure you get
	//source frames in a listed pixel format
	var sourcePixelBufferAttributes: [String : Any]? =
    [kCVPixelBufferPixelFormatTypeKey as String:
                    [kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange]]

	var requiredPixelBufferAttributesForRenderContext: [String : Any] =
			[kCVPixelBufferPixelFormatTypeKey as String: 
                            [kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange]]

	//if this is false, the framework will think you cannot do HDR frames
	var supportsHDRSourceFrames = true
	//also need to make sure you support wide color.
	//framework assumes this if previous value is true, but set to true explicitly anyway
    var supportsWideColorSourceFrames = true

	func startRequest(_ request: AVAsynchronousVideoCompositionRequest) {
			 ...
	}

	func renderContextChanged(_ newRenderContext: AVVideoCompositionRenderContext) {
	}
}
```

## demo

# Have more control over video composition
Table of how we upgrade/downgrade input formats, see [[Export HDR media in your app with AVFoundation]] for details

|             |           | Second asset |     |           |
|-------------|-----------|--------------|-----|-----------|
|             |           | SDR          | HLG | HDR10(PQ) |
| First asset | SDR       | SDR          | HLG | HDR10(PQ) |
|             | HLG       | HLG          | HLG | HLG       |
|             | HDR10(PQ) | HDR10(PQ)    | HLG | HDR10(PQ) |

Set various fields in your `AVVideoComposition`  to override

* colorPrimaries
* colorTransferFunction
* colorYbCr..Matrix

If your app does not explicitly specify these color properties, the default behavior of the built-in compositor will produce HDR videos.
However, if you explicitly set these to SDR video space, then we will convert to SDR.

Not reproducing table for these properties

# HDR playback

How do you find out if you're on an HDR-capable system?

```swift
// API definition 
extension AVPlayer {
    @available(macOS 10.15, *)
    open class var eligibleForHDRPlayback: Bool { get }
}
```

* system is capable of consumign HDR video
* At least one display can display HDR video

HDR playback is not available on watchOS or Catalyst.

```swift
// API definition 
extension AVPlayer {
    @available(macOS 10.15, *)
    open class var eligibleForHDRPlayback: Bool { get }
}

// Set video composition color properties based on HDR playback eligibility 
if AVPlayer.eligibleForHDRPlayback {
     videoComposition.colorPrimaries = AVVideoColorPrimaries_ITU_R_2020
	 videoComposition.colorTransferFunction = AVVideoTransferFunction_ITU_R_2100_HLG
	 videoComposition.colorYCbCrMatrix = AVVideoYCbCrMatrix_ITU_R_2020
}
else {
	 videoComposition.colorPrimaries = AVVideoColorPrimaries_ITU_R_709_2
	 videoComposition.colorTransferFunction = AVVideoTransferFunction_ITU_R_709_2
	 videoComposition.colorYCbCrMatrix = AVVideoYCbCrMatrix_ITU_R_709_2
}
```

Note that a system that isn't capable of playing HDR may still be capable of exporting.  See [[Export HDR media in your app with AVFoundation]]

# Wrap up
* HDR video editing is available
* AVVideoComposition is at the center of video editing
* To enable HDR for your custom compositor
* Set 10-bit pixel formats
* Set `supportsHDRSourceFrames` to true
* Control composition color space
* Check `eligibleForHDRPlayback`.


