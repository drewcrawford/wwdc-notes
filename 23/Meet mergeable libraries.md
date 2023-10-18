Discover how mergeable libraries combine the best parts of static and dynamic libraries to help improve your app's productivity and runtime performance. Learn how you can enable faster development while shipping the smallest app. We'll show you how to adopt mergeable libraries in Xcode 15 and share best practices for working with your code.

new model for building/distributing libraries powered by the static linker.
# static vs dynamic libraries
static: object files archived together
contents get copied into app binary
Library not needed after building
Build time increases as library changes

dynamic libraries
aka dylibs
Binary inside frameworks
Path to library is saved in app binary
Embedded into app bundle
Minimal static link time impact
dyld must find/load framework dependencies
transitive dependencies
increases memory consumption and app launch times
apps can load 100s of frameworks

tradeoffs.
* dynamic libraries: optimal buildtime
* static libraries: optimal runtime
* we have historically recommended measuring

with mergeable libraries, this is no longer needed
unlock the best of both linking strategies

# Meet mergeable libraries
Consider any binary image, like an executable.
frameworks given to the static linker.  mergeable dylib, and the link output is merged binary.

* built as dynamic libraries
* metadata is emitted into the library
* allows the linker to treat similarly to a static library
* can be linked as a dynamic library or merged
* can be an executable or dynamic library
* merging is similar to static library linking
* The binary type remains the same

Enabled by the new static linker.
Libraries emit metadata with `-make_mergeable`.  Tells the linker to record the metadata.
Binary merges all input with `-merge_library` or `-merge_framework`.
Xcode handles build steps
see these in the build log!

Mergeable libraries only needed at build time.  Metadata can be removed.
Linker deduplicates across all mergeable libraries.  Redundant symbols, objc_msgsend stubs, selectors, etc.
Can apply linker options and optimizations

Improves app launch.  Fewer frameworks result in faster launch and less memory consumption.
Reduce frameworks to load with simple options.

All embedded frameworks can become mergeable.  Can create a framework that merges a contents of the other libraries.  dyld only needs to load that one library containing all segments across the embedded frameworks.  merging in this way can simplify large dependency chains.
# Using mergeable libraries
* automatic merging
* manual merging
* debug workflow
* symbolication support

## automatic merging
merges all direct dependencies that are embedded framework targets
works well on app targets

ex
* swifui
* forestbuilder
* forestui
* forest
when automatic merging is enabled, these app frameworks become mergeable.
these frameworks will be directly merged into the app binary.  Can be removed from disk.

9:18 - Enabling library merging
```
MERGED_BINARY_TYPE
```


aka "Create merged Binary".  Set this to automatic.

"Build mergeable library" will enable the library for merging.

* libraries link directly into app binary
* similar launch time performance to static libraries
* symbols from library impact app size

Use exported symbol options to reduce size
Enable `-no_exported_symbols`.  a.k.a, in "Other Linker Flags" set `-Wl, -no_exported_symbols`.
Or use an export list if you need exports to app extensions.

Allows the static linker to do dead code stripping, etc.

Maybe only some of your frameworks should be merged together?
## manual  merging
* fine-grained approach for merging
* Use when some dependencies need to stay on disk
* On merged target, set `MERGED_BINARY_TYPE=manual` on app target
* Set dylibs to merge with `MERGEABLE_LIBRARY=YES`
* Keep  other link dependencies with `MERGEABLE_LIBRARY=NO`

Create ForestKit that merges frameworks from the app, but also satisfies test dependency.  This is a "group library" that encapsulates multiple libraries.  We make the subframeworks mergeable, and the new parent framework, nonmergeable.

 Create new framework target for group library.  
 Link against all dependencies.  Update link binaries to add the frameworks using the plus sign.
 Enable manual merging on the group framework target.  
 Select which libraries to merge by visiting build settings for each framework target.  
 Forest - build mergeable
 forestUI - build mergeable
 forestbuilder - build mergeable
 test support - no?
 Retarget app to point to group library.
Retarget tests to point to group library.

Libraries are merged in release mode.  However, there is a buildtime overhead to merging.  Similar to buildtime behavior instatic libraries.
In debug mode, linker will not merge in debug mode.  Build system tells the linker to re-export dylibs insstead.
Symbols in libraries are accessed through merged binary
Faster debug builds
dyld redirects any references to reexported libraries.
that does mean in the debug case, mergeable libraries stay on disk.

## debugging
let's say we have some function.  It gets compiled.  Symbolication associates machine instructions back to sourcecode.

source location is preserved from the original library.  But keep in mind, where library information is displayed, it will show the path to the merged binary.  Presented in crashlogs, instruments, debugger.
# Considerations for mergeable libraries

Some factors worth noting.

## Mergeable libraries as dependencies
need to update to depend on the merged fraamework, since mergeables are removed from disk.

## autolinking with mergeable libraries
compiler-option that's on by default.  When the compielr fings `import Foo`, it detects framework dependencies to pass to the linker.

So if you're importing a module, this can cause dynamic linking issues.

If enabled, explicitly link against merged framework.  Just add the merged framework to the section in build phases, remove the mergeable ones.


## APIs for runtime lookup
Usually you don't use `dlopen`.  But if you do, those input paths will also need to point to the merged framework target.

Resource lookup can be impacted.  Runtime expects `Bundle(for: class)`.  These APIs are used to work with framework's resources, without having to consider the bundle's structure.
Prior to iOS 12, framework binary was needed for lookup
Mergeable framworks won't have bundles in them.
In iOS 12, a hook was added to enable lookup.

For mergeable libraries, update to iOS 12+.
If not used, can be disabled on the merged target with `OTHER_LDFLAGS = -Wl,-no_merged_libraries_hook`.

Then you won't need to update your app's deployment version.

If you're merging frameworks that don't contain bundle resources, you may not need the bundle hook.  This option improves launch time performance.


## The new static linker
* Merging is restricted to the new linker
* Older linker exists for backwards compatability.
* Can still build for arm7k.
* new linker does not.
* last platform: watchOS8
* Upgrade deployment version to watchOS9+ to use the new linker!

## Distributing frameworks
* distribute mergeable libraries as XCFrameworks
* Clients decide whether to merge
* Metadata size is proportional to the dynamic library
* Metadata will be discarded after building the app
* Metadata is stripped if not merged


# Recommendations
* Link against merged target throughout the project
* Update input in script phases to the merged binary
* Link any mergeable libraries into merged binary
* Set MERGED_BINARY_TYPE & MERGEABLE_LIBRARY at the target level.
* Update static libraries to dynamic libraries.

# Wrap-up
* Merge libraries into framework and executable targets
* Let Xcode do the work with automatic workflow
* Use manual workflow for fine-grained control
* Link against merged framework

see docs


[[Link fast improve build and launch times]]
# Resources
https://developer.apple.com/documentation/Xcode/configuring-your-project-to-use-mergeable-libraries