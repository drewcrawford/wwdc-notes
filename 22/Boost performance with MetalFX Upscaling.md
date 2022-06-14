#metal 

# MetalFX
* provides paltform optimized graphics effects
* high-performance upscaling

# performance
high resolution vs upscaling

# Spatial upscaling
* analyze input spatial informationt o produce new samples
* simple to integrate

performs best if tone-mapped an in a perceptual color space.

MetalFX effect object can encode into the command buffer.
Only create aspatial scaler object when starting or switching resolutions because they're expensive.

1.  MTLFXSpatialScalerDescriptor


```swift
// Spatial upscaling (initialization)

let desc = MTLFXSpatialScalerDescriptor()
desc.inputWidth = 1280
desc.inputHeight = 720
desc.outputWidth = 2560
desc.outputHeight = 1440
desc.colorTextureFormat = .bgra8Unorm_srgb
desc.outputTextureFormat = .bgra8Unorm_srgb
desc.colorProcessingMode = .perceptual //which colorspace input and output is in

spatialScaler = desc.makeSpatialScaler(device: mtlDevice)
```

Can modify object after creation.
make sure textures are set correctly.

```swift
// Spatial upscaling (per frame)

// Encode Metal commands to draw game frame here...

// Begin setting per frame properties for effect
spatialScaler.colorTexture = currentFrameColor
spatialScaler.outputTexture = currentFrameUpscaledColor

// Encode scaling effect into command buffer
spatialScaler.encode(commandBuffer: cmdBuffer)

// Encode Metal commands for particle/noise effects and game UI drawing for frame here...
```
# Temporal AA and upscaling
Performs upscaling using data from previous frames.
Output will be used from prior frames.

In supersampling, multiple samples are taken per pixel which are integrated into a single pixel value.  More samples per pixel, better the result.  But this is expensive.

Instead of sampling multiple locations per pixel, you can perform temporal sampling.  Render different sample locations for all pixels in a given frame.  Supersampling over multiple frames at a significantly lower cost.

By accumulating samples oover multiple frames, integrate samples appropriately in target reoslution pixels reuslting in h igh quality aniti-aliased output.  Sicne content changes between frames, requires more input data.

* previous frame
	* all samples that have been integrated previously.  Blended with jittered input to increase the number of samples.
* jittered color
	* jitter offset is unique for a set number of frames.  
* motion
	* How much objects have moved from previous frame.  Backtrack and find corresponding locations in prior frame.
* depth
	* foreground vs background.  What other objects might be newly exposed.

Only pass color, motion, depth.  

Run before post-processing effects.

Creating a enw scaler is expensive, only when app starts or display changes resolutions.

```swift
let desc = MTLFXTemporalScalerDescriptor()
desc.inputWidth =1280
desc.inputHeight = 720
desc.outputWidth=2560
desc.outputHeight = 1449
desc.colorTextureFormat =.rgba16Float
desc.depthTextureFormat=.depth32Float
desc.motionTextureFormat=.rg16Float
desc.outputTextureFormat=.rqba16Float
temporalScaler = desc.makeTemporalScaler (device: mtlDevice)
temporalScaler.motionVectorScale=CGPoint(x:1280,y:720)
```

## motion data vector format
* render reoslution pixel coordinate
	* `.motionVectorScale` can scale the whole scene?
* motion vector direction from current to prior frame

```swift
// Temporal antialiasing and upscaling (per frame)

// Encode Metal commands to draw game frame here...

// Setup per frame effect properties
temporalScaler.resetHistory = firstFrameOrSceneCut
temporalScaler.colorTexture = currentFrameColor
temporalScaler.depthTexture = currentFrameDepth
temporalScaler.motionTexture = currentFrameMotion
temporalScaler.outputTexture = currentFrameUpscaledColor
temporalScaler.reversedDepth = reversedDepth
temporalScaler.jitterOffset = currentFrameJitterOffset

// Encode scaling effect into commandBuffer
temporalScaler.encode(commandBuffer: cmdBuffer)

// Encode Metal commands for post processing/game UI drawing for frame here...
```

## jitter offset convention
`-0.5` to `+0.5`

verifying jitter offset => check if things move?

# Best practices
* spatial upscaling
	* color input should be anti-aliased and noise-free.  Noise worsens edge detection.
	* Use perceptual color processing mode. SRGB 0-1.
	* Set appropriate negative mip bias for higher texture detail
		* log2(render resolution width / target reoslution width)
	
	ex

| scaling dimension factor | suggested mip bias |
| ------------------------- | ------------------------------------- |
|  2.0x                      |  -1.0      |
|  1.7x                      |  -0.79      |
|  1.5x                      |  -0.58      |

lower mip levels might result in flicker.  adjust mip bias for certain textures.

* temporal AA and upscaling
	* pick a good jitter sequence
	* aim to provide a good number of samples
	* 8 jittered samples per pixel output ideally
	* for 2x upscaling a halton 2,3 sequence with 32 jitters
	* set appropriate negative mip bias
		* log2(render resolution width / target reoslution width) - 1.0

ex
| scaling | mip bias |
| ------- | -------- |
| 2.0x    | -2       |
| 1.7     | -1.79    |
| 1.5x    | -1.58         |


# Demos
Lower mip levels generally result in more details, but may cause texture flickering if textures have high-frequency patterns.

# Perf best practice
* avoid false dependencies between frames
	* binding same resources for reading/writing
	* especially important to metalfx
	* 2 independent frames with various passes.  But if a stage in frame A writes and frame B reads, it's a problem


if your game can generate the required inputs, consider using temporal AA and upscaling.
otherwise, consider using metalFX spatial upscaling.
# Demos
