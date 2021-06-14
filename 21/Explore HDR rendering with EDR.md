# What is EDR?
#metal 
Extended Dynamic Range
HDR representation
HDR rendering technology

## Developing for EDR
* Interest in HDR on apple platforms
* XDR displays

Game developers
* Create realistic and exciting HDR games
* Internal and external HDR displays

Pro app developer
* Achieve reference response for HDR video, stills
* App developer who wants to support HDR content

## HDR Is here and enabled by eDR
* Standard, XDR, external HDR10
* Commercial contente, captured in ecosystem
* Games, pro apps, productivity

## EDR adapts to display and environment
Ever notice a display looks different outdoors?
* Dim, muted colors
* Low dynamic range

It's not the display that changed
* Displays the same light output
* Environment changed
* Your vision adapts to the environment
* In dark: vision adapts to the displayed content
* In bright: vision adapted to the environment

System adapts
* White point
* Black point
* Reference-white (brightness)
* EDR headroom (dynamic-range)

Mapping display to user's vision
EDR provides
* HDR on conventional displays (dim-environments)
* Optimized HDR capability on HDR displays (across environments)
* Shadow detail maintained across environments

* EDR headroom
* How many times brighter than SDR max can be rendered
* Varies with device and brightness setting


| Device | EDR max |
|-|-|
| Conventional backlit mac, ipad | 2x SDR |
| iPhone with Super Retina XDR | Up to 8x SDR |
| iPad Pro Liquid Retina XDR | Up to 16x SDR |
| Typical HDR10 display | 5x SDR (1000 nit-peak display) |
| Pro Display XDR | Up to 400X SDR (XDR present) |

## HDR content now mainstream
Commercial HDR video
* dolby vision
* youtube
* netflix

iPhone capture HDR Photos/video
ProRAW, Dolby Vision

Game adoption
* Bauldur's Gate 3
* Divinity: Original Sin 2
* Shadow of the Tomb Raider

* Apple tV
* Netflix
* YouTube

Pro apps
* Affinity Photo
* DaVinci REsolve
* Cinema 4D
* FCP
* Nuke
* Pixelmator Pro

Differentiate name from HDR. 

* HDR displays (bright with good blacks)
* HDR formats (HDR10, dolby vision)
* HDR transfer functions (PQ,HLG)
* Painterly tone maps (HDR->SDR)

EDR is not a painterly HDR to SDR conversion preserving details while losing intention

EDR maps reference-white in content to reference-white in display and perception

Luxo Double Checker OpenEXR on ProDisplay XDR presents
* Display at full 500 nits reference -w hite brightness
* EDR 0.0, 3.2 acurately represented

etc.

Lowering brightness setting (dark environment), EDR up to 400 become renderable.  

Let's explore the implications.

## EDR: HDR representation
Floating point extended-range representation
* 0 black, amx float damagingly bright
* Future-proof
	* Exceeds human vision
	* Complete human color gamut

## EDR: HDR rendering pipeline
An extension to color management.  
Able to mix HDR and SDR content at same time
Conceptually similar to mixing normal and wide-gamut content

Continue to use SDR assets.
## EDR represetnation
* float point
* 0 represents black
* 1 represents white (SDR max)
* 0-1 is SDR range (never clip)
* Values > 1 represent HDR values (very bright, not all will render)

