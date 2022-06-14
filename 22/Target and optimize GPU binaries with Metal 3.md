#metal 

# Offline compilation
Move GPU binary compilation to build time.

Metal library from source => AIR.  =>PSD (lightweight) => PSO (heavyweight).

Metal stores GPU binaries in file syustem cache.  Newly-generated functions are added.  

Explicitly control gPU binaries using binary archives.  Use PSO descriptor to cache GPU binary.  As many times as you need.

More flexible caching but still have to be generated at runtime.  Now you can build things offline.  Pipeline script is a toolchain equivalent to a collection of pipeline descriptors.  This generates a binary archive.

Benefits
* faster app launch and load times
* remove runtime stutters
	* You might be JITing shaders, and that causes a framerate drop.

generate pipelines script
generate GPU binaries

## pipeline script
* state descriptor in JSON format
* JSON editor, harvest during development, etc.

```objc
// An existing Obj-C render pipeline descriptor
NSError *error = nil;
id<MTLDevice> device = MTLCreateSystemDefaultDevice();

id<MTLLibrary> library = [device newLibraryWithFile:@"default.metallib" error:&error];

MTLRenderPipelineDescriptor *desc = [MTLRenderPipelineDescriptor new];
desc.vertexFunction = [library newFunctionWithName:@"vert_main"];
desc.fragmentFunction = [library newFunctionWithName:@"frag_main"];
desc.rasterSampleCount = 2;
desc.colorAttachments[0].pixelFormat = MTLPixelFormatBGRA8Unorm;
desc.depthAttachmentPixelFormat = MTLPixelFormatDepth32Float;
```

can generate binary archives at runtime and serialize during development and testing

```objc
// Create pipeline descriptor
MTLRenderPipelineDescriptor *pipeline_desc = [MTLRenderPipelineDescriptor new];
pipeline_desc.vertexFunction = [library newFunctionWithName:@"vert_main"];
pipeline_desc.fragmentFunction = [library newFunctionWithName:@"frag_main"];
pipeline_desc.rasterSampleCount = 2;
pipeline_desc.colorAttachments[0].pixelFormat = MTLPixelFormatBGRA8Unorm;
pipeline_desc.depthAttachmentPixelFormat = MTLPixelFormatDepth32Float;

// Add pipeline descriptor to new archive
MTLBinaryArchiveDescriptor* archive_desc = [MTLBinaryArchiveDescriptor new];
id<MTLBinaryArchive> archive = [device newBinaryArchiveWithDescriptor:archive_desc error:&error];
bool success = [archive addRenderPipelineFunctionsWithDescriptor:pipeline_desc error:&error];

// Serialize archive to file system
NSURL *url = [NSURL fileURLWithPath:@"harvested-binaryArchive.metallib"];
success = [archive serializeToURL:url error:&error];
```

`metal-=source` tool extracts a piplines script from an archive
```bash
metal-source -flatbuffers=json harvested-binaryArchive.metallib -o /tmp/descriptors.mtlp-json
```

generate gpu binary from source:
```bash
metal shaders.metal -N descriptors.mtlp-json -o archive.metallib
```


from library:
```bash
metal-tt shaders.metallib descriptors.mtlp-json -o archive.metallib
```

load binaries
```objc
MTLBinaryArchiveDescriptor *desc = [MTLBinaryArchiveDescriptor new];
desc.url = [NSURL fileURLWithPath:@"archive.metallib"];
NSError *error = nil;
id<MTLDevice> device = MTLCreateSystemDefaultDevice();
id<MTLBinaryArchive> binaryArchive = [device newBinaryArchiveWithDescriptor:desc error:&error];
```

## compatibility
* Metal upgrades binaries for forward compatibility
	* "during OS updates or at app install time"

# Optimize for size

compiler optimizations may expand code size
ex, inlining
ex, loop unrolling

optimize for size
* limits size-expanding transformations
* smaller size, faster compile
* change in runtime performance
	* whether it happens depends on the program
* Use with large shaders, deep inlining, loops, and long compile times
* at project build time or at app runtime

can be faster => fewer instruction cache misses, fewer registers, etc.  These results are not typical of all shaders.

xcode build setting: Optimization level => Size \[-Os]

CLI:
```bash
xcrun metal -Os large_shader.metal

# or

xcrun metal -c -Os large_shader.metal
xcrun metal -c     more_shaders.metal
xcrun metal large_shader.air more_shaders.air
```

runtime compile:
```objc
MTLCompileOptions* options = [MTLCompileOptions new];
options.optimizationLevel = MTLLibraryOptimizationLevelSize;

NSString* source = @"...";
NSError* error = nil;
id<MTLLibrary> lib = [device newLibraryWithSource:source
                                          options:options
                                            error:&error];

```
# Wrap up
* offline compilation
* optimize for size
