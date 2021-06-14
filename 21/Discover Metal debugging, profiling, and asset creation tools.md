#metal 
# Ray tracing support
Last year, we introduced new metal raytracing API.  Now supported in metal debugger
Function pointers and function tables
Dynamic libraries
Acceleration structure viewer

## Raytracing in metal debugger
Beautiful raytraced shadow map.  Can open acceleration structure.

Acceleration structure can do mroe than display gemoetry.  Bottom right, various visualizations.

[[Explore hybrid rendering with Metal ray tracing]]
[[Discover ray tracing with metal]]
# New profiling workflows
* Metal system trace
* GPU Counters

Aligning these views may take additional effort.  So we added GPU Timeline.  Specifically for apple GPUs.  Different perspective on the performance data.

Under Performance panel in the debug navigator.  When you open Performance, you see the timeline tracks.

Why are these in parallel?  IN apple GPUs, vtx and frag of different passes will overlap
compute dispatches will overlap if no dependencies.

Expand each track.  Can see each shader.  Select a track to see encoders.  Clicking on an individual encoder will show more info in the sidebar.

When you select an encoder, timeranges where it's active become highlighted.  With this, you can easily examine how different stages overlap and correlate counter values.

Moving away from timeline view, can access GPU counters.  Open encoders context menu.

To learn more, 

[[Optimize high-end games for Apple GPUs]]

## GPU performance state
Managed by operating system
Depends on
* Thermals
* System settings
* GPU utilization

These can affect profiling results!  To get consistent results,

* Instruments
* Metal Debugger
* Device conditions

### GPU performance state
New track in Metal System Trace 

Keep in mind that seeing the state is just part of the equiation.  To get consistent results, you also need a way to set data.  New this year, you can induce a specific state when recording a trace.

Do this inside recording options.  Now you can record the perf state as usual.  

Soemtimes, you might need to check if an existing trace had a performance state reduced.  Can find in recording settings, information popover.

### Metal debugger performance state

By default, when you capture a GPU trace, xcode profiles the trace.  It will do so using the same performance state the device was in.

If you would like to select a performance state yourself, use the "stopwatch" button in the debug bar.  Now it will profile again.

These approaches are tied to suite of metal tools.

### Device conditions performance state

In xcode 13, we've added GPU performance state device conditions.  This forces OS to use specific state.

Window->Devices and simulators->Device->Device condntions->GPU performance state, start.

Help you with profiling and testing your apps.  I think you're going to love our latest additions.

# Debugging improvements
## Shader validation

Last year, we added this.

* Out of bounds access
* nil resource access,
* non-resident resources
* timeouts

If shader validation is enabled an an encoder raises a validation error, you'll get a runtime issue.

[[Debug GPU-side errors in Metal]]

More usecases!

* Indirect command buffers
* Dynamic libraries
* Function pointers and tables

Allow you to use shader validation more extensively


## Precise capture controls
New capture button in debug bar.  When clicked, new menu appears.  

Capture a number of buffers, capture scopes, layers, queues, etc.  Incredible power in deciding how and when to capture.


## New pipeline state workflows

Essential building blocks of metal.  In xc13, we've made it easier than ever to do all the pipeline states.

I can now see pipeline state object in metal debugger.  Examine functions, see other properties, etc.  Check performance data, or go to memory viewer and reveal state.

In xc13, memory veiwer now shows how much memory, etc.  Just some of the additions that make it easier to understand pipleine states.

## Separate debug information
Right now, to use shader debugging

1.  Compile your libraries from sourcecode when the app is running
2.  Build metallib with sources embedded, and load at runtime.

Appstore rules don't allow you to ...

If you compile your libraries offline, and want to debug your shaders, you have to compile them twice.
1.  Once during development
2.  Once without sources, for distribution

This year, you can now generate `metallibsym`.  These have a sym extension, allow you to debug shaders without additional information.

Most important benefit of having separate libs is you don't need to have 2 versions of the same metallib.  With these, you now are able to debug shaders even in release versions of the app.

```bash
$ xcrun -sdk iphoenos metal -frecord-sources=flat Shaders.metal
```

