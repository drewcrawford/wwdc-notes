#metal 

# Metal 2 Developer Tools
## Frame debugger
Up to 10x faster

Argument buffers.

## Improved capture workflow
New lightweight capture API
Capture the exact workload you're interested in
Group workload into logical scopes
Trigger capture programmatically

## Quicklooks
View textures, buffers, and samplers inline
Use when the frame debugger is too invasive
Good for debugging resource loading and setup code

## Advanced filtering
Datamining, autocomplete suggestions, compound terms

## Pixel inspection
## Inspect vertex attribute outputs
Inspect output of vertex shaders, shown inline with vertex attributes inputs

## Demo
Quicklooks are fetched live from the GPU.

Long-press (or option?) has additional capture options??

Can filter to find individual textures, for I guess complicated command sequences.

Long-press on pixel loupe will instantly move.

# GPU Profiling
## Metal System Trace
Investigate timing issues
* CPU/GPU parallelism
* Frame rate stutters

Trace your Metal workloads through the system
* CPU to GPU to Display

Integrated into Instruments

## GPU Shader Profiler
Shader time per draw/pipeline
Per-line execution cost for iOS/tvOS

## Metal Pipeline Statistics

Compiler generated statistics

* Instruction count
* Mix (ratio of ALU to memory etc)
* register usage
* Occupancy

## Remarks
Avoid performance hits

## Demo

View frame by performance -> sorts by time

"Update shader" seems to do an A/B test of current and prior shader improvements.  So this is useful.

# GPU Counter Profiling
New tool for deep GPU profiling
Detailed GPU perf counters

We've defined a high-level set of counters that mean the same across each GPU, so you don't have a per-GPU learning curve.

## Graph view
Detailed GPU counter graphs
X-axis represents draw calls or encoders across time
Top-level counters for each pipeline stage
More detail as you drill down

## Detail view
All counters from counter graph view
Full detail
Frame median, max, total values

Both views are fully filterable

## Bottleneck analysis

Shared and GPU-specific analysis
Intuitive workflow to navigate direct

This basically is some kind of rules on the counters that try to point out problems

## Demo

# Additional Resources

[[Introducing Metal 2]]
[[VR with Metal 2]]
[[Using Metal 2 for Compute]]

