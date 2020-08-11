#metal 

Used by professional content creators 

# pro app requirements
Large assets/data
heavy compute demand
real-time for creativity vs offline for full fidelity

# Optimizing for 8k video editing
## Current 8k proxy workflow
Raw camera footage -> transcode to 4k proxy
Apply video editing to proxy

You cannot colorgrade proxy data because it's not accurate!

## Video editing pipeline

Video toolbox codesample

`CVMetalTextureCache` -> Turns pixel buffer into a metal texture

Can write custom shaders, or use Metal Performance Shaders

## Managing large asset sizes

8k is 16 x larger than an HD frame, 270mb uncompressed
30fps -> 9gbps

Even with prores, 10min playback can be 1tb.


High VM page fault cost with large 8k allocations
Can impact performance at the start of video playback
Prewarm your buffers before playback starts, to make sure all the pages are resident.

Alllocate early to minimze allocation cost mid-workflow
Reuse memory by using buffer pools
Use MTLHeap for transient allocations

MTLHeap advantages
* Allocation is cheap
* Heap is made resident as a whole (not just-in-time which may cause playback stutter)
* uses memory more efficiently
* Resources may be aliased and memory may be reused

## Maintaining a predictable frame rate
`CVDisplayLink` Notifies you before every VBL
# Support for HDR
## Common traits of HDR images
Better contrast levels, more colors, increased brightness, 
## Apple's approach to HDR
EDR.  Use brightness headroom for highlights and shadows.

HDR pixels are scaled relative to SDR display brightness.

SDR range is always pixel values 0-1.  Note that the nits values this represents may change with the display.
HDR headroom is what is leftover.

The advantage of this model is that we can do this on any display where we can set brightness.  But we can do better on displays with better brightness.

## HDR rendering with metal
AVFoundation
Metal (CAMetalLayer and EDR API)

We recommend that you use Float 16 as a preferred pixel format for most of your HDR needs.

Note that `maximumExtendedDynamicRangeColorComponentValue` will change with ambient conditions.

Typically
1.  Sample in YUV
2.  Convert to RGB
3.  Linearize with PQ to RGB
4.  Scale by maxEDR
5.  Editing/grading
6.  Tone map

When you create a metal layer, you can attach an `EDRMetadata` object.  This tells metal how to map your pixels, I think to a final colorspace?  

## Best practices
Update content when display brightness changes!
Use `MTLPixelFormatRGBA16Float`
Select color space and trasnfer function that matches content
Bypass tone mapping if contents are already tone mapped

# Leveraging all platform resources
## Multi-threaded references

Just create multiple command buffers from the queue.
Define order upfront by calling `.enqueue()` on each buffer

Or we can use `.makeparallelRendercommandEncoder` and create variou sub-encoders.

## Use multiple channels
Blit, compute, render, etc.

Prefer using a completion handler rather than `waitUntilCompleted`, to avoid waiting to enqueue frame 2 until frame 1 is done.

## Multiple GPUs

* Alternate frames?
* Interleaved tiling
* Tile queue

`MTLSharedEvent`

Synchronize between
* Multiple GPUs
* CPU and GPU
* cross-process

Idea here is we use signal values to store the **frame number**.    Then that frame is ready to start stage specified by the `SharedEvent`

# efficient data transfers

## Bandwidth and Mac Pro configurations
* Thunderbolt 3 -> 0.25x
* PCIe gen3 x16 ->1x
* dual pcie x16 -> 2x
* Infinity fabric link -> 5x


## Transfer strategies with Infinity Fabric LInk
* Transfer entire frames.
* Transfer tiles


Infinity fabric link avoids the need to roundtrip through syste memory, which improves memory bandwidth.
## Unlock challenging workflows