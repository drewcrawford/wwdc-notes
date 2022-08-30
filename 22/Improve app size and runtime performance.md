Changes we've made on swift/objc ops.

* code
* compiler
* runtime

Several improvements we've made in comiplers/runtimes.  No new APIs, language changes, or build settings.  These improvements are transparent to you.

# Protocol checks
Where possible, we optimize out `as?`.  But at runtime, uses protocol type metadata.

with metadata, runtime knows whether it conforms to the protocol or not. p art of the metadata is built at runtime, but a lot can only be built at launch time, when using generics.

Protocol check metadata costs launch time.  Now precomputed as part of the dyld closure.  
Run on iOS 16, tvOS 16, or watchOS 9
[[App startup time past, present, and future - 18]]

It's not immediately clear to me why this happens at launch – couldn't you emit a table at link-time mapping every type to every protocol?

# Message send
Message send up to 8 bytes smaller.
Binaries up to 2% smaller overall
Build with xcode 14
even with older release as DT

Defaults to balanced performane and size
Opt into optimizing for size only using `-Wl,-objc_stubs_small`
1-2% binary size.

almost every line here needs `bl _objc_msgSend`, for property access, etc.  At copmile time we don't know which method to call.  Only objc runtime knows.  `objcMsgSend` finds the right method.

To tell the runtime which method to call,w e need the selector.  That neesd a few more instructions to prepare selector.  On arm64 it's 4 bytes each.
Each call is 12 bytes, that adds up.

We have two options.  Smallest:

```asm
bl _objc_msgSend$dateFromComponents

//selector stub
_objc_msgSend$dateFromComponents:
adrp x1, [selector "dateFromComponents"]
ldr x1, [x1, selector "dateFromComponents"]
b _objc_msgSend

//call stub
_objc_msgSend:
adrp ...
ldr ...
br ...
```

Fastest:
```asm
bl _objc_msgSend$dateFromComponents

//selector stub
_objc_msgSend$dateFromComponents:
adrp x1, [selector "dateFromComponents"]
ldr x1, [x1, selector "dateFromComponents"]
adrp ...
ldr ...
br ...
```


# Retain and release
Made cheaper.

* up to 4 bytes smaller, down from 8 on arm64
* binaries 2% smaller overall
* Set your DT to iOS 16, tvOS 16, or watchOS 9

When we call `_objc_release` in C calling convention, we have to move into x0 register.  Because probably, the value is in some other register.

New optimization, we can specialize retian/release we can use a different calling convention.  Just `_objc_release_whichregister`.  This adds up.

# Autorelease elision
* faster
* Run on iOS 16, tvOS,16, watchOS 9, macos 13
* smaller binary
* Set DT to latest

Temporaries returned via `autorelease`.  

Comipler emits a special marker.  `mov x29,x29`.  Runtime looks for marker.  However, this is not free.  Not optimal.

So instead.
1.  `_objc_autoreleaaseReturnValue`.  Runtime gets return address.  Very cheap.  Store it away.
2. Do `_objc_retainAutoreleasedReturnValue`.  Get pointer to reeturn address.  Compare this to what we saved earlier.  Comparison succeeds, etc.
3. No marker is required.

It appears we are using `_objc_claimAutoreleasedReturnValue` now.

# Wrap up
* Run on latest OS
* Build with xcode 14
* Update your DT
