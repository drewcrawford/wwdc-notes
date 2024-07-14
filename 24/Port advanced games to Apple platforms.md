Discover how simple it can be to reach players on Apple platforms worldwide. We'll show you how to evaluate your Windows executable on Apple silicon, start your game port with code samples, convert your shader code to Metal, and bring your game to Mac, iPhone, and iPad. Explore enhanced Metal tools that understand HLSL shaders to validate, debug, and profile your ported shaders on Metal.

Last year we brought game porting toolkit.  New GPT2 brings updated set of tools to accelerate porting your games.

* Evaluation environment for windows games
* Human interface guidelines for games
* Game porting example code
* Metal-cpp
* Metal shader converter
* Metal developer tools for Windows

# Evaluate your game

This year we support more game technologies.  AVX.  Raytracing.  Increased compatibility/performance.  

Debug/profile original HLSL shaders.
Whiskey, CrossOver, etc.

Apple design conventions.  [[Design advanced games for Apple platforms]]

New HIG section.  Tips on topics like improving first launch experience, ensuring your fonts are legible, etc.
# Port your game

Amazing new sample code to help you accomplish your goals.  Interactive tutorial, step-by-step through main elements of porting process.  Organize documents into folders, each one providing concrete information on how to port grpahics, shaders, audio, game controllers, etc.

## configuration

Target all apple devices
Create a uniform target, customize when needed
Great for games

Xcode makes it easy to adapt your code to each SDK
Use target conditions and filters - TARGET_OS_IOS, etc.

Contain whole files tailor-made for only one target.  Xcode lets you use filters to specify files for a specific target based on SDK.  For shared files just stick to 'always used'.

Majority of apple frameworks target both macOS and iOS.  Enhancements even easier!

Can now compile your metal shaders once for both iOS/macOS.  Unified metal device initialization.  Certification API for performance profiles.

Game mode is now on iOS.  Reduce background activity, bluetooth latency.  `<key>GCSupportsGameMode</key>` to ensure you opt into it.  We might enable automatically.

Metal-cpp: our official c++ bindings for Metal.  Part of Game Porting Toolkit 2.  
Familiar syntax
No measurable overhead

[[Program Metal in C++ with metal-cpp]]

## shaders

Includes metal shader converter.  Bring your shaders to Metal.  Supports all shader stages
* ray tracing and mesh shaders
* (legacy) geometry and tesselation shaders
accelerates your porting timeline

Learn more about metal shader converter
* port resource layouts
* runtime library
* bind resources to your pipeline
see docs.

[[Bring your game to Mac]]

Leverage globally-coherent texture access in Metal.
Use the full power of metal tools to debug/profile your converted shaders.  Convert your shaders to metalIR once and deploy across all apple devices.

Use as command-line tool or dynamic library
Available on Windows and macOS.

## graphics

Modern graphics and compute.
Designed and optimized for apple devices
Wide range of modern features
Improvements designed for games

### residency

On other platforms we copy resources from main memory.  On apple platform, main memory gives you access to large amounts of memory.  But must tell metal which resources to make resident.

Residency sets -> load resource sinto unified memory.  Residency sets allow you to define groups of resources to make resident all at once.  Don't need to track individual resources.


### Build a residency set - 12:51
```cpp
// Build a residency set. 
// Create a new residency set. 
MTL::ResidencySet* residencySet; 
residencySet = device->newResidencySet(residencySetDescriptor, &error); 
// Add to main command queue. 
commandQueue->addResidencySet(residencySet); 
// Add allocations and commit changes. 
residencySet->addAllocation(texture); 
residencySet->addAllocation(buffer); 
residencySet->addAllocation(heap); 
residencySet->commit(); 
// Use residency sets. 
// Allocate and encode a command buffer. 
MTL::CommandBuffer* commandBuffer = commandQueue->commandBuffer(); 
// ... 
// The command queue marks residency for the set for this command buffer. 
commandBuffer->commit();
```

Check out documentation and sample code.

Simplified residency.  
Row-major matrices
Direct access to the intersection result.

Upscaling with MetalFX.  Scaling a lower-resolution image up to target output resolution.  


### Upscale image with MetalFX - 14:46
```objc
// Upscale image with MetalFX. 
mfxTemporalScaler->setColorTexture(currentFrameColor); 
mfxTemporalScaler->setDepthTexture(currentFrameDepth); 
mfxTemporalScaler->setMotionTexture(currentFrameMotion); 
mfxTemporalScaler->setOutputTexture(currentFrameUpscaledColor); 
mfxTemporalScaler->setJitterOffsetX(currentFrameJitter.x); 
mfxTemporalScaler->setJitterOffsetY(currentFrameJitter.y); 
mfxTemporalScaler->setReactiveMaskTexture(currentFrameReactiveMask); 
mfxTemporalScaler->encodeToCommandBuffer(commandBuffer);
```

