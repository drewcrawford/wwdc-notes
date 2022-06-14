#metal 

* graphics and compute
* dramatically erduced overhead
* efficient multithreading
* explicit shader compilation
* GPU developer tools

# metal 3
* fast resource loading
* fofline ocmpilationmetalfx upscaling
* mesh shaders

etc

## fast loading
* fast loading of many small requests key to high qualify visuals
* traditional strage APIs designed for large request

we can efficiently do small requests
explicit command queue model
loads directly to metal buffers/textures
synchronize between grpahics, compute, and file loading

[[Load resources faster with Metal 3]]

## Offline compilation
shader binary generation
* gpu-specific machine code generated @ runtime
* expensive
* results cached for efficiency
* generate shader binaries at project build time


[[Target and optimize GPU binaries with Metal 3]]

## MetalFX upscaling
High-performance upscaling and anti-aliasing
temporal OR spatial solutions

[[Boost performance with MetalFX Upscaling]]

## Mesh shaders
* no access to primitives
* No geometry amplification
* Requires compute for advanced techniques

Mesh shaders
* flexible two-stage geo pipeline
* access,expand,contract primitives
* Compute for geometry in the render pass
* Good for culling, LOD selection, and procedural geometry generation

[[Transform your geometry with Metal mesh shaders]]

## Optimized ray tracing
* faster building, intersection, and shading
* etc

* acceleration are faster
* culling moves to gpu
* direct access to primitive data on gpu?

[[Maximize your Metal ray tracing performance]]

## Accelerated ML
* GPU-accelerated training on Mac
* faster inference for graphics and image processing

TF
* up to 16x faster on mac studio m1 ultra
* acceleration for many new ops

PyTorch
[[Accelerate machine learnign with Metal]]

# Hardware support
|        | GPUs                                                                                | products                                                            |
| ------ | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| iOS    | A13 bionic or newer                                                                 | iPhone 11, 11pro,11promax,iPhone SE(2g)                             |
| iPadOS | a13 bionic or newer, M1                                                             | iPad (9th gen), iPad air (4th), iPad mini (6th), iPad pro (5th gen) |
| macOS  | Apple silicon, AMD vega, AMD 5000-series,6000-series, Intel UHD 630, Intel Iris Pro | MacBook 2017, MBP 2017,MBA 2018,imac 2017,iMac pro(2017),Mac mini(2018), Mac Studio (2022), Mac Pro (2019)                                                                    |

if `device.supportsFamily(.metal3)`

# Heading
New advanced dependency analysis
Acceleration structure debugging

# See also
[[Maximize your Metal ray tracing performance]]
[[Go bindless with Metal 3]][[Profile and optimize your game's memory]]
[[Bring your game to the Mac]]
[[Scale compute workloads across Apple GPUs]]
[[Load resources faster with Metal 3]]

# summary
* resource loading
* compilation
* fx upscaling
* mesh shaders
* optimized ray tracing
* accelerated ml
* tooling

developer.apple.com/Metal
https://developer.apple.com/documentation/metal
