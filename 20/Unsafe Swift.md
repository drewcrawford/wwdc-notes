#swift 

# What is `unsafe`?

Most operations in the stdlib validate their input.  e.g., `assert` or `!`

*Safe operations have well-defined behavior on all input* (No UB)

*Unsafe operations have undefined behavior on some input*


ex `.unsafelyUnwrapped`.  However, `-O`, we don't verify the unwrap.

UB.

Assumptions that they are unable or unwilling to verify.

Unsafe names are like a hazard symbol.  

# Beneifts of unsafe interfaces
* Interoperability with code written in C or objc
* Control over runtime performance (or some other aspect of the execution of your program)

Note that `unsafelyUnwrapped` validates in `-O0`, but not `-O`.  Recommend you replicate this idea.

The goal of safe APIs is *Not to prevent crashes*.  It's maybe the opposite?  Safe APIs *guarantee* to stop execution.

# Unsafe Pointers
Swift has a flat memory model.  Linear address space of 8-bit bytes (?)

At rutnime, address space is sparsely populated with data.  

* App executable
* Linked libraries
* Stack (includes some fucntion arguments)
* Dynamic allocations
* Resources

Each item is assigned a contiguous memory region in a shared address space.

Unsafe pointers give is low-level operations to effectively manage memory ourselves.  With great power, comes great responsibility.

They have to trust you that you will use them correctly.  

demo: dereferencing a dangling pointer

# Address sanitizer
#asan
Detects memory management issues
[[Safely Manage Pointers in Swift]]

# C interop

| C                               | swift                           |
|---------------------------------|---------------------------------|
| `const Pointee *`               | `UnsafePointer<Pointee>`        |
| `Pointee *`                     | `UnsafeMutablePointer<Pointee>` |
| `const void *`, opaque pointers | `UnsafeRawPointer`              |
| `void *`, opaque pointers       | `UnsafeMutablePointer`          |


Consider using the `Buffer` variants to get OOB checks.  In unoptimized builds, these check for OOB through subscript operator.  Still, partial checking is better than none.

## Convenience syntax

Note that *we can pass an array to a function expecting an unsafe pointer*.  Note that the pointer is only valid for the duration of the call.  So if the function saves the pointer, that is UB.



| C pointer      | Swift pointer             | Swift shorthand |
|----------------|---------------------------|-----------------|
| `const T *`    | `UnsafePointer<T>`        | `[T]`           |
| `T *`          | `UnsafeMutablePointer<T>` | `inout [T]`     |
| `const char *` | `UnsafePointer<CChar>`    | `String`        |
| `T *`          | `UnsafeMutablePointer<T>` | `inout T`       |

Swift 5.3 compiler now produces a warning when it can detect dangling pointer use in Swift.

# Array initializers

Now you can init `Array` and `String` with `unsafeUninitizedCapacity:initializingWith:`.

This avoids the need to allocate a temporary buffer and then copy into string as would be teh case with `cString:`-like initializers

# Pattern involving calling C func twice

First to get the length of a buffer, second with an output buffer

# Best practices

Follow requirements of each unsafe interface
Isolate use of unsafe APIs
Use buffer pointers where possible

Use #asan 