## EDR displayed
* 0 EDR drives 0 nits
* 1 controleld by brightness slider (typically 400-500 nits, say it's 500)
* Peak white is always ex 1600 nits
* Thus EDR max is 3.2 EDR (1600 nit / 500 nit)
* Correspondingly we say there is 3.2x EDR headroom.

EDR 0 to 3.2 -> displayed as ex 0 to 1600 nits.
EDR 1.0 -> displayed as ex 500 nits

Finally, EDR values > EDR max (ex 3.2)
clipped to peak white (ex 1600 nits)


# Add EDR to your existing app
4 steps to adding support

## 4 steps
1.  Request EDR on object
2.  Select extended-range colorspace
	1.  Note that if there is some implicit conversion between 2 different colorspaces, some color management might kick in, so be on guard for that.
3.  Select pixel representation that can encode values beyond SDR
4.  Generate pixels that exceed 0-1 range.

## Simplest path to adding EDR support to existing apps
Substitute HDR still or video content
Follow the four opt-in steps
Enable EDR when HDR content is encountered?

## AVFoundation supports HDR content
dolby vision, hdr10, hlg
Automatically rendred as EDR (except watchOS)

## AVPlayer automatically uses EDR

```objc
// Instantiate AVPlayer with HDR Video Content

VPLayer*      player      = [AVPLayer playerWithURL:HDRVideoURL];
AVPlayerLayer* playerLayer = [AVPlayerLayer playerLayerWithPlayer:player];

// Play HDR Video via EDR

AVPlayerViewController* controller = [[AVPlayerViewController alloc] init];

controller.player = player;

[player play];
```

 Will enable/disable EDR based on content type.
 
 ## HDR still image rendering
 
 ImageIO supports various HDR formats, and conveniently return fp16 or fp32.
 
 Rendered as EDR on macOS
 * CAMetalLayer
 * CAOpenGLLayer
 * NSOpenGLView


# using EDR native API
Most flexibility for games, pro apps
* render arbitrary pxiels
* Query EDR headroom

Presently available on macOS supporting
* CAMetalLayer
* NSOpenGLView

## CAMetalLayer

1.  Opt into EDR
2.  colorspace
3.  fp16 pixel buffer

```objc
// Opt-in to EDR

metalLayer.wantsExtendedDynamicRangeContent = YES;

// Set extended-range colorspace

metalLayer.colorspace =
              CGColorSpaceCreateWithName(kCGColorSpaceExtendedLinearDisplayP3);

// Select FP16 pixel buffer format

metalLayer.pixelFormat = MTLPixelFormatRGBA16Float;
```

1.  create CGImage from HDR Image
2.  Draw into fp bitmap
3.  Create fp texture
4.  Load EDR bitmap into texture
5.  Draw the texture in your EDR enabled metal pipeline

```objc
// Create CGImage from HDR Image

CGImageSourceRef isr = CGImageSourceCreateWithURL((CFURLRef)HDRimageURL, NULL);
CGImageRef       img = CGImageSourceCreateImageAtIndex(isr, 0, NULL);

// Draw into floating point bitmap context

size_t width  = CGImageGetWidth(img);
size_t height = CGImageGetHeight(img);

CGBitmapInfo info = kCGBitmapByteOrder16Host | kCGImageAlphaPremultipliedLast |
                    kCGBitmapFloatComponents;

CGContextRef ctx = CGBitmapContextCreate(NULL, width, height, 16, 0,
                                         metalLayer.colorspace, info);

CGContextDrawImage(ctx, CGRectMake(0, 0, width, height), img);

// Create floating point texture

MTLTextureDescriptor* desc = [[MTLTextureDescriptor alloc] init];

desc.pixelFormat = MTLPixelFormatRGBA16Float;
desc.textureType = MTLTextureType2D;

id<MTLTexture> tex = [metalLayer.device newTextureWithDescriptor:desc];

// Load EDR bitmap into texture

const void* data = CGBitmapContextGetData(ctx);

[tex replaceRegion:MTLRegionMake2D(0, 0, width, height)
       mipmapLevel:0
         withBytes:data
       bytesPerRow:CGBitmapContextGetBytesPerRow(ctx)];

// Draw with the texture in your EDR enabled metal pipeline
```

Still supporting OpenGL?  

## OpenGL

1.  Opt-in to EDR
2.  Select OpenGL float pixel buffer fmt
3.  Draw EDR tontent into NSOpenGLView

```objc
// Opt-in to EDR

  - (void) viewWillMoveToWindow:(nullable NSWindow *)newWindow {
     self.wantsExtendedDynamicRangeOpenGLSurface = YES;
  }

// Select OpenGL float pixel buffer format

  NSOpenGLPixelFormatAttribute attribs[] = {
    NSOpenGLPFADoubleBuffer,
    NSOpenGLPFAMultiSample,
    NSOpenGLPFAColorFloat, //here we go
    NSOpenGLPFAColorSize, 64,  //and here
    NSOpenGLPFAOpenGLProfile, NSOpenGLProfileVersion4_1Core,
    0};

  NSOpenGLPixelFormat* pf = [[NSOpenGLPixelFormat alloc] initWithAttributes:attribs];

// Draw EDR content into NSOpenGLView
```
# EDR best practices
## Generating extended-range version of `CGColorSpace`.

Default `CGColorSpace` associated with display is not extended-range.

Use the following to 'promote' to XR.

```objc
// Get existing colorspace from the window 

CGColorSpaceRef color_space = [view.window.colorSpace CGColorSpace];

// Promote the colorspace to extended-range

CGColorSpaceRef color_space_extended = CGColorSpaceCreateExtended(color_space);

// Apply the extended-range colorspace to your app

NSColorSpace* extended_ns_color_space
                   = [[NSColorSpace alloc] initWithCGColorSpace:color_space_extended];

view.window.colorSpace = extended_ns_color_space;

CGColorSpaceRelease(color_space_extended);
```

## Best practices for content
Beware of glowing bunny syndrome
Content has an eerie, iridescent glow

* Don't 'scale' SDR content to make it HDR
	* EDR Emissive surfaces, specular content, etc.
* If most pixels > 1.0 EDR, then
	* You have created a glowing bunny
	* Viewers' vision will adapt and it stops being HDR

Subject
* less than reference-white 1.0 EDR
* Most UI shouldn't exceed.  
	* Except: EDR color pickers, scaled UI
* Emissive surfaces and specular highlights are potentially brighter.  
* SDR content
	* Already appropriately encoded as EDR (0,1)

## Ingest: converting HDR formats to EDR
* ImageIO does this automatically
* HEIC HLG (0,12)
* OpenEXR (0, Max float)
* 1.0 EDR representing the source's reference-white
* Other content requires a linearlizatioin and normalization based on content's reference white
	* e.g. HDR10 (PQ10)
	
Steps to convert PQ to EDR
* Decode PQ to Linear-light by applying Electro Optical Transfer Function
* Divide Linear-light pixels by 100 nits reference-white
* Reference-white -> 1.0 EDR
* PQ 10,000 nit max -> 100.0 EDR

EDR will clip values above reference white.  Usually ok to be clipped, however clipping of bright details is not always acceptable.

e.g. consider the subject.  If this number was a crucial plot element, than dynamic tonemapping may be employed to manage clipping.

## EDR and dynamic tone-mapping

Subscribe to `NSScreen` notifications to
* adapt rendering brightness of objects
* adapt exposure (selection of reference white)
* apply bloom
* soft-clip

`maximumExtendedDynamicRangeColorComponentValue`
Current max renderable linear EDR value
EDR values > max will clip to EDR max.
May change over time due to brightness, true tone, etc.
Interestingly my mac always returns 1 here, I checked nightshift and truetone with no effect.  Also brightness.  The "potential" EDR is 2.0.

static values
`maximumPotentialExtendedDynamicRangeColorComponentValue`, max for the display (if the display brightness were set appropriately)
Useful for selecting content or modes
Current actual max representable pixel might be below this value
1.0 for displays not supporting EDR

maximumReferenceExtendedDYnamicRangeColorComponentValue
* Brightest pixel value renderable without distortion
* interesting for reference standards
* 0.0 for displays not supporting reference rendering

## EDR display change notifications
```objc
// Read static values

NSScreen* screen = window.screen;

double maxPotentialEDR = screen.maximumPotentialExtendedDynamicRangeColorComponentValue;
double maxReferenceEDR = screen.maximumReferenceExtendedDynamicRangeColorComponentValue;

// Register for dynamic EDR notifications

NSNotificationCenter* notification = [NSNotificationCenter defaultCenter];  

[notification addObserver:self
                 selector:@selector(screenChangedEvent:)
                     name:NSApplicationDidChangeScreenParametersNotification
                   object:nil];

// Query for latest values

- (void)screenChangedEvent:(NSNotification *)notification {  
    double maxEDR = screen.maximumExtendedDynamicRangeColorComponentValue;
}
```

to complete our discussion, explore CAMetalLayer tone mapper.

## CAMetalLayer tone mapper

System video apps
* QT player, appletv, final cut
* Provide media OOTF (optical to optical transfer function)
* Soft clip HDR values not representable at current EDR max
* Opt in via `CAMetalLayer` `CAEDRMetadata` attribute
* available on macOS

```objc
// HLG

CAEDRMetadata* edrMetaData = [CAEDRMetadata HLGMetadata];

// HDR10

CAEDRMetadata* edrMetaData
   = [CAEDRMetadata HDR10MetadataWithMinLuminance:minLuminance
                                     maxLuminance:maxContentMasteringDisplayBrightness 
                               opticalOutputScale:outputScale]; 

// Set on CAMetalLayer

metalLayer.EDRMetadata = edrMetaData;
```

Min/max luminance
Optical output scale

Closely related, an app may want to draw the brightest white currently renderable.

EDR headroom is a linear value and pixels are most often non-linearly encoded.

## Convert EDR max into non-linear colorspace

Computing your app's brightest pixel

```objc
// Create the linear pixel we want to render

double EDRmaxComponents[4] = {EDRmax, EDRmax, EDRmax, 1.0};

CGColorSpaceRef linearColorSpace =
                   CGColorSpaceCreateWithName(kCGColorSpaceExtendedLinearDisplayP3);

CGColorRef EDRmaxColorLinear = CGColorCreate(linearColorSpace, EDRmaxComponents);

// Convert from linear to application’s colorspace

CGColorSpaceRef winColorSpace = [self.window.colorSpace CGColorSpace];

CGColorRef EDRmaxColor = CGColorCreateCopyByMatchingToColorSpace(winColorSpace,
                                                                 kCGRenderingIntentDefault,
                                                                 EDRmaxColorLinear,
                                                                 NULL);
```

# managing power/performance
* EDR power/performance implications
* depends on architecture and display technology
* Potentially more power used in display
* More bandwidth, memory footprint
* Previous 10bpc - 4 bytes per pixel
* fp16 - 8 bytes per pixel
* `CAEDRMetadata` causes extra processing pass

Use EDR judiciously

Selectively enable and disable EDR
* enable for viewing HDR content, disable for SDR
* Enable for duration of HDR UI element (for example, higlight)
* Open or stream HDR version of content
* When EDR potential headroom is significantly greater than 1

# Wrap up
* EDR is used on macOS, iOS, iPadOS, and tvOs devices
* An extension to color management and SDR representation

[[Edit and Playback HDR Video with AVFoundation]]
[[Support Apple Pro Display XDR in your apps]]
[[Metal enhancements for A13 Bionic]]

