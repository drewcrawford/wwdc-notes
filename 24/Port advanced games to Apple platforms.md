Discover how simple it can be to reach players on Apple platforms worldwide. We'll show you how to evaluate your Windows executable on Apple silicon, start your game port with code samples, convert your shader code to Metal, and bring your game to Mac, iPhone, and iPad. Explore enhanced Metal tools that understand HLSL shaders to validate, debug, and profile your ported shaders on Metal.

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
