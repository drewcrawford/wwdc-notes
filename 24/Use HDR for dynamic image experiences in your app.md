

Discover how to read and write HDR images and process HDR content in your app. Explore the new supported HDR image formats and advanced methods for displaying HDR images. Find out how HDR content can coexist with your user interface — and what to watch out for when adding HDR image support to your app.

continues [[Support HDR images in your app]]

We created some exceptional tech related to HDR images.

# HDR concepts and technologies

What is HDR?  At its core, HDR is a set of technologies created to represent the visual world with greater fidelity.  Capture and display of a wider range of light intensity found in real life.  Compared to SDR, it can display a deeper range of colors.

Rules, transformations that enable displaying brighter/deeper content on HDR screens.  Tonemapping.

## headroom

Dynamic range refers to contrast between brightest and darkest tone of an image.  On SDR display you can display only part of ranges.  Tonal compression.

HDR can represent both dark and bright tones better than SDR with fewer compromises.  ex specular highlights, or light coming from emissive objects.

Brightest SDR signal is known as 'reference white.'  Ex keynote presentation or book in an indoor setting.

Headroom is the ratio between the HDR peak and the reference white.  Can also be represented as logarithm of the value.  ex 1 or 2 stops above reference white to indicate HDR Is 2-4x brighter.

When a file is encoded that the data contains brightness level above reference white, we call it content headroom.

Display's ability to show levels above reference white -> display headroom.

Content headroom.  If display headroom capabity is sufficient, image will be decoded and rendered with full fidelity.

Display has 3 stops of headroom.  However, there are situations where the display may not be able to show the entire content headroom, ex current stream brightness.


## tonemapping

Adjusts the image brightness and color values to fit within the range that the medium or display can handle, ensuring accurate representation.  Occurs when a photo is captured/edited, or image is decoded/displayed.

Artistic or creative adjustment.  ex, an artist will import an image, and analyze it on a reference display, and tonemap to either SDR or a defined HDR headroom based on creative intent.

Display adjustment.  After HDR image is decoded, it can be displayed on a variety of devices, including some that may have limited headroom.  Each device can be located on a variety of physical environments.  In each case, the display will need to adjust the rendering of the image according to the situation.

Display capabilities -> how many nits the hardware can show.
Brightness settings -> resulting headroom available
battery -> display may need to dim in low power
Coexistence -> OS will promote the image in the fg to HDR and tonemap to SDR images in the bg.
Application -> present photo in a different manner.
## ISO HDR

First standard for HDR photography

ISO/TS 2208-5

part 5

iso.org/standard/8163.html

10bit PQ/HLG encoding
HDR reference display
HDR metadata
HEIF, AVIF, PNG, JPEG XL, etc.

Refer to the document number above.

Tone mapping to SDR (ITU-R BT.2408/2446/2390)
## adaptive HDR
Builds upon ISO HDR. 
* backward compatibility
* dual rendering
* tone map
Fundamental idea is to store in a file a fully backward-compatible SDR baseline representation.
Together with specific metadata that preserves spatial locations of bright areas.  Commonly called a gain map.

Since 2020, iPhone cameras have been capturing images with embeded gain maps.  See developer portal.

New this year we are standardizing gain map technologies.  

G = log2((HDR + kHDR)/(SDR + kSDR) )
heic, jpeg

ISO/NP 21496-1 committee draft.  Working towards the final stage.  For more details, locate the ISO doc.  Guarantees a uniform experience across software and hardware platforms.  

Backward compatibility -> guaranteed because we have an sdr baseline
Dual rendering -> since you can show both versions
Tone map -> since gain map is defined as ratio, we can tonemap based on display headroom.  Any desired output headroom can be mapped.

Can be an RGB 3-channels map, providing greater control over image appearance.  Standard allows for symmetrical transformations.

Baseline transition can be HDR and gain map can tonemap to SDR.  With release of iOS 18, we are transitioning to adaptive HDR gainmaps and related metadata.  New generation is based on the latest draft.
## file anatomy

New HEIC files captured on iOS 18 will still contain only one image.  In fact, `CGImageSourceGetCount` will return 1.

But developer can request alternate look.  By default, images decoded to standard range look.  When apps request it, they can contain the alternate representation of the image, the HDR image.  

In HEIF parlance, we call this the TMAP alternate, or tonemapped image.  File does not contain an HDR image at all.  But rather a set of ingredients and a recipe that together will create the HDR look.

Working with MPEG body to formalize and standardize adaptive HDR for HEIF which is currently included in working draft of heif spec.  ICC -> part of color profiles.

File anatomy.  ProRAW also supports HDR by including gain map and metadata.

| x               | apple gain map, iOS 17 | adaptive HDR, iOS 18  |
| --------------- | ---------------------- | --------------------- |
| File formats    | HEIC, JPEG             | same                  |
| baseline format | SDR 8-bit              | same                  |
| Gain map        | Apple Gain Map         | ISO Gain Map          |
| Gain map format | 8 bits, 1/4 size       | same                  |
| Metadata        | Apple metadata         | ISO gain map metadata |
| SDR color space | P3                     | same                  |
| HDR color space | N/A                    | P3-PQ                 |
## Display rendering

