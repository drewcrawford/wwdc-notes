Learn how to identify, load, display, and create High Dynamic Range (HDR) still images in your app. Explore common HDR concepts and find out about the latest updates to the ISO specification. Learn how to identify and display HDR images with SwiftUI and UIKit, create them from ProRAW and RAW captures, and display them in CALayers. We'll also take you through CoreGraphics support for ISO HDR and share best practices for HDR adoption.

# Intro to HDR images
HDR vs SDR.

HDR - brighter, more vibrant, etc.

having a display with an HDR range lets us render brighter tahn SDR white.  Headroom.  In this paradigm, reference white is brightest SDR white.  Above that point is headroom.

EDR - rendered in headroom.  In EDR paradigm, reference white is 1.0, and peak is EDR max.

[[Explore HDR rendering with EDR]]

why support HDR?  User created/provided content, supporting HDR makes it even better.  Available on all apple platforms.  

ISO/TS 22028-5 is an ISO tech spec
defining requirements and guidelines for HDR color encoding of still images
publication in 2023

ISO HDR

provides a new HDR reference display spec and color encoding for output-referred HDR still photographs

0.0005 - black
203 - reference white

what's in here?

| foo                | bar                                                 |
| ------------------ | --------------------------------------------------- |
| transfer functions | hybrid-log-gamma (HLG) or perceptual quantizer (PQ) |
| color encoding     | ITU-R BT.2020/2100-2 color primaries                |
| signal range       | 10 bit+                                             |
| metadata           | Coding-independent code points (CICP) or ICP profile                                                    |

Note that JPEG is not compliant as they only support 8 bits per component.

 optional metadata
 * reference environment
 * diffuse white luminance
 * scene referred
 * mastering and content color volume
 * content light level

see the spec.

Since 2020, iphone images have been captured with additional data that let us gain HDR from SDR image.  Gain-mapped HDR.  New APIs for accessing this in your app.  
# Displaying HDR images

`.allowedDynamicRange(.high)`.  Enables HDR.

UIKit - `.preferredImageDynamicRange = .high`.

3 options.
* high - want to display HDR content.  We figure it out, we update when display changes, etc.  safely with non-HDR content
* standard - disables HDR rendering.  Tone-map content outside SDR range.  
* constrainedHigh - default for UICollectionViews constrained option for showing some HDR, not all of it.

ex supopse we have mixed content, where some is HDR, some is HDR, etc.  By limiting how much headroom it uses, we make the carousel look more consistent.

"default for UIImageView and NSImageView shows fully dynamic range possible on the current display"

HDR can sometimes be very bright.  ex, thumbnails

> there is no option that does not involve tonemapping the image

if you want to avoid tonemapping, use a lower-level API

* hdr support requires a full pipeline
* deprecated APIs may not be HDR-safe

resizing image with UIGraphicsBeginImagecontextWithOptions, you will lose HDR and wide-gamut color.  Avoid 

If you are creating a thumbnail with `image.preparingThumbnail(of:)`.  right way to get HDR thumbnail.

iOS15 - UIGraphicsImageRenderer.  UIKit knows how to construct a renderer that avoids things getting lost.

photos picker `-preferredItemEncoding: .current`. and `matching: .images`.

[[Embed the Photos Picker in your app]]

with ISO HDR, I can create an image from the data representation with no extra code.
Supporting gain-mapped HDR, I can use `UIImageReader.default.image(data: data)`.  This returns HDR representation by default on an HDR display.

HDR view apis display HDR when provided HDR images
SDR images are always displayed as SDR.

In UIKIt
* UIImage.isHighDynamicRange
* in appkit/CG/CI, check CGColorSpace.  `usesITUR_2100TF` is ISO HDR.

wide range of headroom on displays.

iPHone 14: 8x
ipad pro 12.9: 16x
mbp: 16x
apple pro xdr: 400x
other apple displays: 2x
other displays: ?

UIScreen.main.potentialeDRHeadroom

on macOS NSScreen.main.maximumPotentialExtendedDynamicRangeColorComponentValue

* user content is great for HDR
* hdr has impact - it draws attention
* best to use when driven by user experience

recap
identify ISO hdr images
display ISO hDR images
adccess HDR images from photos
get max display headroom



# Reading and writing HDR

* read from file into memory
* modify images in memory
* convert
* write

keep in mind that image objects have a colorspace.  ex cgimage, ciimage, etc. 

variety of supported colorspaces, but ISO will have itur_2100_HLG or itur_2100_pq.

## read
UIImage(contentsOfFile:)
NSImage(contentsOf: url)

to support gain map, use UIImageReader with a configuration with `prefersHighDynamicRange = true`.

Only impacts gain-mapped HDR images.

with coreimage: just create CIImage.  can inspect with quicklook.

The image .colorspace property will tell you the colorspace of the file.

coregraphics:
`CGImageSoruceCreateImageAtIndex(source, 0, [kCGImageSourceDecodeREquest: kCGImageSourceDecodetoHDR])`

reading hdr as sdr using core image:

`CIImage(contentsOf: url, options: [.toneMapHDRToSDR: true])`

only works if the image has an HLG or PQ colorspace
looks the same as setting a view to use `dynamicRange.standard`

for coregraphics, the API is

`CGImageSourceCreateImageAtIndex(source, 0, [kCGImageSourceDecodeRequest: kCGImageSourceDecodeToSDR])`

Reading gain map hdr using core image (previously, only SDR was available):

`CIImage(contentsOf: url, options: [.expandToHDR: true])`

The `.colorspace` now tells you the colorspace of the file.

coregraphics:
`CGImageSourceCreateImageAtIndex(source, 0, [kCGImageSourceDecodeREquest: kCGImageSourceDecodeToHDR])`

also works if the image is a RAW file.

## raw
* raw files contain plenty of dynamic range
* raw files can be displayed as HDR

getting the default look for a RAW image
`return CIImage(ocontentsOf: url, otppions: nil)`

get HDR look: `.expandToHDR` option.

To get full functionality of raw, create `CIRawFilter`.  It's `.outputImage` is the default look. But the filter can be modified.  Many properties that your application can change to alter the output image.

[[Capture and process ProRAW images]]

adjust dynamic range amount `.extendedDynamicRangeAmount`.  Default value is 0, which indicates that the output image should be SDR.  max amount is 1, indicates that it uses the most headroom present in the file.

## modify HDR images
* over 150 of the filters built into core image support HDR
* all filters generate or pcoess images that contain HDR content
* most built-in filters work because CI's working space is unclamped linear
* check if a built-in filter supports HDR
* `f.attributes[kCIAttributeFilterCategories] as! Array<String>; categories.contains(kCICategoryHighDynamicRange)`

[[Display EDR content with Core Image, Metal, and SwiftUI]]

How to write HDR image.

`image.jpegData(compressionQuality:)` and `image.pngDAta()`.  

New this year, UIImage can write `image.heicData()` with ISO HDR, png is ISO as well.  Also works for gain-mapped images.

Core image can save ISO png file if you specify the right colorpsace, format rgba16

also hdr tiff.

Note that PNG/TIFF use lossless compression and have large filesizes.  Best practice is to write a heif file.  Specify HDR colorspace.

## conversions

### cvpixelbuffer
CVPixelBuffer/IOSurface.  These can be contents of CALayer, or biplanar chromasubsampled.

declare that it is `kCVPixelFormatType_420YpCbCr10BiPlanarFullRange` or other appropriate.
specify that the buffer should be surface-backed.

attachments:
* matrix: ITU_R
* primaries: ITU_R
* transferfunction: SMPTE_ST_2084_PQ


CIImage(withCVPixelBuffer:)
ciContext.render(image to: buffer)

## coreimage / CGImageRef

`context.createCGImage` with deep colorspace and RGBA16 or RGBAh.  New this year, we support RGBA10.  Deep and half the memory.

CGImage can be passed to many other APIs, but avoid for interactive displaying of images.  For that, we recommend using coreimage render directly to MTKView or pixelbuffer to CALayer.
# Advanced  HDR display
* sometimes you need more control over how content is composited into your app
* CALayer now supports
`CALayer.wantsExtendedDynaimcRangeContent`

cametallayer EDR uses
`CAMetalLayer.wantsExtendedDynamicRangeContent`

note this difference
* calayer enables tone mapping
* cametallayer does not!!

suppose display has 5x headroom and image has 10x.
* cametallayer - clamped (clipped) to what the display can show
* calayer - tonemapped with a curve

[[Explore HDR rendering with EDR]]

CAMetalLayer - create your own tonemapping pipeline

CALayer supported image classes:
cgimage
cvpixelbuffer
iosurface

with correct colorspace, etc.

## HDR pixel formats

| ciformat                     | kCVPixelFormatType              |
| ---------------------------- | ------------------------------- |
| .RGBAf                       | `_128RGBAFloat`                 |
| .RGBAh                       | `_64RGBAHalf`                   |
| .RGBA16                      | `_64RGBALE`                     |
| RGB10                        | `_30RGBLEPackedWideGamut`       |
| suppported via CVPixelBuffer | `_420YpCbCr10BiPlanarFullRange` |


CGImage flags.
not captured here.

older versions of iOS/macOS

isohdr:
* use the coreimage `.tonemapHDRToSDR ` option.
* Use Coregraphics with an SDR colorspace
gain map HDR
* use version checks to include new read option
* files will load and render as SDR

# wrap up
* new apis for reading/writing/displaying hdr images
* how to access gain map hdr
* hdr workflow apis

 
# Resources
* https://apple.co/3orEzXX
* https://developer.apple.com/documentation/avfoundation/video_effects/editing_and_playing_hdr_video
* https://apple.co/2NA4pft
* https://developer.apple.com/documentation/metal/hdr_content/processing_hdr_images_with_metal
* https://developer.apple.com/documentation/uikit/images_and_pdf/supporting_hdr_images_in_your_app
* 