Now when I debug a shader, I'll be prompted to import the file.  Now can pick individual libraries.  Import any metallibsym file.  Libraries matched automatically.  Can close dialog and can now see shader sources and debug.


## Selective shader debugging

Large shaders slow down debugger startup
Selective debugging lets you narrow the debug scope
Fast ieteration times for better productivity.

If I try to debug this whole kernel, shader debugger "will take a long time to start" (lol)

Great way to pinpoint bugs in your shaders quickly.
# Advances in texture compression

Updates to texture compression tools.  

## Basics of texture compression
Fixed-rate lossy compression of texture data
Offline compression of static texture data, e.g. normals, etc.
Can compress at runtime, not discussed here
Block-based encoding

Per-pixel index selecting from palette.
Multiple formats to suit any kind of texture data
Lossless compression on some devices

[[Optimize Metal apps and games with GPU counters]]

Can also do lossless compression of *files* on top of GPU compression.  App download size.

## GPU texture compression benefits

* Most memory is textures
* Use more textures
* More detailed textures
* Reduce download size or memory footprint

## current situation
1.  Read input image
2.  generate mipmap if desired
3.  compress
4.  output texture

As algorithms increase in complexity, need more advanced processing.  

1.  colorspace
2.  rounding

New compression tool with new options.

## new situation
1.  Input image
2.  To linear gamma
3.  transform
4.  mipmap
5.  alpha handling
6.  to target gamma
7.  compress
8.  output texture

Trade off between speed and quality. 

Each stage is configurable.  Gamma-aware.  Available for macOS and *windows*.  Optimized for apple silicon.

`-gamma_in = VALUE / sRGB`

Best choice is the type of data.  Most visual data do best in non-linear space like sRGB. 

Non-visual data, like normal maps, should be in linear space.  

Compression should be performed *in your target color space*.  Specified with `-gamma_in` and `-gamma_out` value.  Either float value, or use a string.

Can use these to convert to a different space.  

Mipmap in linear space.

### Linear space processing

1.  Transform
	1.  Scale
	2.  Flip
2.  Mipmaps
3.  Alpha handling
	1.  Alpha to coverage
	2.  Premultiplied alpha



```
--max-extent=SIZE
--resize_filter=Box/Kaiser/Mitchell/Triangle
--resize_round_mode=None,NearestMultipleOfFour...
```

If you're unsure which to use, we picked defaults that work well in most cases.

Flip options `--flip_x, y,z`

Mipmaps are precalculated sequence of images that reduce in resolution.  Height/width is a power of 2 than the prior level.  When customizing mipmap, specify

`--max-mipmaps=VALUE --mipmap_filter =Box/Kaiser/Triangle`

Alpha handling
`--alpha_to_coverage --alpha_reference=VALUE`

Coverage: replaces alpha blending with a coverage mask.  When antialiasing... order-independent transparency, useful for rendering dense greenery.

`--alpha_mode=Ignore / Premultiply / Preserve`

Partly transparent pixels will be premultiplied with a matting color.

Move back to target gamma.  

Last step is compression.

### compression
1.  Channel mapping.  Optimize general-purpose algorithms.
2.  Encode


`--rgbm_encoding --rgbm_range=MAX_VALUE --normal_map`

#### RGBM encoding
Encode RGB HDR in LDR formats
Uses alpha channel to encode multiplier

```cpp
float4 EncodeRGBM(float3 in)
{ 
    float4 rgbm; 
    rgbm.a = max3(in.r, in.g, in.b) / RGBM_Range;
    rgbm.rgb = in / (rgbm.a * RGBM_Range);
    return rgbm;
}
```

rgbm_range defaults to `6.0`
input texture's red, green, blue channels.

```cpp
float3 DecodeRGBM(float4 sample)
{ 
    const float RGBM_Range = 6.0f;
    float scale = sample.a * RGBM_Range;
    return sample.rgb * scale;
}
```

#### normal map encoding
Object-space normal maps.  

Usually with normal maps, we know that each normal is a unit vector.  Can be represented in 2 axes, with the third axis trivial.  Remap 2 channels.

How you remap chanels varies depending on compression format.  ASTC example.


