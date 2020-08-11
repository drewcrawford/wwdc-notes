# What is HDR?
* extends video dynamic range over SDR
* Significantly increased luminance range
	* SDR up to 100 cd / m^2 (nits)
	* HDR up to 10k cd / m^2 (nits)
* Brighter whites and deeper blacks
* Better contrast in shadows and highlights

Often associated with Wide Color (WC) gamut
Increased bit depth (10-bit or higher)

## Transfer function
Trasnfer functions map between linear light and non-linear code values
* Preserve artistic intent of original scene to user's display
* Opto-electronic transfer functions (OETFs) map from scene to code values
* Electro-Optical Transfer Functions (EOTFs) map from code values to display
* EOTF is not necessarily exact inverse of OETF.

### transfer functions
* Hybrid Log-Gamma (HLG)
	* backwards compatible with SDR, some clipping
	* scene-referred – code values are relative to scene
* Perceptual Quantizer (PQ)
	* Designed for constrast sensitivity of the human visual system
	* Display-referred – code values are relative to display
	
Dolby vision, HDR 10.

### HDR metadata
* designed to improve encoded video to display light mapping
* accounts for differences in mastering environment vs current environment
* contains stream, scene, or frame statistics

No metadata (HLG)
Static metadata (e.g. HDR10)
* constant over entire video

Dynamic metadata (e.g. dolby vision)
* varies per scene or per frame

BT.709 == srgb
P3
BT.2020 -> ultra hd television



# HDR formats supported by export
* HLG and HDR10
	* Industry standards that are supported by most HDR TVs and monitors
	* Note that in 2017 we added this to Final Cut X and Compressor
	* This year we're adding export to SDK
	* 
# HDR export via `AVAssetExportSession`
Typically,  you might export a single asset, or a composition.

* Designed to make exporting simple
* Use presets to define output encoding parameters
* H.264, HEVC, and Apple ProRes.
* Different maximum video resolutions
* Different video quality levels

```swift
// AVAssetExportSession code snippet

guard let exportSession = AVAssetExportSession(asset: sourceAsset,
                      presetName: AVAssetExportPresetHEVCHighestQuality) else {
	// Handle error
}

exportSession.outputURL = outputURL 
exportSession.outputFileType = AVFileTypeQuickTimeMovie 

exportSession.exportAsynchronouslyWithCompletionHandler {
	// Handle completion 
}
```

Existing HEVC presents are upgraded to preserve HDR format.

source -> destination
HDR10 -> HDR10
HLG -> HLG
SDR -> SDR

Apple ProRes presets also preserve HDR format.
HDR to SDR export.

HDR to SDR export
* Use H.264 presets to maximize backwards compatibility.  

Note that HDR is not currently supported for HEVC with alpha.

Mix of HDR and SDR content
* Inspects all source asets andp romotes to HDR

Mix of different HDR formats
1.  Prefer HLG over PQ.  AVAssetExportSession can convert between these.
2.  Prefers HDR metadata over no HDR metadata

|             |           | Second asset |     |           |
|-------------|-----------|--------------|-----|-----------|
|             |           | SDR          | HLG | HDR10(PQ) |
| First asset | SDR       | SDR          | HLG | HDR10(PQ) |
|             | HLG       | HLG          | HLG | HLG       |
|             | HDR10(PQ) | HDR10(PQ)    | HLG | HDR10(PQ) |

# HDR export via `AVAssetWriter`
Two common workflows.

Assets -> maybe AVComposition -> AVAssetReader -> AVAssetWriter
AVCAptureVideoDataOutput -> AVAssetWriter

Puts you in complete control of the export process.  Can explicitly specify video codec, bitrate, framerate, etc.

* codec
* bitrate
* framerate
* dimensions
* color space
* dynamic range
* video encoder (mac)

Not an exhaustive list.

```swift
// AVAssetWriter with sourceFormatHint

let assetWriter = try AVAssetWriter(url: outputURL, fileType: AVFileTypeQuickTimeMovie)

let outputSettings: [String: AnyObject] = [
 			//avassetwriter will construct reasonable defaults for other parameters base
			AVVideoCodecKey: AVVideoCodecTypeHEVC
		]

let assetWriterInput = AVAssetWriterInput(mediaType: AVMediaTypeVideo,
                                     outputSettings: outputSettings
                                   sourceFormatHint: videoFormatDescription)

assetWriter.add(assetWriterInput)

guard assetWriter.startWriting() else {
	throw assetWriter.error!
}
```

