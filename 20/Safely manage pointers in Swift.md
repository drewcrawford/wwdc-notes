#swift 

[[Unsafe Swift]]

**Unsafe operations have undefined behavior on some input**

# Pointer Type Safety
Write code at the highest safetly level possibile

1.  Safe code.  e.g. Collections, slices, iterators.  Don't use pointers at all.
2.  Unsafe APIs.  `UnsafePointer<T>`.  
3.  Raw `UnsafeRawPointer`.  Knowing the layout types
4.  Mutable type, memory-binding APIs.

# Pointer scope vs object lifetime
# Object boundaries (OOB)
# Pointer types
(e.g. pointer type vs pointee type)

If we say `assumingMemoryBound`, we are giving the compiler information that some other type is not affected.  e.g. `assumingMemoryBound(Int)` is an assertion that it doesn't alias to `UInt32`.  This could be a problem for the optimizer.

C has rules for "strict aliasing" and "type punning"

# `UnsafePointer<T>`
Typed pointer.
Memory locations bound to a typed
Typed poitners only read or write values of the memory's bound type

It may be natural to think that pointer types don't matter, as long as the values are laid out in memory correctly.  In C, this is sometimes allowed with complex rules. However, in Swift, compatible memory layout is insufficient for exchanging different typed pointers.

Memory is bound to array's element type.

## Type-safe direct memory allocation
```swift
let tPtr = UnsafeMutablePointer<T>.allocate(capacity: count)
tPtr.initialize(repeating: t, count: count)
tPtr.assign(repeating: t, count: count)
tPtr.deinitialize(count: count)
tPtr.deallocate()
```


1.  No address
2.  `.allocate`
3.  Initialized: NO,Bound type: T
4.  `.initialize`
5.  Initialized: YES, Bound type: T
6.  `.assign`.  Assignment implicitly deinitializes old memory, and re-initializes to a new value of the same type.
7.  `.deinitialize`.  At this point, memory is still bound to the same T, but "it's now safe to deallocate".  I think what this means is we run `deinit` on the type(s)?
8.  `deallocate`.  Doc says "must be uninitialized or initialized to a trivial type."

Note that Swift still handles type safety, you just handle memory lifecycle.

## Composite types

```swift
struct MyStruct {
	var i: Int
}
```

Note that in this case, `UnsafePointer<MyStruct>` and `UnsafePointer<Int>` point to the same location.

This is ok, since binding `MyStruct` implicitly binds to its members.

the case where pointers *disagree* on types is a different case, the case of reinterpreting bytes from 1 type to another, such as `fast_inv_sqrt` for example.

# `UnsafeRawPointer`

Raw pointer can load any type.  `.load(as: UInt32.self)`.  

Note that we can load memory `Int64` as `UInt32`, as long as we account for the target platform's endianness.

You can use a raw pointer to write, but it's asymmetric from loading.  Unlike assignment using a typed pointer, storing draw bytes does not de-initialize the previous value in memory.  So you have to make sure you're not breaking object references for example.

Note that casting a raw pointer back to a typed pointer is not allowed.  This is because we would wind up both with a pointer to type T and a pointer to type U, for `T!=U`, which is not allowed.

different lifecycle here

```swift
let rawPtr = UnsafeMutableRawPointer.allocate(
	byteCount: Memorylayout<T>.stride * numValues
	alignment: Memorylayout<T>.alignment
)
let tPtr = rawPtr.initializeMemory(as: T.self, repeating: t, count: numValues)
//must use typed ptr tPtr to deinitialize
```

1.  No address
2.  `.allocate
3.  Initialized: No, bound type: None
4.  `.initializeMemory<T>
5.  Initialized: Yes, Bound type: T
6.  No way to deinitialize with a raw pointer.

##  Contiguous storage for different types
This might be a reason to use `UnsafeRaw` as distinct from a typed pointer.  e.g., header and elements.

## Decode byte buffers

Use `.load(as: Descriptor.self)`
`.load(...)`

The advantage of this level is they can safely coexist with typed pointers.  So the use of a raw pointer does not contradict some other typed pointer.

However, we can *GO DEEPER*

# Memory-binding APIs
API names refer to the memory's "bound type"
Can introduce undefined behavior on existing use of typed pointers

Rule: **every typed pointer access must agree with memory's bound type**.

## Usecase: recovering a typed pointer
Only use `assumingMemoryBound` when you know memory is already bound to `T` by a prior operation.

## Usecase: pointing to tuple elements

```swift
func takesIntPointer(_ : UnsafePointer<Int>)

let tuple = (0,1,2)
withUnsafePointer(to: tuple) { (tuplePtr: UnsafePointer<(Int,Int,Int)>) in
	takesIntPointer(UnsafeRawPointer(tuplePtr).assumingMemoryBound(to: Int.self))
}
```

**Memory bound to a tuple type is also bound to its element types**

Note that lowering the pointer like this *requires knowing the layout of the tuple*

