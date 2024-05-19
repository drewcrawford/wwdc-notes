Learn how you can match color requirements in demanding pro workflows using Reference Mode on the 12.9-inch iPad Pro with Liquid Retina XDR display. We'll show you how Reference Mode enables you to represent color accurately and provide consistent image representation in workflows like review and approval, compositing, and color grading. We'll also go over APIs to use with Reference Mode, explore its capabilities and supported media formats, and discover how Reference Mode enhances Sidecar.

# What is a reference display?
Many displays intentionally alter images
TVs have multiple modes
Mobile displays adapt to surroundings
Reference displays render to a specification
Color is consistent between reference displays
Need to be viewed in reference environment

# Reference Mode overview
## use cases
color-critical workflows
* color grading
* content view and approve

other applications:
* video editing
* compositing and VFX
Metal-based and AVFoundation-based rendering supported

On macOS, multiple references are available.  On iPad, it's either on or off.
Get reference color rendering for multiple formats without the need to change settings.  Only available for settings by user, can't be enabled/disabled by app.

12.9" iPad pro (5th gen)
Brings capabilities to sidecar.  Previously, sidecar did not support EDR.  But if it's enabled on an iPad as sidecar, it does.
Apple silicon mac, macOS Ventura required.

Brightness slider is locked
dynamic display features are off
truetone, nightshift, auto brightness.
# Reference color

Like macOS presents, reference mode uses color management to support varoius formats.
Provides accurate color rendering
requires media to be tagged

Should only be used in reference environment:
dim ambient lighting for video formats (10-1 lux)
bright for graphics (200 lux)

color management
CIE standard illuminant D65 whitepoint
SDR 100nits
HDR 1000 nits
HDR tone-mapping disabled
Colors displays correctly within iPad P3 color gamut
Coors exceeding gamut or dynamic range are clipped.

supported formats

| source formats                | equivalent macOS  | H.273 color code points <br>(primaries - transfer function - matrix) |
| ----------------------------- | ----------------- | -------------------------------------------------------------------- |
| BT.709                        | HDTV video        | 1-1-1                                                                |
| BT.601 SMTPE-C                | NTSC video        | 6-1-6                                                                |
| BT.601 EBU                    | PAL & SECAM video | 5-1-5                                                                |
| sRGB (100-nit peak lum)       | Internet & web    | 1-13-1                                                               |
| HDR10 B.2100PQ                | HDR video         | 9-16-9                                                               |
| BT.2100 HLG, Dolby Vision 8.4 | HDR video         | 9-18-9                                                               |
| Dolby Vision Profile 5        | HDR video         | 2-2-2 (RPU metadata, h.273 cp2 refers to 'unspecified)               |

On iPad, we determine color management based on tagging, so no need to switch between presets.

If your app uses a format that isn't on this list, don't worry, it's rendered in a similar way to default.

Generally speaking, reference displays are calibrated.  Like macOS presets, you can fine-tune.  Supports adjusting white point and luminance.
* differences in measurement instruments
* color drift due to display aging

# UIScreen and Reference Mode

EDR is a rendering technology and pixel representation
Pixel representation is intended for SDR and HDR content.
EDR headroom = HDR peak / SDR peak, ex 10x = 1000 nits / 100 nits.
For SDR content, EDR always renders 0 to 1 values.
Any values greater than 1 are rendered properly up to current headroom
Brighter values get clipped.

headroom may not be high enough for rendering EDR content.
Use the current headroom to tonemap your content before displaying?
UIScreen now supports notification and query APIs for renferece mode state

Register to receive a notification whenever the status changes
Query for change in status
Supports query for changes in current or potential EDR headroom

`UIScreen.referenceDisplayModeStatusDidChangeNotification`
Use notification for querying status changes
* new status
* new potential EDR headroom
* use this notification for rendering decisions

enabled -> it's enabled
limited -> enabled, but temporarily can't be achieved, ex headroom is less than 10x
not enabled -> supported on display but not enabled by user
not supported -> not supported on the display

UIScrene headroom
* query potential headroom
* Returns maximum possible headroom of the display
* Apps might select to use SDR media if headroom does not exceed 1.0
Query current headroom
* use the return value for tone-mapping
* Content doe snot exceed the expected headroom and won't clip.

# recap
* reference mode overview
* reference color
* UIScreen and reference mode

[[Edit and Playback HDR Video with AVFoundation]]
[[Explore HDR rendering with EDR]]
[[Explore EDR on iOS]] -> especially relevant for EDR on iPad.
[[Display EDR content with Core Image, Metal, and SwiftUI]]
[[Display HDR video in EDR with AVFoundation and Metal]]


