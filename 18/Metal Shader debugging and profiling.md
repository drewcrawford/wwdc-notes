#metal 

# Metal frame debugger
Step through API calls
resource inspection
shader edit and reload
GPU counters
pipeline statistics
integrated into xcode

New this year, dependency viewer, geometry viewer, shader debugger, enhanced shader profiler

# Geometry viewer

Visualize the post-vertex transform data in 3D
Access to all vertex data
Per-draw call view

Can detect cases like visibly wrong geometry, out-of-frustrum drawing
Missing vertices, like nans / infinities.  These are surfaced as issues in the issue navigator

# Shader debugger
Math heavy code
Highly parallel

## Demo

## Detail views

Functions are grouped in the debugger
Can debug backwards as well

Access to other threads
 
 * vertex - primitive of the selected vertex
 * Fragment - rectangle arund the pixel
 * compute - threadgroup of the selected thread

Seeing variables in context can help you identify problem areas

### Understanding divergence

Mask helps you understand which threads executed same line of code
White appears to be "executed"

### Demo

# Shader Profiler
* GPU counters
* Pipeline statistics
* Shader profiler

Provides per-pipeline timing information

A11 will provide isntruction category cost breakdown *per line*
e.g. 
* ALU (float/half/complex)
* Memory - sample, load, store
* synchronization – wait memory, barriers, atomics

Visibility into inline function cost

## Demo
Dependent texture read – what can we do about this?

# Access to shader sources
Save shader sources into built metal thing.  Mostly for profiling or debugging workflows
`-MO` compiler option
xcode project, "Yes, include source code" for "Produce debugging information"

[[Metal Game Performance Optimization]]