**Homogeneous tuples have a guaranteed layout**

*Tuples of different types have no layout guarantees.*

## Usecase: pointing to struct properties

```swift
func takesIntPointer(_ UnsafePointer<Int>)

struct MyStruct {
    var status: Bool
	var value: Int
}

let myStruct = MyStruct(status: true, value: 0)
withUnsafePointer(to: myStruct) { (ptr: UnsafePointer<MyStruct>) in
    let rawValuePtr = (UnsafeRawPointer(ptr) + MemoryLayout<MyStruct>.offset(of: \MyStruct.value)!)
	takesIntPointer(rawValuePtr.assumingMemoryBound(to: Int.self))
}
```
A proeprty's memory is always bound to the memory's declared type.

*In general, struct layout is not guaranteed, so when you get a pointer to the member you can only use it to point to a single value for that property.*

Note that there's a syntactic sugar for this, by passing the struct as an inout

```swift
func takesIntPointer(_ UnsafePointer<Int>)

struct MyStruct {
    var status: Bool
	var value: Int
}

let myStruct = MyStruct(status: true, value: 0)
withUnsafePointer(to: myStruct) { (ptr: UnsafePointer<MyStruct>) in
    takesIntPointer(&myStruct.value)
}
```

## `bindMemory(to: capacity:)`

e.g.

```swift
let uint16Ptr = UnsafeMutablePointer<UInt16>.allocate(capacity: 2)
uint16Ptr.initialize(repeating: 0, count: 2)
let int32Ptr = UnsafeMutableRawPointer(uint16Ptr).bindMemory(to: Int32.self, capacity: 1)
//accessing uint16Ptr is now undefined, since memory can only be bound to 1 type
int32Ptr.deallocate()
```

Note that nothing really happens here at runtime, `bindMemory` just tells the compiler that the type has changed at this location.

## Changing the bound type of a memory region
* Modifies the abstract memory state
* Doesn't physically modify memory
* Reinterprets the memory region's raw bytes in place
* But it also invalidates existing typed pointers.  Their pointer address is still valid, but accessing them is UB while memory is bound to the "wrong" type.
* Can be undefined for variable, array, and collection storage.  This is because if you rebind memory "out from under" the container, the container itself is partially invalid.


## Temporarily changing the bound type
avoid a copy in this situation
```swift
func takesUInt8Pointer(_ UnsafePointer<UInt8>)

let intPtr: UnsafePointer<Int8> = ...
int8Ptr.withMemoryRebound(to: UInt8.self, capacity: count) {
    (uint8Ptr: UnsafePointer<UInt8>) in
	//int8ptr cannot be used WITHIN this closure
	takesUInt8Pointer(uint8Ptr)
}
//uint8Ptr cannot be used OUTSIDE this closure
```


## Using `bindMemory(to: capacity:)` safely

`withMemoryRebound(to: capacity:)` has limitations

* Requires a pointer to the original type
* Both types require the same stride

Instead can call `bindMemory(to:capacity:)` directly
* LImit pointer use to a controlled scope
* Rebind memory back to the original type when the scope ends

## Memory-binding APIS

* `assumingMemoryBound(to:)` recovers a type-erased pointer type.
	* Requires prior knowledge of the memroy's bound type state
* `bindMemory(to: capacity:)` Global change to the memory's bound type state
	* Low-level operation that _invalidates existing typed pointers_
* `withMemoryReboudn(to:capacity:)` Temporarily change memroy's bound type state
	* Useful for calling C apis that disagree about types


When you just want to re-interpret a type, `rawPtr.load` is usually better, because it avoids changing the in-memory type and invalidating other pointers.

# Example: buffer view

```swift
//view memory as a different type _without restricting other pointer types_

struct UnsafeBufferView<Element> : RandomAccessCollection {
	let rawBytes: UnsafeRawBufferPointer
	let count: Int
	
	init(reinterpret rawBytes: UnsafeRawBufferPointer, as: Element.Type) {
	    self.rawBytes = rawBytes
		self.count = rawBytes.count / MemoryLayout<Element>.stride
		precondition(self.count * MemoryLayout<Element>.stride == rawBytes.count)
		precondition(Int(bitPattern: rawBytes.baseAddress).isMultiple(of: MemoryLayout<Element>.alignment))
	}
	
	public var startIndex: Int { 0 }
	public var endIndex: Int { count }
	
	public subscript(index: Int) -> Element {
	    rawBytes.load(fromByteOffset: index * MemoryLayout<Element>.stride, as: Element.self)
	}
}
```

Since loading from raw memory is safe wrt pointer types, we don't have to worry about how other pointers view the same memory.

# Wrap up
* try to avoid using pointers
* Avoid using typed pointers to reinterpret memory as different types
* Use `UnsafeRawBufferPointer` to
	* Reinterpret raw bytes as differen ttypes
	* Decode Swift types from bytestream
	* Implement a container to hold heterogeneous types in a contiguous buffer
