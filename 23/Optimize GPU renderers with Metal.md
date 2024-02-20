Discover how to optimize your GPU renderer using the latest Metal features and best practices. We'll show you how to use function specialization and parallel shader compilation to maintain responsive authoring workflows and the fastest rendering speeds, and help you tune your compute shaders for optimal performance.
### Reduce Branch Performance Cost - 3:45
```cpp
// Reduce branch performance cost
fragment FragOut frag_material_main(device Material &material [[buffer(0)]]) {
    if(material.is_glossy) {
        material_glossy(material);
    }

    if(material.has_shadows) {
        light_shadows(material);
    }

    if(material.has_reflections) {
        trace_reflections(material);
    }

    if(material.is_volumetric) {
        output_volume_parameters(material);
    }

    return output_material();
}
```

### Function constant declaration per material feature - 3:55
```cpp
constant bool IsGlossy        [[function_constant(0)]];
constant bool HasShadows      [[function_constant(1)]];
constant bool HasReflections  [[function_constant(2)]];
constant bool IsVolumetric    [[function_constant(3)]];
```

### Dynamic branch for the feature codepath is replaced with function constants - 3:59
```cpp
if(material.has_reflections) {
  trace_reflections(material);
}
```

### Dynamic branch for the feature codepath is replaced with function constants - 4:05
```cpp
/* replaced with function constants*/

if(HasReflections) {
    trace_reflections(material);
}
```

### Reduce branch performance cost with function constants - 4:13
```cpp
constant bool IsGlossy        [[function_constant(0)]];
constant bool HasShadows      [[function_constant(1)]];
constant bool HasReflections  [[function_constant(2)]];
constant bool IsVolumetric    [[function_constant(3)]];

// Reduce branch performance cost
fragment FragOut frag_material_main(device Material &material [[buffer(0)]]) {
    if(IsGlossy) {
        material_glossy(material);
    }

    if(HasShadows) {
        light_shadows(material);
    }

    if(HasReflections) {
        trace_reflections(material);
    }

    if(IsVolumetric) {
        output_volume_parameters(material);
    }

    return output_material();
}
```

### Function constants for material parameters - 4:58
```cpp
// Function constants for material parameters

constant float4 MaterialColor  [[function_constant(0)]];
constant float4 MaterialWeight [[function_constant(1)]];
constant float4 SheenColor     [[function_constant(2)]];
constant float4 SheenFactor    [[function_constant(3)]];

struct Material {
    float4 blend_factor;
};

void material_glossy(const constant Material& material) {
    float4 light, sheen;
    light = glossy_eval(MaterialColor, MaterialWeight);
    sheen = sheen_eval(SheenColor, SheenFactor);
    glossy_output_write(light, sheen, material.blend_factor);
}
```

### MaterialParameter structure for constant parameters - 5:21
```cpp
struct MaterialParameter {
    NSString* name;
    MTLDataType type;
    void* value_ptr;
};

MaterialParameter is_glossy{@"IsGlossy",      MTLDataTypeBool,   &material.is_glossy};
MaterialParameter mat_color{@"MaterialColor", MTLDataTypeFloat4, &material.color};
```

### Declare and populate MTLFunctionConstantValues - 5:51
```objc
// Declare and populate MTLFunctionConstantValues
MTLFunctionConstantValues* values = [MTLFunctionConstantValues new];
    
for(const MaterialParameter& parameter : shader_parameters) {
    [values setConstantValue: parameter.value_ptr
                        type: parameter.type
                    withName: parameter.name];
}
```

### Create pipeline render state object with function constant declarations - 5:51
```cpp
struct Material {
  bool is_glossy;
  float color[4];
};

struct MaterialParameter {
    NSString* name;
    MTLDataType type;
    void* value_ptr;
};

// Declare material
Material material = {true, {1.0f,0.0f,0.0f,1.0f}};

// Declare function constant paramters
MaterialParameter is_glossy{@"IsGlossy",      MTLDataTypeBool,   &material.is_glossy};
MaterialParameter mat_color{@"MaterialColor", MTLDataTypeFloat4, &material.color};

MaterialParameter shader_parameters[2] = {is_glossy, mat_color};

// Declare and populate MTLFunctionConstantValues
MTLFunctionConstantValues* values = [MTLFunctionConstantValues new];
    
for(const MaterialParameter& parameter : shader_parameters) {
    [values setConstantValue: parameter.value_ptr
                        type: parameter.type
                    withName: parameter.name];
}

// Create MTLRenderPipelineDescriptor and create shader function from MTLLibrary
MTLRenderPipelineDescriptor *dsc = [MTLRenderPipelineDescriptor new];
NSError* error = nil;

dsc.fragmentFunction = [shader_library newFunctionWithName:@"frag_material_main"
                                            constantValues:values
                                                     error:&error];

// Create pipeline render state object
id<MTLRenderPipelineState> pso = [device newRenderPipelineStateWithDescriptor:dsc
                                                                        error:&error];
```

