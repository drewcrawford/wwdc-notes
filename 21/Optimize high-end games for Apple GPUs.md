#metal 

We work with game developers.

How we optimize high-end games for apple GPUs.

# Optimization process
* Collaboraing with Larian Studios and 4A games
* opportunities to optimize
* Utilizing GPU tools

Identified some common scenarios for graphics performance.  

How to pinpoint problem areas?

1.  Measure, define performance targets
2.  analyze
3.  improve
4.  verify, compare before/after.  Performance targets met?

For these games, we use xcode metal debugger to give insights into performance and how framegraphs are structured.

Save GPU trace file and instruments tracefile before and after.

What to consider?
## checklist
* use xcode and instruments regularly
* Shader performance
* Memory bandwidth
* Workload overlap
* Resource dependencies
* Development workflows
* Redundant bindings

# Baldur's Gate 3
GPU frame capture.  Analyze frame graph.  Shows how render passes depend on results of previous passes.

Metal System Trace - GPU
and Game Performance - thread stalls and thermals

Timeline.  Expensive workload in render passes.  e.g., long compute shader holds up frame and creates a bubble.

Over 4k instructions is quite a lot!  
Up to 120 textures to produce output.  However, 6-12 are used 90% of the time.

## Improving complex shader performance
* Break complex shaders into shader permutations
* Use fewer registers due to simplified controlflow
* Simplify shaders to reduce register pressure
* Prefer 16-bit types if appropriate over 32-bit types to reduce register spilling

If you look at most expensive PSOs, you may find a complicated shader that could be simplified.  Especially true if shader is used by later pass.

## Compression

Lossless compression reduces bandwidth.  You may notice lossless compression warnings for some resources.  You may pay a bandwidth penalty.

One reason is "shader write" flag.  Can sort by usage with bottom-right corner in memory view.

## Verify your texture usage flags
* Unknown
* shaderwrite
* pixelformatview

Use only when required.  Prevent compression.  Rendering to a texture bound as color attachment doesn't require shader flag.

Don't use pixel view when only reading component values in different order.  Instead create a texture view using a swizzle pattern.

Don't set pixel format view options if your view only converts between linear and srgb.  Check docs.

## overlap
How to overlap between vtx, frag,kernel?

We can think fo our frame graph as a list of rendering tasks.  Getting good overlap can be changing the order.

Another optimization.  Games often have 2-3 frames in flight.  So can overlap 2 frames when there are no resource dependencies.

To efficiently update constnat buffer data with a discrete GPU, we blit shared buffers on the CPU to a private buffer on GPU which will be used per frame.  This strategy is efficient for discrete memeory, so keep it around for tha tpurposes.

But in UMA, no need to use a blit encoder.  However, when you use a shared buffer in a ring buffer, need to watch out for sync issues if your CPU writes to data being read by GPU.

### Searching for data race conditions

Very common to have latency between encoding and rendering.  This causes a shift.  

However, what happens if latency keeps increasing?  Can create a data race.

Use a completion handler for synchronization.  Wait until it is safe to update the shared buffer in the encoding thread.

But how else?

We added an extra buffer to the ring buffer to avoid the wait.  The memory consumption remains the same... ? Use 3 buffers.

How many shared and private buffers to create?

```cpp
// Number of frames application is processing
static const uint32_t MAX_FRAMES_IN_FLIGHT = 3;

uint32_t sharedBuffersCount  = 0; // Number of buffers with MTLStorageModeShared to create
uint32_t privateBuffersCount = 0; // Number of buffers with MTLStorageModePrivate to create
if (device.hasUnifiedMemory)
{     // Use extra buffer to reduce impact of completion handler, which we are using to avoid data race
    sharedBuffersCount = MAX_FRAMES_IN_FLIGHT + 1;
    privateBuffersCount = 0;
}
else // GPUs with dedicated memory
{
    sharedBuffersCount = MAX_FRAMES_IN_FLIGHT;
    privateBuffersCount = 1;
}

// Create shared buffers MTLStorageModeShared
// If applicable, create private buffer
```
1-2ms, depending on scene.  Can be applied for all buffer data you transfer from CPU to GPU.

