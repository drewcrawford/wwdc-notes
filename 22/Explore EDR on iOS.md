EDR and its implications.
* extended dynamic range
* Apple's HDR technology
* rendering technology
* pixel representation
In well-exposed content, the subject should be int he standard dynamic range, whereas special emissive should be in the higher range.

EDR's representation is capable of describing any arbitrary value.  Any display to render high dynamic range content regardless of its peak.

Baldur's gate 3
divinity: original sin 2
shadow of the tomb raider

apple tv, netflix
affinity photo, davinci resolve, cinema 4d, fcvinal cut pro, nuke, pixelmator pro

# What's new
EDR APIs now available on iOS and iPadOS
12.9" ipad pro on liquid retina xdr display
* reference mode
* edr rendering over sidecar

reference mode: enables pro workflows.  Similar to the reference presents on macOS
100 nits SDR peak
1000 nits HDR peak
Disables HDR tone mapping
disables ambient adaptation
white-point and luminance calibration


supports
| video format             | macos preset                   |
| ------------------------ | ------------------------------ |
| PQ (HDR10, Dolby vision) | HDR video (P3-ST 2084)         |
| HLG                      | HDR video (P3-ST 2084)         |
| BT.709                   | HDTV video (BT.709-BT.1886)    |
| BT.601 PAL/SECAM         | PAL & SECAM video (BT.601 EBU) |
| BT.601 NTSC/SMPTE-C      | NTSC Video (BT.601 SMPTE-C)    |

Reference mode is a single toggle that supports the 5 most common formats.
Dont' worry if your content is in another format.  Every other format will be colormanaged as in the detailed display mode.
By enabling reference mode in settings... colors with P3 color gamut are rendered accurately, so users can be confident videos are accurate/consistent.

without reference mode, headroom may change dynamically.

## Sidecar with reference mode
Eanbles EDR rendering with sidecar
iPad pro as secondary reference monitor
Same response as native Reference Mode

MBP - HDR video preset
In this configuration, both devices have P3 colro gamut, d65 whitepoint.
Similar response.  Additionally, both ocnfigurations support peak luminance of 1000 nits.

# EDR overview
Traditionally, floating point SDR was 0-1 range.
0.0 black, 1.0 SDR white

In EDR, SDR is still 0,1 while EDR > 1.0
Note that EDR is represented in a linear space.  So 2.0 is not *perceptually* 2x brighter
Not tone-mapped in 0-1 range.

EDR rendering ensures SDR always renderable
1.0 to current EDR headroom renderable

However, values over current EDR headroom is clipped.
Higher headroom eanbles brightter content
EDR headroom is dynamic.  
* display technology
* Current display brightness

Roughly equal to the display peak / SDR brightness.
So 10x = 1000 nits / 100 nits.

Device capabilities
| device                                         | edr headroom                         |
| ---------------------------------------------- | ------------------------------------ |
| conventional backlit mac, iPad, studio display | Up to 2x SDR                         |
| typical HDR10 display                          | Up to 5x SDR (1000 nit-peak display) |
| iPhone with super retina XDR                   | Up to 8x SDR                         |
| iPad Pro Liquid Retina XDR                     | Up to 16x SDR                        |
| Pro display XDR                                | Up to 400x SDR (XDR preset)                                     |

# Reading EDR content
See other EDR talks for different frameworks.
1.  JPG
2. CGImage
3. CGBitmapContext decode and convert
4. Create MTLTexture
5. Render with metal engine

1.  Create CGImage from an HDR image
2. Draw into float-point bitmap bontext
3. Create a metal texture
4. Load bitmap context into texture and draw

```swift
// Create CGImage from HDR Image
let isr = CGImageSourceCreateWithURL(HDRImageURL, nil)
let img = CGImageSourceCreateImageAtIndex(isr, 0, nil)

// Draw into floating point bitmap context
let width  = img.width
let height = img.height

let info = CGBitmapInfo(rawValue: kCGBitmapByteOrder16Host |CGImageAlphaInfo.premultipliedLast.rawValue | CGBitmapInfo.floatComponents.rawValue)

let ctx = CGContext(data: nil, width: width, height: height, bitsPerComponent: 16,
    bytesPerRow: 0, space: layer.colorspace, bitmapInfo: info.rawValue)

ctx?.draw(in: img,
          image: CGRect(x: 0, y: 0, width: CGFloat(width), height: CGFloat(height)))
```

```swift
// Create floating point texture
let desc = MTLTextureDescriptor()

desc.pixelFormat = .rgba16Float
desc.textureType = .type2D

let texture = layer.device.makeTexture(descriptor: desc)

// Load EDR bitmap into texture
let data = ctx.data

texture.replace(region: MTLRegionMake2D(0, 0, width, height),
                mipmapLevel: 0,
                withBytes: &data,
                bytesPerRow: ctx.bytesPerRow)
```

# Opting into EDR
If you already have EDR for macos, you're fine.