### Create MTLRenderPipelineDescriptor and create shader function from MTLLibrary - 6:14
```objc
// Create MTLRenderPipelineDescriptor and create shader function from MTLLibrary
MTLRenderPipelineDescriptor *dsc = [MTLRenderPipelineDescriptor new];
NSError* error = nil;

dsc.fragmentFunction = [shader_library newFunctionWithName:@"frag_material_main"
                                            constantValues:values
                                                     error:&error];
```

### Shader library creation - 8:07
```objc
- (void)newLibraryWithSource:(NSString *)source 
                     options:(MTLCompileOptions *)options
           completionHandler:(MTLNewLibraryCompletionHandler)completionHandler;
```

### Render pipeline state creation - 8:09
```objc
- (void)newRenderPipelineStateWithDescriptor:(MTLRenderPipelineDescriptor *)descriptor
              completionHandler:(MTLNewRenderPipelineStateCompletionHandler)completionHandler;
```

### Use as many threads as possible for concurrent compilation - 9:00
```objc
@property (atomic) BOOL shouldMaximizeConcurrentCompilation;
```

### Assign symbol visibility to default or hidden - 10:58
```objc
__attribute__((visibility("default")))
             void matrix_mul();

__attribute__((visibility("hidden")))
             void matrix_mul_internal();
```

### Verify device support - 11:19
```objc
//For render pipelines
@property (readonly) BOOL supportsRenderDynamicLibraries;

//For compute pipelines
@property(readonly) BOOL supportsDynamicLibraries;
```

### Compile dynamic libraries - 11:46
```objc
//create a dynamic library from an existing Metal library

- (id<MTLDynamicLibrary>) newDynamicLibrary:(id<MTLLibrary>) library 
                                      error:(NSError **) error

//create from the URL
- (id<MTLDynamicLibrary>) newDynamicLibraryWithURL:(NSURL *) url
                                             error:(NSError **) error
```

### Dynamically link shaders - 12:18
```objc
//Pipeline state
MTLRenderPipelineDescriptor* dsc = [MTLRenderPipelineDescriptor new];
dsc.vertexPreloadedLibraries   = @[dylib_Math, dylib_Shadows];
dsc.fragmentPreloadedLibraries = @[dylib_Math, dylib_Shadows];

//Compile options
MTLCompileOptions* options = [MTLCompileOptions new];
options.libraries = @[dylib_Math, dylib_Shadows];
[device newLibraryWithSource:programString
                     options:options
                       error:&error];
```

### Specify desired max total threads per threadgroup - 13:45
```objc
@interface MTLComputePipelineDescriptor : NSObject
@property (readwrite, nonatomic) NSUInteger maxTotalThreadsPerThreadgroup;
```

### Match desired max total threads per threadgroup - 14:12
```objc
@interface MTLCompileOptions : NSObject
@property (readwrite, nonatomic) NSUInteger maxTotalThreadsPerThreadgroup;
```

### Tune Metal dynamic libraries - 14:25
```objc
MTLCompileOptions* options = [MTLCompileOptions new];
options.libraryType = MTLLibraryTypeDynamic;
options.installName = @"executable_path/dylib_Math.metallib";
if(@available(macOS 13.3, *)) {
    options.maxTotalThreadsPerThreadgroup = 768;
}

id<MTLLibrary> lib = [device newLibraryWithSource:programString
                                          options:options
                                            error:&error];
       
id<MTLDynamicLibrary> dynamicLib = [device newDynamicLibrary:lib
                                                       error:&error];
```
# Resources
* https://developer.apple.com/documentation/metal