|       | R   | G | B   | A   |
|-------|-----|---|-----|-----|
| ASTC  | X   | X | X   | Y   |
| BC3   | N/A | Y | N/A | X   |
| BC5   | X   | Y | N/A | N/A |
| Other | X   | Y | Z   |     |

Texture converter takes care of encoding for you if you pass normal map paremeter.

When sampling, it's important to know the channel mapping.  If you use multiple formats, you'll have to handle this at runtime.

```objc
// Remap the X and Y channels to red and green channels for normal maps 
compressed with ASTC.
 
MTLTextureDescriptor* descriptor = [[MTLTextureDescriptor alloc] init];

MTLTextureSwizzle r = MTLTextureSwizzleRed;
MTLTextureSwizzle g = MTLTextureSwizzleAlpha;
MTLTextureSwizzle b = MTLTextureSwizzleZero;
MTLTextureSwizzle a = MTLTextureSwizzleZero;

descriptor.swizzle = MTLTextureSwizzleChannelsMake( r, g, b, a );
```

How to avoid shader variants?  Apply custom swizzles to the texture to remap so that shaders can be format-neutral.

```cpp
// Reconstruct z-axis from normal sample in shader code.

float3 ReconstructNormal(float2 sample)
{
    float3 normal;

    normal.xy = sample.xy * 2.0f - 1.0f;
    normal.z  = sqrt( saturate( 1.0f - dot( normal.xy, normal.xy ) ) );

    return normal;
}
```

### encoding

```bash
--compression_format=FORMAT
--compressor=Auto / ARM / ISPC / NVTT / PVRTC / STB
--compression_quality = Fastest / Highest / Normal / Production
```


| iOS | ASTC LDR (A8+) ASTC HDR (A13+) PVRTC |
|-|-|
| macOS (AMD/intel) | BCn |
| macOS(Apple Silicon) | BCn ASTC LDR ASTC HDR PVRTC |


#### BCn formats

BC1/3 are commonly used for rgb and rgba
bc6 - hdr images
bd5 - normal map encoding


|  |  |
|-|-|
| BC1 | 4bpp RGB + Alpha/Opaque |
| BC2 | 8bpp RGB+Explicit alpha |
| BC3 | 8bppp RGB + Interpolated alpha |
| BC4 | 4bppp single channel |
| BC5 | 8bppp dual independent channels |
| bc6 | 8bppp RGBA HDR |
| bc7 | 8bppp RGBA |


#### ASTC
Adaptively scalable texture compression
Family of RGBA formats in LDR, sRGB, and HDR color space

Recommended over pvrtc.
Range of bpp vs quality for each format

Highest quality, lowest compression rate : ASTC4x4 - 8bpp
Lowest quality, highest rate: ASTC 12x12 - 0.89bpp
LDR, SRGB, and HDR color spaces

#### PVRTC formats
* RGB(A) 2bpp or 4bpp
* Always occupies 8 bytes
	* 2bppp 1 block for 2x2?
	* 4b 1 block for 4x4

#### guildelines

On iOS, always use ASTC compression.  With PVRTC and per-device thinning only if you're A7/earlier.

For macOS, BCn is always available.  Apple silicon lets you use ASTC.  Consider this if also targeting iOS.  PVRTC is available on apple silicon, we don't recommend this option.

Select the compression format based on the type of data you're compressing.

Choosing a format with 2 independent channels.  When compressing ASTC, consider a subset of the formats.  Highest quality vs lower compression rate.

# Wrap up
* New texture pipeline.
* We recommend `TextureConverter` (new) over `TextureTool` (old).
* Compatibility mode will tell you what new options are.

* Ships in xcode 13
* available in seed 1
* Also ships in Metal Developer Tools for Windows

developer.apple.com/download

In windows, no support for PVRTC.  

## Metal compiler
* Introduced in 2020
* version 1.0 - support for MSL 2.3
* version 1.2 - added support for apple silicon
* 2.0 now available
	* MSL 2.4
	
# Wrap up for real now
* Ray tracing support
* New profiling workflows
* Debugging improvements
* Advances in texture compression


