#metal 
# Metal counters API review
New in iOS 14 and available in macos catalina

On iOS/macOS with apple silicon, you get "stage boundary timings" – precise vertex, fragment, and compute start/end times.

On intel embdded GPUs and AMD discrete GPUs, you get "draw boundary timings" – precise GPU timestamps for any draw command.

# Live profiling HUD recap
* used to trac your app perfomrance at runtime
* Help find problem areas that need in-depth investigations with the GPU tools
* Help tune settings for each device.

## CPU markers
Used to instrument your important functions
start/end timestamps with `mach_absolute_time`
A good start is the command buffer work
* start marker when creating a new command buffer
* end marker when committing the command buffer

## GPU markers
You may be familiar with gpu events in metal system trace timeline.

"stage boundary" - vertex vs fragment
"draw boundary" - obj1 vs obj2

# Using the API step by step
* check for iOS 14 and macOS big sur
* Query which sampling modes are available
	* stage and tile boundaries
	* draw/dispatch/blit boundaries

`supportscounterSampling:MTLCounterSamplingPointAtStageBoundary` etc.

```objc
if (@available(macOS 11.0, iOS 14.0, *))
{
    _supportsStageBoundary = [_device supportsCounterSampling:MTLCounterSamplingPointAtStageBoundary];
    _supportsDrawBoundary  = [_device supportsCounterSampling:MTLCounterSamplingPointAtDrawBoundary];
}
```

Choose a counter set.
* absolute timestamps
* stage utilization
* pipeline statistics.

`device.bountersets enumerateObjects...` 

```objc
[_device.counterSets enumerateObjectsUsingBlock:^(id<MTLCounterSet> nonnull obj,
                                                  NSUInteger                idx,
                                                  BOOL * nonnull            stop) {
       if ([[obj name] isEqualToString:MTLCommonCounterSetTimestamp])
            _counterSetTimestamp = obj;
}];
```
Note that some devices do not support timestamps.

1.  Create a sample buffer
	1.  number of samples, storage mode, counterset
	2.  specify index of each sample in the buffer
2.  use it in the pass descriptor.  Note that this means you need at least 1 buffer per pass.
3.  Insert sampling commands at key points.
4.  Resolve the counters.
	1.  Align CPU/GPU timestamps if needed.

```objc
// When setting up the render pass descriptor

if (_supportsStageBoundary || _supportsDrawBoundary)
{
    MTLCounterSampleBufferDescriptor *desc = [MTLCounterSampleBufferDescriptor new];

    desc.sampleCount = 6; // Number of samples to store 
    desc.storageMode = MTLStorageModeShared;
    desc.label       = @"Live Profiling HUD Metal counter sample buffer";
    desc.counterSet  = _counterSetTimestamp;

    id<MTLCounterSampleBuffer> sampleBuffer =
                               [_device newCounterSampleBufferWithDescriptor:desc error:nil];

    MTLRenderPassSampleBufferAttachmentDescriptor *sampleBufferDesc =
                                  renderPassDescriptor.sampleBufferAttachments[0];

    if (_supportsStageBoundary)
    {
        sampleBufferDesc.startOfVertexSampleIndex   = 0;
        sampleBufferDesc.endOfVertexSampleIndex     = 1;
        sampleBufferDesc.startOfFragmentSampleIndex = 2;
        sampleBufferDesc.endOfFragmentSampleIndex   = 3;
    }

    sampleBufferDesc.sampleBuffer = sampleBuffer;
}
```

### sample at draw boundary
```objc
// After creating a new render command encoder
[renderCommandEncoder sampleCountersInBuffer:sampleBuffer
                               atSampleIndex:4
                                 withBarrier:NO];

// All draw calls
[renderCommandEncoder sampleCountersInBuffer:sampleBuffer
                               atSampleIndex:5
                                 withBarrier:NO];

// End encoding
```

Note that in this example, we chose a sample buffer of size that would support both draw and stage boundaries.  However, you could optimize this to support only 1 or the other, since they are mutually exclusive.

### collecting timestamps
```objc
// For each tracked sampleBuffer, resolve the counters
NSData *data = [sampleBuffer resolveCounterRange:NSMakeRange(0, 6)];

MTLCounterResultTimestamp *sample = (MTLCounterResultTimestamp *)[data bytes];

// And simply access the timestamps
if (_supportsStageBoundary)
{
    double vertexStart = sample[0].timestamp / (double)NSEC_PER_SEC;
}

// Check for errors
if (sample[0].timestamp == MTLCounterErrorValue)  //GPU uses a predefined error value if it can't get a timestamps
{
  // Handle error
}
```

### aligning timestamps
On apple GPUs - timestamps are aligned to `mach_absolute_time`.
So you can directly compare to CPU timestamps.
On intel/amd, timestamps are in an arbitrary domain.  The rate of time varies with GPU clock frequency, which may also vary over time.

```objc
// On immediate mode GPU
MTLTimestamp cpuTimestamp;
MTLTimestamp gpuTimestamp;
[_device sampleTimestamps:&cpuTimestamp gpuTimestamp:&gpuTimestamp];

// Do a linear interpolation between correlated timestamps
gpu_ns = cpu_t0 + (cpu_t1 - cpu_t0) * (gpu_timestamp - gpu_t0) / (gpu_t1 - gpu_t0);
```

Basically, the idea here is to get timestamps once per frame, and assume everything in the frame has the same scale.  Then with known cpu timestamps, you can work out the gpu via a linear interpolation.

# Putting it all together
Draw your own hud.  `addPresentedHandler:`.

## gotchas
* Don't update the HUD too often.  Live data is hard to read if constantly changing.  Only update the screen once per second, which makes it easier to follow.
* GPU activity depends on GPU clock rate.  High occupancy does not necessarily mean it's maxed out.  The system only clocks as high as needed.  GPU might be 80% occupied in the hud, but if it's running at half max clock, only 40% used.
* Handle errors and watch for inconsistencies.
	* An overflow may occur and cause a "negative duration"
	* Waking from sleep can cause a large duration

# Wrap up
* metal counters API
* `MTLDevice supportscounterSampling`
* Counter sets – MTLCommonCounterSetTimestamp
* new counter sample buffer
* render command descriptor
* sampleCountersInBuffer
* resolveCounterRange
* Aligned if needed

## next steps
* stage utilization
	* summarized data per stage
* statistics
	* number of vtx/fragment invocations
	* clipped primitve counts and more