```swift
// AVAssetWriter with AVOutputSettingsAssistant
//this API is presets-based

let assetWriter = try AVAssetWriter(url: outputURL, fileType: AVFileTypeQuickTimeMovie)

let settingsAssistant = AVOutputSettingsAssistant(
                                     preset: AVOutputSettingsPreset.hevc1920x1080)

//we strongly recommend prividing this.  Prior to retrieving video settings.
//AVOutputSettingsAssistant can merge source and settings to figure out the right export.
settingsAssistant.sourceVideoFormat = videoFormatDescription

let newVideoSettings = settingsAssistant.videoSettings

// Opportunity to modify a few fields in newVideoSettings here

let assetWriterInput = AVAssetWriterInput(mediaType: AVMediaTypeVideo,
                                     outputSettings: newVideoSettings)

assetWriter.add(assetWriterInput)
guard assetWriter.startWriting() else {
	throw assetWriter.error!
}
```

## Relevant keys


### `AVVideoCodecKey`

HEVC
* common distribution format
* Provides good compression efficiency

Apple ProRes
* Mezzanine format
* Designed for pro workflows

### `AVVideoCodecsProperties`
Contains 3 children
* `AVVideoTransferFunctionKey`
	* `ITU-R BT.2100-2` Hybrid Log-Gamma (HLG)
	* `SMPTE ST 2084` (PQ)
* `AVVideoColorPrimariesKey`
	* Typically set to a wide color gamut, e.g. `ITU-R BT.2020-2`
* `AVVideoYCbCrMatrixKey`
	* Typically set to `ITU-R BT.2020-2` non-constant luminance system.  Specifies YUV-RGB conversions.

### `AVVideoCompressionsPropertiesKey`
`AVVideoProfileLevelKey`
* Must be set to `kVTProfileLevel_HEVC_Main10_Autolevel` for HEVC HDR exports
	* 8-bit HEVC HDR is not supported
* Not applicable to ProRes exports

### HLG settings example

|                               | Valid settings                                                                      |
|-------------------------------|-------------------------------------------------------------------------------------|
| `AVVideoCodecKey`             | `AVVideoCodecTypeHEVC`<br>Apple ProRes codec types                                  |
| `AVVideoColorPrimariesKey`    | Typically `AVVIdeoColorPrimaries_ITU_R_2020`                                        |
| `AVVideoTransferFunctionsKey` | `AVVideoTransferFunction_ITU_R_2100_HLG`                                            |
| `AVVideoYCbCrMatrixKey`       | Typically `AVVideoYCbCrMatrix_ITU_R_2020`                                           |
| `AVVideoProfileLevelKey`      | if HEVC, `kVTProfileLevel_HEVC_Main10_AutoLevel`<br>If Apple ProRes, not applicable |

### HDR10 settings example
|                                                         | Valid settings                                                                      |
|---------------------------------------------------------|-------------------------------------------------------------------------------------|
| `AVVideoCodecKey`                                       | `AVVideoCodecTypeHEVC`<br>Apple ProRes codec types                                  |
| `AVVideoColorPrimariesKey`                              | Typically `AVVideoColorPrimaries_ITU_R_2020`                                        |
| `AVVideoTransferFunctionsKey`                           | `AVVideoTransferFunction_SMTPE_ST_2048_PQ`                                          |
| `AVVideoYCbCrMatrixKey`                                 | Typically `AVVideoYCbCrMatrix_ITU_R_2020`                                           |
| `AVVideoProfileLevelKey`                                | if HEVC, `kVTProfileLevel_HEVC_Main10_AutoLevel`<br>If Apple ProRes, not applicable |
| `kVTCompressionPropertyKey_MasteringDisplayColorVolume` | Optional `NSData *`                                                                 |
| `kVTCompressionPropertyKey_ContentLightLevelInfo`       | Optional `NSData *`                                                                 |

Mastering Display Color Volume (MDCV)
* Describes display on which content was msastered

Content Light Level Information (CLLI)
* Information about the light levels of movie content

Ideally both values should be specified
* Allows more accurate HDR rendering
* If absent, decoders will be forced to assume some default

MDCV and CLLI SEI message formats are defined in `ISO/IEC 23008...`


# Supported platforms for HDR export
iOS
* HEVC hardware encoding
* Requires A10 fusion or newer

Macs
* HEVC and Apple Pro Res software encoding -> all macs
* hardware encoding -> newer macs runnign the new macOS.  Significantly faster

[[Edit and Play Back HDR video with AVFoundation]]