## Review
* Optimize shaders
* Opt into lossless compression
* Overlap vtx and frag workloads
* Check deps within and across frames

33% improvement in frame time

# Metro Exodus
Translation layer is optimized for metal but some issues can arise.

Start by investigating framegraph in gpu trace.  
More registers, less work can be done in parallel.

Sometimes it's justa c omplex shader, but sometimes compiler settings can affect.

Turns out this shader was compiled without fast math.  That allows the compiler to optimize fast math.  Some workflows can disable this.  We discovered that the translation layer had its default behavior set to not use fast math.

## what is fast math?
* Trade between speed and correctness
* No NANs
* No Infs
* No signed zeros
* Allows algebraically equivalent transformations
* Check if you need IEEE 754 conformance

Check your compiler options to verify that you have this on!

Works at frontend *and* backend.  Hints to backend that it can generate more optimal GPU machinecode.

See how the instructions and registercounters have been improved.  After changing the bheavior, 21% frame time decrease.

## Redundant bindings
Returning to the summary, we can see many redundant bindings rendering the frame.  Either resources like textures, buffers, samplers, or render states like depth stencil stat,e etc.

Repeatedly binding resources might affect encoding time.  

For a given frame, it takes 8.5ms to encode.  22ms for GPU to render.

Cause of redundant bindings?  Translation layer.

Instead of binding unconditionally, check if the binding is needed.

```cpp
void Renderer::SetFragmentTexture(uint32_t index, id<MTLTexture> texture)
{
    if (m_FragmentTextures[index] != texture)
    {
        m_FragmentTextures[index] = texture;
        m_FragmentTexturesChanged = true;
    }
}

void Renderer::BindFragmentTextures()
{
    if (m_FragmentTexturesChanged)
    {
        [m_RenderCommandEncoder setFragmentTextures:m_FragmentTextures 
                          withRange:NSMakeRange(0, m_LastFragmentTexture + 1)];

        m_FragmentTexturesChanged = false;
    }
}
```

Set all the textures with one call to `setFragmentTextures` instead of in a 1x1 loop.

Apply a similar approach to buffers, samplers, etc.

Reduce encoding times between 30-50% by not binding same resource nad render states.  However, GPU time also decreased.

15% speedup in benchmark.  If you have a few redundant binding warnings it's not an issue, but 100s or 1k are an issue.

## summarize
* Check shader compiler settings
* Reduce redundant bindings



# Divinity: Original Sin 2
Interestingly, this game requires A12 and is selling for $25.  Currently ranked at 56.

# New GPU Timeline in Xcode 13

Each GPU pipeline stage can run in parallel on apple gpu

Keep all stages as busy as possible by maximizing overlap.

Composed of 2 sections
* GPU section, separate tracks for each pipeline stage
* Counters section, curated set of important counters, such as occupancy, bandwidth, ALU, etc.

Rendering encoder brings up sidebar with additional information (inspector)
Since render encoders have 2 shader stages...

If we select fragment track, sidebar contains all encoders.

Expand fragment track see shader timeline.  All shaders used by the encoders during execution.  Identify long-running shaders, etc.

Fragment track, 2 additional tracks.  See when GPU is loading and storing textures between memory, reduce bandwidth usage.

Inspector panel has various stats.  Flow of workload, order of execution.  

How to find performance bottlenecks?

Register pressure.  GPU runs out of fast register memory and has to use main memory instead.  High ALU limiter does not indicate bottleneck, might just be math heavy.  However, when combined with occupancy, a shader may be experiencing register pressure.

Pin tracks to timeline with "plus" button.
Counters update based on selected region.
Compare F16 vs F32
Right click and open shader.
Simplified version of shader source for demo purposes.  Can also see per-line costs.  

Candidates for using fp16, which gives us double the registers.  Helps to reduce register pressure.  Makes it convenient to update sourcecode inside the source editor.

Use an updated version of the shader with a mixture of f32 and f16.

Reload shaders to trigger a shader update that recompiles and reprofiles the shader.  fp16 improves by 30%.

# wrap up
Many ways to improve performance.
MST and GPU timeline will help you improve games

[[Discover Metal debugging, profiling, and asset creation tools]]
[[Optimize Metal apps and games with GPU counters]]
