Discover how the power of A17 Pro can help you maximize your game on iPhone 15 Pro and iPhone 15 Pro Max. We'll share best practices and technical resources, and explore ways to optimize game performance, input, and asset management.

# Make a game plan

* rapid iteration in xcode and instruments
* Additional memory and other resources
* Rapid input, testing, and automation bring-up

We recommend game porting toolkit.

[[Bring your game to Mac]]


# Target iPhone 15 pro
Performance gaming tier.

Min spec -> 15 pro and 15 pro max.

or consider how to scale an existing game to more iPhone 15 features.

Apple 9 GPU family.

# Optimize for performance

Metal performance HUD.
 [[Discover Metal Performance HUD]]

new system performance profile called 'sustained execution mode'.  Default profile has 3 domains:
1.  Burst workload
2. Consistent performance for majority of customer sessions
3. steady state for longer workloads.

To provide a consistent experience, we're doing sustained execution mode for 15 pro / max.  By opting in you can get consistent state right from launch.  Quickly identify a level of quality and performance to be confident your players get consistent experience.

Capabilities -> sustained execution.

Avoid on-device shader compilation.  
Build metallibs AOT.
Turn the metallib into a GPU binary.  

```swift
MTLDevice *device = MTLCreateSystemDefaultDevice();
if ([device supportsFamily:MTLGPUFamilyApple9]) {
    // features available in Apple GPU Family 9:
    // hardware accelerated mesh shaders
    // hardware accelerated ray-tracing
} else {
    // fall back on alternative techniques
}
```

[[Bring your game to Mac]]

MetalFX upscaling.  
Spatial, temporal.  This year we're bringing it to iOS.  All iPhones in 17 for spatial, and temporal is A14+.


[[Boost performance with MetalFX Upscaling]]
[[Bring your game to Mac]]


# Focus on controllers

New generation of form-fitting controllers with low-latency usbc.

Add game controllers capability. Plist. Controllers recommended badge.
Game must support touch.  Quickly add even more customized controls to your app.  GCVirtualController API lets you draw your own on-screen controls.  

# bring your console-quality assets

Now seamlessly supports highest quality assets.

Texture decompression, geometry instancing, audio mixing, etc.
Before 15 pro / pro max, you mm ay have worried about the effort of targinet mobile devices.  Entirely new asset pipeline, etc.

But with 15pro/max, simply use existing PC/console asset pipeline.  Hardware accelerated BC  and texture supports mean you don't need to switch formats, although ASDC is superior.  Seamlessly loading, storing, moving large HQ assets.


### Slide 54: Scale textures size & quality - 0:02
```swift
MTLDevice *device = MTLCreateSystemDefaultDevice();
if (device.supportsBCTextureCompression) {
    // BCn textures are available
} else {
    // fall back to ASTC texture assets for maximum compatibility
}
```

`com.apple.developer.kernel.increased-memory-limit`.  
Check available meomry using `os_proc_available_memory()`.  

* on-demand resources (ODR)
* background assets
* custom downloading

Store them into something like Library/Caches so system can purge.

[[Meet Background Assets]]
[[What's new in Background Assets]]

# Summary
* iPHone 15 pro/max are ready for your high-end game
* everything you need, simple figure it out, no compromises
