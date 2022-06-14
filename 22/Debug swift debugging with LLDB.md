#lldb

[[Discover breakpoint improvements]]
# Integrate a third-party framework
> In my spare time I write parsers for text adventures.

> Couldn't realize type of self

yesterday I was using the debugger with no problems, and then last night I added this UI framework.  The developers of that framework cranks out nightly builds, and I'm linking against it.  Does this have something to do with my debugging troubles?

I only see disassembly even though I downloaded the debug build.  How do we understaand this?

What does lldb need to show suorcecode?  Compiler generates machinecode.  And it leaves breadcrumbs in the debugger so an address in the executable can be mapped to sourcefile.  These are called debug info.  On apple platforms, stored in object files.  For archiving and distribution, debug info can belinked into dSYM.  lldb uses spotlight to locate these bundles so it's quite flexible in terms of locating them.

Let's verify that lldb has found the dSYM for the framework.
```lldb
image list
```
Yes, we see the dSYM there.  We can use image lookup to get more info.

```lldb
image lookup -va $pc
```

lldb has excellent built-in help.
image lookup shows our sourcecode on `/BUILD_SERVER`, not my local machine.  Can fix that.

```lldb
settings list target.source-map
```

In the scheme editor, we can make a permanent change to adjust this.  LLDB init file.  We make it say some lldb commands, that is

```lldb
settings set target.source-map /Volumes/BUILD_SERVER/projects /Users/demo/Desktop/Adventure/3rdparty
```
this tells it to look in the right folder.

Customizable lldb setting.  Put this into your project lldb init file to have it run automatically.  Alterantively each DSYM bundle has an XML plist file where you can put `DBGSourcePathRemapping` dictionary.  Can inject the remapping dictionary in there.  See lldb website.

Not language-specific, works for swift, c++, objc.  [[Symbolication Beyond the basics]]

```lldb
settings set target.source-map prefix new
```

Build paths may be different on machine to machine.  We can instruct the compiler to canonicalize source paths.
```lldb
-debug-prefix-map $PWD=/BUILDROOT
```



```lldb
po words
expr -O -- words
```
> Cannot realize type of self

!!!

this doesn't work either:
```lldb
p words
expr words
```

Instead we can use `frame variable` or `v` to see the variables

```lldb
v words
frame variable words
```

check out [[LLDB Beyond PO - 19]] for details.

What is `po` and why doesn't it work?   Well lldb is a debugger, but it is also a compiler.  You have to think of these two halves as independent components that work somewhat differently.

lldb has a fully-functioning swift/clang compiler.  computation, call functions, change state

[[Advanced debugging with Xcode and LLDB]]

Reading raw bytes:
```lldb
mem read UnsafePointer<Items>(self.inventory)
```

We turn that into output with types.  Allows lldb to understand structure of source variable.  Knows fields, etc.

Where do types come from?
On debugger side, `frame variable` comes from debug info.  
And reflection metadata in swift.

On compiler side, `p`/`po` gets type information from *modules*.
This separation is new in xcode 14.  

Swift compiler knows many ways of importing modules, but first, let's diagnose an issue on the compiler (as opposed to debugger) side of lldb.

```lldb
swift-healthcheck
```

We can get access to a log of the swift expression evaluation.  I think it's the prior command.

We see from the log we're missing some module inside our framework.  Our type is generic over that type, so we'll see I guess.

How lldb finds swift modules:
my module imports system `.swiftinterface`
might import clang module (e.g. modulemap, headers, etc)
might depend on other clang modules.

My app might import a swiftmodule from some framework
[[Binary frameworks - 19]]

There are also bridging headers that can import clang modules.
lldb only – some module ocntents can be constructed from debug info alone.

That's a lot of sources.  The build system is supposed to package up data where lldb can find them.  System frameworks stay in the SDK.  Lldb will find a matching SDK when it attaches.  lldb finds all non-sdk modules where they were at buildtime.  dsym bundle => ddynamic framework, dylib, executable.  dsym may contain binary modules (may contain bridging files), etc.

This covers everything except swift modules that belong to static archives.  Dynamic libraries the build system happens automatically.  But static archives are *not produced by the linker*.  That means responsibility for registering swift modules with the linker falls on anything that links the static archive.  Usually, xcode's build system will do this for you, but if you have your own custom build system or rules, be aware.

* apple platforms:
```lldb
ld … -add_ast_path /path/to/My.swiftmodule
```

Check your build log to verify this is the case.
Verify in executable:

```lldb
dsymutil -s MyApp | grep .swiftmodule
```

On linux, you register with the linker like so:
```lldb
swiftc -modulewrap My.swiftmodule -o My.swiftmodule.o
```

> The developers of the framework were *incredibly* responsive!
The swift module from some static archive was missing from the dsym bundle.

Now self can be resolved
```lldb
p self
```
and this works
```lldb
po words
expr -O -- words
```

```lldb
s
thread step-in
```

```lldb
n
thread step-over
```
# Add serialized search paths in swift modules
compiler will serialize the headers in during build.  But when you're building on a different machine, these local paths can be detrimental.
Consider building with
```lldb
-no-serialize-debugging-options
```
or in xcode:

```bash
SWIFT_SERIALIZE_DEBUGGING_OPTIONS=NO
```
can reintroduce search paths with these LLDB settings:

```lldb
settings set target.swift-extra-clang-flags …
settings set target.swift-framework-search-paths …
settings set target.swift-module-search-paths …
```

Various cases depending on what debugging you will be doing.
Crash logs only (library vendor):
* ship a textual `.swiftinterface`

To step into the code with LLDB (build server):
* consider `no-serialize-debugging-options`
* `-debug-prefix-map` or

# wrap up
LLDB is a debugger (frame variable / v / xcode variable view)
* debug info
* reflection metadata

lldb is a compiler (expr / p / po)
* modules
* bridging headers
* clang module search paths
* swift-healthcheck
