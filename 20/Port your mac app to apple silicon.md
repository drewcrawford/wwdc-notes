#applesilicon
# The mac is transitioning to apple silicon
No build settings changes

Porting documentation
developer.apple.com/documentation

Programs and libraries on disk are stored in mach-o
* can be built for a single arch
* can be universal
* Use `lipo` to examine

## Rosetta
The entire process is always translated (no mixing)
Can't use kernel extensions, **avx vector**, and virtualization

Debugger shows intel instructions

# Building apps as universal
Endianness of arm64 is the same as x86
Any iOS code is already ported to arm64

You can use any mac you already have to build universal apps.
Toolchain supports building for all mac architectures
## Start with trying xcode 12 (rosetta)

Native page size is different - intel is 4kb, applie silicon is 16kb
PAGE_SIZE macro si no longer a constant
Use PAGE_MAX_SIZE for a compile-time upperbound
Use bm_page_size to read a runtime value
Rosetta will provide intel environment: 4kb pages

## Build natively for apple silicon

### Use `#if` correctly


| for code specific to | objc                                        | swift                                          | Notes                                                |
|----------------------|---------------------------------------------|------------------------------------------------|------------------------------------------------------|
| macOS                | `#if TARGET_OS_OSX`                         | `#if os(macOS)`                                | Don't assume a CPU architecture implies the platform |
| Intel                | `#if TARGET_CPU_X86_64`                     | `#if arch(x86_64)`                             |                                                      |
| arm64                | `#if TARGET_CPU_ARM64`                      | `#if arch(arm64)`                              |                                                      |
| Simulator            | `#if TARGET_OS_SIMULATOR`                   | `#if targetEnvironment(simulator)`             |                                                      |
| iOS device           | `#if TARGET_OS_IOS && !TARGET_OS_SIMULATOR` | `#if os(iOS) && !targetEnvironment(simulator)` |                                                      |

### CPU-specific code, vector instructions, and assembly
Needs to be guarded by `#if`

Each x86-only usage needs a second implementation for arm64

## Resolve link-time issues
Precompiled binary framework are not ideal

All static and dynamic libraries used to buidl your app need to be built as universal.

Consider temporarily removing a dependency.

# Runtime testing
Requires apple silicon

`mach_absolute_time` is defined to be an arbitrary tick unit, and in fact is different on these architectures.

Something is running roughly 40 times slower - search your codebase for `mach_absolute_time`.  

Avoid spinlock/busywait.  Prefer os_unfair_lock.
prefer pthread_mutex, NSCondition, pthread conditions.

[[Modernizing Grand Central Dispatch Usage]]
[[meet audio workgroups]]

# Plugins

## in-process plugins
typically`dlopen` or `bundle`-based

Note that all code in 1 process must have same cpu architecture.

Always check the return path of `dlopen`.  error message is `dlerror`

Explore XPC as a solution.  Out of process plguins provide better stability and security.

[[efficient design with xpc]]

Users can force a universal app to launch in Rosetta if they have an intel plugin.

## out-of-process

Can be mixed

# Tips

## Distributing your apps
[[all about notarization]]

Archive build in xcode, produces universal apps.

[[understanding crashes and crash logs]]

Note you get 3 crash logs
* x86_64
* native arm64
* Rosetta x86_64

## System extensions
More types of kernel extensions are now deprecated or disallowed
Use DriverKit

## Watch out for multithreading bugs
Intel/apple has a different memory ordering model
Bugs might have different observable effects.
A data race might appear to be benign on x86, but can crash on apple
rosetta provides intel cpu memory ordering
Use #tsan 

# Summary
Search for non-universal binaries
Use xcode
Test/benchmark

developer.apple.com/documentation - "apple silicon" provides guidance and answers to porting your app.

#arm64