Achieve higher fidelity, etc.  

Metal offers many more advanced grphics and compute features

[[Bring your game to Mac]]

[[Design advanced games for Apple platforms]]


## input and rumble
Input takes many forms.  Common to iOS/mac devices, use gamecontrolers, mice, keyboards.

iOS recognizes up to 10 touches at once.  GC framework is a unified solution.  Its modern and flexible design enables you to register event callback and poll.
Preferred method for input.


Input is only half the story.  Also convey feedback with rumble.  Implement rumble using GC framework in combination with corehaptics.

Haptic engine abstracts away vendor differences.  Check out [[Advancements in Game Controllers]] and [[Introducing core haptics - 19]]
## audio
PHASE framework.  Understand your app's scene.  Check out [[Discover geometry-aware audio with the Physical Audio Spatialization Engine (PHASE)]]

Games may integrate with middleware SDKs.  FMOD, etc.  
## cloud saves
Keep game state synchronized across all player devices.  

### Use the cloud save manager - 19:53
```objc
// Use the cloud save manager. 
CloudSaveManager* cloudSaveManager = [[CloudSaveManager alloc] initWithCloudIdentifier:@"iCloud.com.mycompany.mygame" saveDirectoryURL:[NSURL fileURLWithPath:@"/path/to/saves"]]; 
[cloudSaveManager syncWithCompletionHandler:^(BOOL conflictDetected, NSError *error) { 
  // Handle conflicts or errors, for example, by presenting a choice. 
}]; 
// Access and write saves 
[cloudSaveManager uploadWithCompletionHandler:^(BOOL conflictDetected, NSError *error) { 
  // Handle errors and conflicts or delay until the next sync. 
}];
```

Each achievement has a unique id.  After authenticating your player, use `reportAchievements`.  Achievements in GC allows your player to continue to make progress towards them as they switch between devices.  See example code.

# Debug and profile with Metal tools

Runtime validation.
Performance HUD.
Metal debugger.
Metal system trace.

This year, use all tools with your HLSL shaders.

* Runtime validation
* Shader debugger
* Shader profiler

Include debug information when compiling.  `-Zi -Qembed_debug` to DXC command.  Than metal shader converter can propagate info from DXIL.

## Runtime validation

API validation
* detect invalid use of metal API
* Lightweight, so keep it enabled for development
* Improved for metal shader converter

Shader validation
* identify shader issues at runtime
* asan
* resource usage and residency
* stack overflow
* texture type mismatch

Shader validation
* select which pipelines to validate
* Focus on a specific area of your game
* Ignore errors that are not relevant at the moment
* Mitigate perf impact

see docs

## shader debugger

Debug navigator.  

## shader profiler

Identify most expensive shaders to identify which are the most expensive functions/line of code

Shader cost graph.  Source code is annotated with performance statistics, indicationg how expensive each line of code is.  

Quickly hone in the most expensive part of your shaders.  May behave differently for different pixels, etc.  Using distinct resources depending on pixel location.  In these situations, heat map helps you visualize most expensive pixels.

Perf analysis workflows bring best tooling experience for your games, whether you rely on mS converter or metal shading language.

See [[Discover Metal debugging, profiling, and asset creation tools]]
[[Discover new Metal profiling tools for M3 and A17 Pro]]

# Wrap up

* bring your gams to mac, iPad, and iPhone
* Game porting toolkit 2 helps you with the process
* New example code and metal tools make it easier than ever

[[Design advanced games for Apple platforms]]



# Resources
* https://developer.apple.com/games/game-porting-toolkit
* https://developer.apple.com/metal/shader-converter/
* https://developer.apple.com/metal/cpp/
* https://developer.apple.com/documentation/metal
* https://developer.apple.com/metal/
* https://developer.apple.com/documentation/metal/metal_sample_code_library/rendering_reflections_in_real_time_using_ray_tracing
* https://developer.apple.com/documentation/metal/resource_fundamentals/simplifying_gpu_resource_management_with_residency_sets
* https://developer.apple.com/documentation/Xcode/Validating-your-apps-Metal-API-usage
* https://developer.apple.com/documentation/Xcode/Validating-your-apps-Metal-shader-usage
