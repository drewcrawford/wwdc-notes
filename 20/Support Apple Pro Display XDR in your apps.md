# Pro Display XDR: Technical features
* 32-inch lcd
* 6k retina
* P3 wide color
* 10-bit color
* 1600 nits peak
* 1000 nits fs
* reference modes
* factory calibrated

# Designed for pro workflows
proaps - dvanici resolve, affinity photo, adobe premiere pro, final cut pro , pixelmator, foundry nuke, etc.

# Understanding reference modes
## What are reference modes?
Configures the luminance, color, and gamma response of the display itself.  Display changes its own behavior to adapt to your workflow.
Allows for the simulation of ideal calibrated display devices.
More than ICC Display profiles.
Works in conjunction with color management.

## Built-in reference modes
Apple-designed modes
* Reflect characteristics of our displays
* Dynamic features such as True Tone, etc.

Standards-based modes
* Built for specific workflows
* Viewing conditions are important!!!

```markdown
| Color Primaries       | Rec.709                                                                        |
|-----------------------|--------------------------------------------------------------------------------|
| White Point           | D65                                                                            |
| Transfer Function     | BT.1886                                                                        |
| Peak SDR Luminance    | 100 nits                                                                       |
| Brightness Control    | Fixed at 100 nits                                                              |
| Automatic adjustments | None.  All controls are fixed for use in controlled viewing conditions (10lux) |
```
HDR reference mode - HDR Video (P3-ST 2084)
```markdown
| Color Primaries       | P3 (Wide Color)                                                   |
|-----------------------|-------------------------------------------------------------------|
| White Point           | D65                                                               |
| Transfer Function     | SDR: Gamma 2.20 (power-law curve)  HDR: Perceptual Quantizer (PQ) |
| Peak SDR Luminance    | 1000 nits (fullscreen)                                            |
| Brightness Control    | 100 nits                                                          |
| Automatic adjustments | fixed at 100 nits                                                 |
```
Reference mode: Pro Display XDR (P3-1600 nits)

```markdown
| Color Primaries       | P3 (Wide Color)                                                                                                                  |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------|
| White Point           | ~D65                                                                                                                             |
| Transfer Function     | SDR: Gamma 2.20 (power-law curve)  HDR: Perceptual Quantizer (PQ)                                                                |
| Peak HDR Luminance    | 1600 nits peak with 39% screen coverage (XDR)                                                                                    |
| Peak SDR Luminance    | Up to 500 nits, based on brightness control                                                                                      |
| Brightness Control    | User selectable, up to 500 nits                                                                                                  |
| Automatic adjustments | HDR tone mapping, screen brightness, black level and True Tone (whitepoint) are adjusted for ambient lighting conditions via ALS |
```

# Developing for Pro Display XDR
## Working in an SDR reference mode
SDR video -> Gamma -> Compositing (+UI) -> Mapping display to colorspace -> Pro display

## working in an HDR reference mode
SDR is like before with gamma, HDR does with PQ/HLG.

## What is EDR?
Extended Dynamic Range, represent SDR and HDR simultaneously.
SDR-referred working space.  e.g., SDR is 0-1, EDR is brighter above 1.0.
The range above (headroom) is reserved for specular highlights or emissive surfaces.

**Always tag your media content**
Apple platforms rely on tag-based color management.
Correct interpretation of media data
Assign ICC color profiles to your images
Tag your video surfaces with standard color descriptors

## Tag your video files with color descriptor
Quicktime player -> view -> show movie inspector
Color primaries: RGB to XYZ transform
Transfer function: R'G'B' (non-linear) to RGB (linear)
Y'CbCr matrix: Y'CbCr to R'G'B' matrix
[[Export HDR media in your app with AVFoundation]]
Not reproducing the table of color descriptors for reference modes.

## Use EDR paramters on Pro Display XDR
Potential headroom - maximum headroom possible in current display configuration
Can be different for different reference modes
Examples: HDR Video (10.0), HDTV Video (1.0)

Headroom - amount of headroom currently available
Less than or equal to the potential headroom.
Changes as brightness setting changes
Example - Pro Display XDR has peak of 1600 nits
If the brightness slider is 200 nits, headroom = 1600/200 = 8

Reference Headroom- amount of headroom available considered "reference"
Guarantee of accurate reproduction of luminance in reference headroom
Less than or equal to Headroom
Example: HDR Video (10.0), HDTV video (1.0)

Get these off `NSScreen`,
`potentialheadroom = screen.maximumPotentialExtended...`

## Evaluate your app's color performance
Maybe use a spectro-radiometer.

## Apple provided test pattersnw ith reference values
developer.apple.com/av-foundation

21-step SDR and HDR luminance ramp,
100% amplitude color primaries
100% amplitude color secondaries
Reference values: CIE 1931, Y,x,y
# Deploying Pro Display XDR
Factory calibrated, ready-to-go
Reference modes for most popular media creation workflow
Supports an array of customization options

## Customized reference modes
Available in macOS 10.15.4 and later
For supporting unique workflow needs

You can now customize these parameters to create a unique reference mode.
Create a customized reference mode

## Fine-tune calibration
Available in macOS 10.15.6 and later
Calibrate display to a house standard
Adjusts luminance and white point to match another monitor

# Measuring Pro Display XDR
Front-of-screen objective evluation process

## Set up your instrument and environment
Recommended instruments
* Photo Research PR788, PR745, PR740
* Colorimetry Research CR300

Configure instrument settings correctly
Dark ambient
Controlled room temperature (~25C)

## Perform measurement error analysys
Measured values: CIE 1931 Y,x,y
Target/reference values CIE 1931 Y,x,y
Color difference metrics: DeltaE ITP, DeltaE 2000, Delta PQ

Compute color difference metrics

## In-field recalibration
Re-calibrates your display to align with your instrument
Adjusts primaries, luminance, and gamma
Spectro-radiomater and Dark Environment

# Wrap up
Reference modes - much more than ICC display profiles
macOS color management works in sync with reference modes
Create customized reference mode for unique workflows
Evaluate your workflow using Apple Quciktime test patterns
Calibrate your display to align with in-house instrument
