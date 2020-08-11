#metal 

Errors?

* OOB access
* nil resource access
* Invalid resource access (forgot to call `useResource` with argument buffers)
* timeouts (long-running or infinite loops)

## Current state of errors reporting
`Discarded (victim of GPU error/recovery) IOAF code 5`

vs API errors where we get an assert etc.  Type of error, callstack, etc.

# Enhanced command buffer errors

Basically this identifies which encoder is the issue.
`desc.errorOptions = .encoderExecutionStatus`

* `faulted`:  encoder was directly responsible for the command buffer fault
* `affected`: maybe, suspicious
* `unknown`: we're not sure

This should always be on for development and QA.

Since enhanced command buffer errors are built into metal
You *can* ship your application with this enabled.  That said, it may impact performance.

Note that this tool only gets to the encoder level.

# Shader validation

Catches metal shader code issues
Prevents & logs a GPU backtrace

Basically this is GPUsan.  This additionally checks for UB.  nOte that reading out-of-bounds is UB even if you don't do anything with it.

Can detect:
* OOB global memory
* OOB threadgrou pmemory
* Null texture access
* Infinite loops – not caught by shader validation but is caught by ECBE
* Resource residency – not caught by shader validation but is caught by ECBE

Click the arrow to the right to add a "Metal Diagnostics" breakpoint.  You still need this breakpoint in order to get debug breaks.  This shows the CPU and GPU Backtrace.

Note this config (not clear if it's default):

* Enable runtime issue breakpoint
* Type: System Frameworks
* Category: Metal Diagnostics


## Without xcode
New environment variables
Must be set before metal device is created
`MTL_DEBUG_LAYER=1` (any nonzero value)
`MTL_SHADER_VALIDATION=1`

`commandBuffer.logs` => only valid after command buffer finishes.  For that reason, do the work inside the completion handler.

Can also get info from syslog

`log stream --predicate "subsystem = 'com.apple.Metal' and category = 'GPUDebug'"`

## Tips

Expect pipelines to take longer to compile.  You may want to paralellize compilation.

Enable debug symbols.  Automatically with xcode, or otherwise `metal -g`.

Recommend using the `#line` preprocessor directive for online compile, so as to get good debug info.

Shader validation is process-wide.

It has a large performance impact.

Not intended for use in the field.

Changes to occupancy `maxTotalThreadsPerThreadgroup` and `threadExecutionWidth`.

## Configuration settings

`MTL_SHADER_VALIDATION_TEXTURE_USAGE=0` -> disables texture checks in the shader
Removing unneeded checks can improve performance.

Full list can be found with `man MetalValidation`.

Note that

* Binary function pointers
* Dynamic linking

are not supported with validation.

Note that instrumenting pointers from argument buffers are not supported on

* MTLGPUFamilyApply5 and older
* MTLGPUFamilyMac1