1.  Use `CAMetalLayer`
2. Opt-in using `wantsExtendedDynamicRangeContent`
3. Have bright content
	4.  Supported pixel buffer formats
	5. Supported transfer functions

 ```swift
 // Opt into using EDR
var layer = CAMetalLayer()
layer?.wantsExtendedDynamicRangeContent = true

// Use supported pixel format and color spaces
layer.pixelFormat = MTLPixelFormatRGBA16Float
layer.colorspace  = CGColorSpace(name: kCGColorSpaceExtendedLinearDisplayP3)
```

Color spaces
| pixel format                 | color space                          |
| ---------------------------- | ------------------------------------ |
| MTLPixelFormatRGBA16Float    | kCGColorSpaceExtendedLinearSRGB      |
| ''                           | kCGColorSpaceExtendedLinearDisplayP3 |
| ''                           | kCGColorSpaceExtendedLinearITUR_2020 |
| MTLPixelFormatBGR10A2Unorm   | kCGColorSpaceITUR_709_PQ             |
| ''                           | kCGColorSpaceDisplayP3_PQ            |
| ''                           | kCGColorSpaceITUR_2100_PQ            |
| ''                           | kCGColorSpaceDisplayP3_HLG           |
| ''                      |kCGColorSpaceITUR_2100_HLG

 *be sure to use extended variant of the colorspace as shown in the table*

# Querying headroom
Headroom queries are found on NSScreen.
* static color comopnent values
* potential max value for the screen in EDR mode
* max value for reference preset

dynamic color component values
current max value
notifications
EDR headroom changes

UIScreen
Dynamic color copmonent values
* potential max value for the screen
* current max value
* reference mode status
* no notificatiosn for changes in EDR headroom
* only reference mode status changes
in iOS you can't get display max headroom, you must infer from the situation in reference mode.

```swift
// Query potential headroom
let screen = windowScene.screen
let maxPotentialEDR = screen.potentialEDRHeadroom
if (maxPotentialEDR < 1.5) {
    // SDR path
}

// Query current headroom
func draw(_ rect: CGRect) {
    let maxEDR = screen.currentEDRHeadroom
    // Tone-map to maxEDR
}

// Register for Reference Mode notifications
let notification = NotificationCenter.default
notification.addObserver(self,
                         selector: #selector(screenChangedEvent(_:)),
                         name: UIScreen.referenceDisplayModeStatusDidChangeNotification,
                         object: nil)

// Query for latest status and headroom
func screenChangedEvent(_ notification: Notification?) {
    let status          = screen.referenceDisplayModeStatus
    let maxPotentialEDR = screen.potentialEDRHeadroom
}
```
example
```
screen Built-in Retina Display headroom 16.0
screen Studio Display headroom 2.0
```

status
| UIScreenReferenceDisplayModeStatusEnabled        | enabled and being displayed accurately     |
| ------------------------------------------------ | ------------------------------------------ |
| UIScreenReferenceDisplayModeStatusLimited        | enabled but cannot be achieved temporarily |
| UIScreenREferenceDisplayModeStatusNotEnabled     | Supported on this display but not enabled  |
| UIScreenReferenceDisplplayModeStatusNotSupported | Not supported on the display                                           |

# Tone mapping
* CA provides interfaces associated with EDR metadata
* supports multiple metadata parameters
* Tone-mapping is **Not** universally supported

```swift
// Check if EDR metadata is available
let isAvailable = CAEDRMetadata.isAvailable

// Instantiate EDR metadata
// ...

// Apply EDR metadata to layer
let layer: CAMetalLayer? = nil
layer?.edrMetadata = metadata
```
```swift
// HLG
let edrMetadata = CAEDRMetadata.hlg

// HDR10 (Mastering Display luminance)
let edrMetaData = CAEDRMetadata.hdr10(minLuminance: minLuminance,
                                      maxLuminance: maxContentMasteringDisplayBrightness,
                                      opticalOutputScale: outputScale)

// HDR10 (Supplemental Enhancement Information)
let edrMetaData = CAEDRMetadata.hdr10(displayInfo: displayData,
                                      contentInfo: contentInfo,
                                      opticalOutputScale: outputScale)
```

* HLG
* HDR10
* HDR10 (supplemental enhancement information)

typically
| color space                          | constructor |
| ------------------------------------ | ----------- |
| kCGColorSpaceDisplayP3_HLG           | HLG         |
| kCGColorSpaceITUR_2100_HLG           | HLG         |
| kCGColorSpaceITUR_709_PQ             | HDR10       |
| kCGColorSpaceDisplayP3_PQ            | HDR10       |
| kCGColorSpaceITUR_2100_PQ            | HDR10       |
| kCGColorSpaceExtendedLinearSRGB      | Depends     |
| kCGColorSpaceExtendedLinearDisplayP3 | depends     |
| kCGColorSpaceExtendedLinearITUR_2020 | Depends            |

Use supplemental information if available for HDR10.

# wrap up
[[Edit and Playback HDR Video with AVFoundation]]
[[Explore HDR rendering with EDR]]
[[Display EDR content with Core Image, Metal, and SwiftUI]]
[[Display HDR video as EDR with AVFoundation and Metal]]

