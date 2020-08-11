#objectivec

No code changes, no new APIs, just get faster.

Partly, we think this is cool and interesting.  Only possible because our internal datastrructures are hidden behind APIs.

# Class data structures changes
Class on disk.  

1.  Myclass.  Metaclass, superclass, flags, method cache
2.  `class_ro_t`: Flags, size, name, methods,protocols,Ivars,properties

Clean memory vs dirty memory.  `class-ro_t` is clean.  

Dirty memory is especially costly in iOS, because iOS has no swap.  Can't swap out the dirty memory.

This structure allows most class data to be clean.

Note the class also has `class_rw_t` which has flags, first subclass, next sibling class, methods, properties, protocls, demangled name.  Why do we have `class_rw_t` with methods, properties, protocls?  Because in ObjC these can be dynamically modified, swizzling, etc.  

We measured about 30MB in class_rw_t across the system on an iPhone.  But only around 10% of classes have their methods changed.  And the demangled name is only used by swift class, and it isn't even used unless somebody asks for the objc name.

So we can split into `class_rw_t` and `class_rw_ext_t`, for the parts that aren't commonly used.  This saves 14mb systemwide.

```bash
% heap Obsidian | egrep 'class_rw|COUNT'
    COUNT     BYTES       AVG   CLASS_NAME                                       TYPE    BINARY
     2914     93248      32.0   Class.data (class_rw_t)                          C       libobjc.A.dylib
      325     15600      48.0   Class.data.extended (class_rw_ext_t)             C       libobjc.A.dylib
	  
```

Note that if you were relying on this directly, you will break this year.  Also watch out for external dependencies.

You can use official apis like `class_getName`, `class_getSuperclass`, `class_copyMethodList`.

# Relative method lists

Method lists contain 3 pieces of information
1.  Method name
2.  Type encoding - string that represents parameter and return types
3.  pointer to the impl

Note that each is 8 bytes, for 8 * 3 = 24.  

A binary image can be loaded anywhere in memory.  

Method list entries don't actually need to refer to the entire 64-bit address space, only within their own binary.

So instead of an absolute 64-bit address, they can use a 32-bit relative offset.

Firstly, the offsets are always the same, so they don't need to be "fixed up".
Therefore, they can be held in read-only memory.  And 32-bit offets mean 4 * 3 = 12 bytes.

80MB of these methods systemwide on an iPhone.  Half the size = 40MB.

How to handle swizzling relative method lsits?

Global table.  

Note that you need DT =14 to get these behaviors.

If DT=13, you get old behavior.  If you can target this year's OS releases, you get smaller binaries and less memory usage.

Minimum SDKs aren't just about API versions.  Xcode can emit better-optimized code or data.  We understand that many of you need to support older OS versions.  But increase your DT whenever you can.

## Mismatched deployment targets
Beware code that is built with an old DT and tries to interpret these methods.  It will try to interpret method name and type encoding as the method name itself (since size halved), and that's a bit weird.

You can recognize this by a cras in the runtime reading method information, where the bad ptr looks like 2 32-bit values smusshed together (e.g. `0`s inbetween).


# Targed pointer format changes

Notice that
* low bits are unused, due to alignment requirements.  Obj must be located at an address that's a multiple of the pointer size.
* High bits are always 0 because the address space is limited.

Note that tagged pointers are obfuscated with a randomzied value that's initialized at process startup, for security reasons.  This makes it more difficult to forge a tagged pointer.

On intel, low bit is 1 to indicate it's tagged.  Next 3 bits indicate the type of tag, the rest is the payload.

Tag 7 is an extended tag, which uses the next 8 bits to encode the type.  This is used for UIColors or NSIndexedSets.

Note that if you use Swift, an enum like

```swift
enum Tagged {
    class(SomeClass)
	other()
}
```

the `other` case is stored in the spare bits of the associated value payload.

## ARM
On arm64, we flipped this around.  Highest bit is tag, next highest 3 is tag, etc.

Why flip?  `objc_msgSend`.  

Turns out we can do this
```c
if (ptrValue <= 0) //is tagged or nil
```
in a single comparison, this saves a comparison in the common case.


This year, we are shuffling the bits around a bit.

`[t][extended][payload][tag]`

This is ultimately because we want to put a normal pointer in the payload of a tagged pointer.  Which can be done by chopping some things off on arm.  this is useful so that we can refer to some string or datastructure in your binary.

Don't rely on these internal details!
