#metal 

# Shader compilation model
From `.metal` to `.air`.   Typically this happens in xcode.
However, when PSOs are created we may do additonal compilation on-device.

We cache the metal function variants produced in this step for future pipeline creation.  But for games and apps that adopt best-practice of building pipeline objects upfront, may take long loading screens.

Developers want to
* save the entire time cost of compilation
* enable sharing common subroutines
* ship apps that already include the final compiled code for executables and libraries
* share GPU binaries and libraries with other developers

# Metal binary archives
Cache pieline state binaries and distribute them

Feature of MTLGPUFamilyApple3 and mac1

`newBinaryArchiveWithDescriptor`

Basically this seems to be a way to cache the result of a PSO.  You need the exact device and OS, and you can serialize the thing into a file, which you can then deploy to compatible devices.  This avoids the compilation process.

Create `MTLBinaryArchiveDescriptor`.  

## Memory implications
Memory-mapped in.  Therefore we need to reserve virtual memory.  This will be released with the archive, so close binary archives that are no longer needed.

Unlike the metal shader cache, we can free up the cache memory.  When you re-use an existing archive, the pipelines in this archive do not count against the active app memory.  

## Best practices
* Divide your game or app into several caches.  Consider frequently-used vs per-level
* Release no-longer-needed binary archives to reclaim VM
* Favor binary archives to metal shader cache

## Case study: fornight / UE4
many cahraacter / item customization options
11k functions


# Dynamic libraries
Some runtime cost of GPU binary creation at PSO time
No reuse of machine code subroutines across pipeline states

`MTLDynamicLibrary`
Dynamically link, load, share GPU Binaries
Reusable between multiple compute pipelines
Serialzable, reloadable, and shippable

* collection of exported functions
* callable from multiple compute functions
* Dynamic libraries cannot be used to create MTLFunction objects.  However standard libraries can call

At PSO time, library is linked, not compiled.

Prevents recompiling and duplicating machine code across pipeline states.  Metal middleware provide the ability to ship utility libraries to your users.

Unlike previously, where you would have to ship source, a dynamic library can be provided and updated iwthout requiring users to build their own mtllib files

Create custom kernels.  Inject user-defined behavior into your shaders without creating the MTLLLibrary and MTLFunctions with your entrypoints.

`device.supportsDynamicLibraries()`

Note that this feature seems to require A13 (iPhone 11).


To create an `MTLDynamicLibrary`, we create a metal library, but we specify that we'd like a dynamic library.

`@executablePath` - relative to the metallib containing the exeuctable kernel
`@loader_path`: relative to the metallib containing the load command
can also use an absolute path

Dynamic lbiraries can reference other dynamic libraries.  Just set `.libraries` option on creation in `MTLCompileOptions`.  Although multiple kernels link the same dylib, only 1 dylib exists in memory.

`descriptor.insertLibraries = []`   Basically this is a higher-priority version of the `comipleOptions.libraries`.


## Distribution
Much like binary archives, dlibs can be serialized to url.  If you end up distributing the dynamic library as an asset, the metal framework will recompile the air slice into amchine code if the target device cannot use the compiled binary.  This complation is not added ot the metal shader cache, so make sure to serialize next time.

Sample xcode project on developer.apple.com.

# Developer workflows

Libraries come in 3 flavors

* MetalFunctions - executable metallibs
* Non-entrypoint or utility code, you can now create static or dynamic libraries.
* More tools in the toolchain.

`metal-libtool -static` creates `libUtility.a`

## Static libraries

metallibs are self-contained and easy to deploy
comipler and linkner have access to concrete implementations
allows for link-time optimizations
overall larger due to duplication

## dynamic libraries

`-dynamiclib` 
`-install_name` - recorded into the metallib for the loader to find the library at runtime

With dynamic libs, does not get compiled into executable `metallib` but is linked at runtime.

Embeds a "load command" into the exectuable dylib.  The load command is how we load the dylib.

How do transitive dependencies work?  

`@loader_path` is the path of the metallib containing the load command, which may be a library
`@executable_path` always points to the executable itself.

This allows different shaders to share the same utility library.

* no duplication of utility code
* Runtime dependency

Note that name resolution works a little differently.  The loader will pick 1 defintion and bind all references to it.  So you may not get the local symbol.

By default, metal exports all symbols in your library.  You can check symbols using `metal-nm foo.metallib --demangle`

How do we control what symbols are exported?  Just like on the CPU, you can use `static`, anonymous namespaces, and visibility attributes.

Use namespaces when defining you rinterfaces.
`-exported-symbols-list` linker option.

## Harvesting binaries

`metal-lipo` can peek into harvested metallib.

Fat binary that contains 2 independencent slices.  The a13 slice is backend -compiled, and the generic `air` slice.

On non-a13 devices, it will use the `air` slice.

You mgiht want to bundle all slices into a single metallib.  `metal-lipo` can achieve this, also with binary archives.