ISO HDR -> Global curve for all images
Adaptive HDR -> Local information for each pixel

this year we have a new apple reference white tone mapping operator

Adaptive hdr images will be adjusted down to display headroom using a curve optimized according to gain map in the file.

Before david shares the new apis, I'd like to highlight the system maps.

In iOS 17 and macOS 14, phtos app was the only app capable of rendering with full display headroom.

in iOS 18 we're adding messages, quicklook, and preview.  We changed photos app to use all these new apis.


# Read HDR images

Incredible thing about adapative HDR files is the flexibility.  because of gain map and metadata, image can be loaded in SDR, for backwards compatibility.  Or HDR for max fidelity.  By default, reading a gain map image will load the SDR representation into memory.  

`CGImageSourceCreateImageAtIndex` with `KCGImageSourceDecodeRequest:...`.

CIImage `image.contentHeadroom` property.  For common SDR images, the headroom is 1.

For iPhone HDR photos, they're >1, up to 8 depending on scene content.  For some images, it may be 0 to indicate headroom is unknown.

`CGImageGetContentHeadroom`.  
IOSurface has an equivalent property

`CIImage(cvPixelBuffer:)`

# Edit strategies

Perhaps the obvious way is to treat like SDR.  
Treat as HDR.  
Two images.  SDR, HDR. 

To use HDR approach, read the image with the `expandToHDR` option.  HDR image can be edited using filtters that preserve HDR range.  See [[Support HDR images in your app]] for filters that preserve HDR.

How editing an image will alter its content headroom property.  Certain modifications, CI knows that the headroom will be unchanged.  Ex if you scale, crop, warp, convolute, etc.

Sometimes CI does not know how the headroom is affected.  Resulting property may be 0.

To use HDR+Gain, load them as two in-memory image objects.  Use auxiliary gain map option to load the gain map as a ci image object.  Keep in mind, base image is SDR.  So its content headroom will be 1.

SDR image, analagous edits are applied to the gain map.  ex if SDR image is cropped, gain map should be cropped too.  gain image is typically half the size, so account for scale difference.

To use sdr+hdr, load them both as dedicated images.  Keep in mind that SDR image headroom will be 1, and HDR headroom will be >1.

when editing SDR image, analagous edits to the HDR.

pros and cons

| edit strategy | pros                                                                                    | cons                                                                |
| ------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| HDR           | simpler to implement<br>works with both HDR and gain map image                          | Some edits don't support HDR<br>tone map back to SDR is approximate |
| SDR+gain      | Preserves original gain for saving<br>works well for simple edits                       | Some edits can't be applied to gain                                 |
| SDR+HDR       | Allow apps to tune both HDR and SDR looks<br>Allows regeneration of gain map for saving | Need to track two images                                            |

# Display tone mapping
Tone mappingn gives the best rendering of an image given the current display state
May need to tone map to SDR or to some constrained headroom
The system's tone mapping is available via multiple APIs
Provides a consistent look across applications
UIImageview and SwiftUI support this automatically

Can display with CoreImage and metal.  when interactively changing the image.  When displaying adaptive hdr images, depends on what edit strategy was used.  Regardless of strategy, goal is to use image content headroom and display headroom so it looks optimal for display state.  Using HDR strategy, use new tonemapHeadroomFilter before displayed.  If the file is an ISO HDR file, than the reference white tonemapping operator will be used.

Adaptive HDR, apply custom tonemap that is optimized according to the file's unique gainmap.

[[Display EDR content with Core Image, Metal, and SwiftUI]]

`imageByApplyingGainMap:headroom:`.

Can render HDR image into EDR bitmap context.  
# Saving images

Best practice for saving depends on what strategy was used for loading/editing.

If you have loaded as HDR, modern method is to save 10-bit heif with PQ.

Loaded/edited as SDR+HDR, most compabile method is to save adaptive HDR.  `writeHEIFRepresentation` with two CIImages.  Edited image and colorspace.  CI will calculate the gain map and include with base HDR image.

SDR+Gain.  Call `writeHEIFRepresentation` to save both of these images.  Pass sdr image and colorspace, and gain image.  If gain image has original metadata, we use these to save to adaptive HDR.

Can use ImageIO to save SDR CGImage and gain map data.

# Wrap up
* adaptive HDR features and principles
* Strategies for reading, editing , displaying, saving





# Resources
* [Forum: Photos & Camera](https://developer.apple.com/forums/topics/media-technologies/photos-and-camera?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10177/4/5F09C774-7C5B-4605-98F5-8C70C4A56CF0/downloads/wwdc2024-10177_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10177/4/5F09C774-7C5B-4605-98F5-8C70C4A56CF0/downloads/wwdc2024-10177_sd.mp4?dl=1)
* [Support HDR images in your app](https://developer.apple.com/videos/play/wwdc2023/10181)
* [Display EDR content with Core Image, Metal, and SwiftUI](https://developer.apple.com/videos/play/wwdc2022/10114)