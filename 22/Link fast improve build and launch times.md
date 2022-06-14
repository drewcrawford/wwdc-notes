What is linking?

https://www.youtube.com/watch?v=qJRKedSUHg4

static vs dynamic linking.

# What is static linking?
How did it all start?
1970s - one source file.  proc.cc => cc => prog
Split compiler into 2 parts
* compiler
* linker

How to package up `.o` files into a library?  At the time, the standard way to bundle files together was `ar`, used for backups and distributions.

Linker was enhanced to read `.o` directly out of archive.

Now, the final program was getting big.  Many things copied in, even if only a few were used.  Instead of having thelinker use all `.o`, would only pull `.o` if doing so would resolve some undefined symbol.  So someone can build a big lib, and everybody gets what they need.

To make the selective loading a little clearer, I have a simple scenario.

in main.c => fn `main`. 
In foo.c => foo->bar
in bar.c => bar + unused
In baz.z => calls the function named `undef`.

Let's say tou decide to combine bar/baz into static lib.  What happens?
1.  Linker looks at main.o
2. sees undefined for `foo`
3. gets `foo.o`.  So `foo` is no longer undefined.  
4. But this adds a new undefined symbol for `bar`.
5. Linker checks if there are any remaining undefined symbols.  In this case `bar` remains undefined.
6. Looks at libraries to see if it will satisfy.
7. `bar.o` can satisfy.  Now there are no longer undefined symbols.
8. Linker moves on to next phase and assigns addresses for all functions.
9. Copies all functions and data to the output file.
10. Notice that `baz.o` was in the static library but not loaded.

# Recent ld64 improvements
By popular demand, we spent some time optimizing ld64.  This year it's 2x faster.

Better use of cores.  A number of areas where we can do linker work in parallel
* ocntent copied in parallel
* linkedit
* compute hashes

also
* optimized exports trie-builder algorithm
* accelerated UUID computation (now SHA256 hardware accelerated)
* improved library processing
* etc


# Static linking best practices
## When to use static libraries
Code changes force static library rebuild
including TOC.  Just a lot of extra IO.
Stable code is good.
Consider moving code under active development out of the library.

To make builds reproducible and follow traditional static library semantics, linker has to process library in a fixed order.  Some parallelization cannot be used.

Can use a linker option
## -`all_load` or `-force_load`

Enables linker to parse all `.a` files in parallel, just like `.o` files.
Helpful if you're going to load all libraries anyway, or most of them
If you have multiple static libraries implementing the same symbol, and depends on the CLI order to drive which implementation is used, you may get duplicate symbol errors.
May pull in too much code.
Add `-dead_strip` to compensate.  Linker will remove unreachable code and data.
Usually pays for itself, but if you are interested in `-all_load` and `-dead_strip`, time with and without to see if it wins for your case.

## `-no_exported_symbols`
Linker builds a trie data structure of all *exported* symbols.
Usually main executables don't need exports
This improves link time
Exports trie required if app loads plugins, or **used as host in xctest**
It only makes sense to try to suppress this if it is large.  You can run
`dyld_info -exports /path/to/binary | wc -l`
If the export count is high, but trie is never used, consider skipping it.

> One large app had about 1M exported symbols, and the linker took 2-3 symbols for that many symbols.  So that shaved 2-3s off of link time.

## `-no_deduplicate`
Combines C++ functions that contain exact same code
Expensive operation, unneded during development
Because of the expense, we limted to only look at weak-def symbols.  Those are emitted for template expansions that were not inlined.
Xcode adds this by default to Debug builds.
clang adds this when link line has `-O0`
If you ahve custom build system, consider adding it for Debug builds.

## Summary
When using xcode, change product build settings instead.

Other Linker Flags.  `-all_load`.  Notice Dead Code Stripping is set as well.
For `no_exported_symbols` and `-no_deduplicate`, we need the `-Wl,-no_deduplicate` for some reason.

# Surprises
## code missing in static library
objc categories and `__attribute__((used))`
Those object files won't get loaded by the linker if they're not referenced.

## dead stripping hides errors
Normally, duplicate symbols make us error out.  But dead tripping makes us do a reachability test.  If it turns out the missing symbol is from unreachable code, it suppresses the error.
If there are duplicate symbols in static libraries, the linker **will pick the first and not error**

## Static library may be incorporated into multiple dylibs
Each of the frameworks runs fine in isolation, but at some poitn some app uses both frameworks and you get weird runtime issues because of multiple definitions.
objc runtime will warn about multiple isntances of the same classname.

# What is dynamic linking?
As more and more libraries are made available, end program may grow in size.
Static link time will increase over time.

dylibs.  DSOs, DLLs.

What exactly is going on here?  Well, the key is that static linking treats linking with dylib differently.  Linker just records a kind of 'promise'.  symbol name used, and waht the libary's path will be at runtime.

Your program file size is under your control.  Just your code, and a list of dynamic libraries it needs at runtime.   Your program's static link time is now proportional to the size of your code and independent of the dylibs you link with.

Virtual memory ses the same dynamic library used in multiple processes, it reuses memory across processes.

What are costs for those benefits?

* Slower launch time
	* all dylibs need to be connected together.
* More dirty memory (`__DATA` pages)
	* In static library case, linker can co-locate all `__DATA` pages from all libraries together.
	* With dylibs, each library has its own data page.
* Requires a dynamic linker (dyld).
	* Need runtime to fulfill the promise.

executable has segments.  usually
* `__TEXT`
* `__DATA`
* `__LINKEDIT`

multiple of page size.  `__TEXT` has execute permissions.  At runtime, dyld has to mmap the executable with each segment's positions.  Because segments are page sized and page-aligned, it makes it straightforward for virtual memory to set up the program as backing store.  Ntohing is loaded itno ram until there is some memory access on those pages.  This triggers a page fault, etc.

Just mapping is not enough.  Somehow the program needs to be bound to the dylib.  Fixups.  

## Fixups
MACH_O file.  `__TEXT` is immutable due to codesigning.  The relative address of `_malloc` can't be known during build.  What happens is the static linker bgave us `_malloc$stub`.  This is a stub synthesized by the linker at buildtime.  This has a known address and the jump can be formed.
The stub loads a pointer from data and jumps to that location.  Now, `__DATA` is changed by dyld.  All fixups are setting the pointer in `__DATA`.

Somewher in `LINKEDIT` is the info dyld needs.

* REbases => dylib or app points within itself.  ASLR causes dyld to load dylibs at random address.  So these interior pointers cannot be set at buildtime.  dyld needs to "rebase" these pointers at launch.  On disk, these contain their target address assuming an ASLR of 0.  That way, all dyld needs to record is the *location* of each rebase location.  dyld can add actual address to each rebase location to fix them up.
* Binds.
	* Symbolic references
	* Target is a symbol name, not a number
	* e.g. pointer to `malloc`.  The string `_malloc` is stored in LINKEDIT and we use that string to look up.
	* dyld stores that value in the location specified by the bind.
* Chained fixups (new)
	* Instead of storing all fixup locations
	* Just stores where the first fixup is in each DATA page.
	* rest information is encoded in `__DATA` itself, in the place wehre the fixups will ultimately be set.
	* named from the fixups are chained together.  The LINKEDIT just says where the first fixup was.  Then, in the 64-bit pointer in `__DATA`, we have a pointer to the next fixup.
	* bit for bind vs rebase.
		* If bind, rest bits are the index
		* If rebase, rest of bits are the offset of the target within the image.
	* Runtime support already exists in 13.4 and later.
	* Use as long as your DT is 13.4 and later.

## How dyld works
1.  main executable
2. parses to find dylibs
3. mmap segments
4. recurses and parses their mach-o, etc.
5. Once it's all loaded, we lookup all bind symbols and use those when doing fixups
6. Once all fixups are done, dyld runs initializers, bottom up.

Five years ago, we announced new tech.  Some steps are the same on each launch.  Specifically parse mach-o, find dependent dylibs, lookup symbols.  As long as th eprogram and dylibs did not change, these can be cached and re-used.


# Recent dyld improvements
this year, we're announcing page-in linking

## page-in linking
Instead of dyld applying all the fixups to all dylibs at launch, kernel can apply fixups to your DATA pages lazily, on page-in.

Always been the case that the first use of an address causes the kernel to read in the page.  But now if it's a DATA page, the kernel will fixup.

For over a decade, we did this in the dyld shared cache, this year we generalized it and made it available to everyone.

* reduces dirty memory
* reduces launch time
* `DATA_CONST` pages are clean
	* evicted and recreated just like text pages
Available in iOS 16, amcOS 13, and watchOS 9
Only works for binaries with chained fixups

Linker generates chained fixups when targeting iOS 13.4 or later.

**does not work for dlopen()**, just dylibs linked at launch.


# Dynamic linking best practices
How to improve link performance?

* Use fewer dylibs
* Optimize or eliminate static initializers
	* anything that can take more than a few ms, never do in an initializer
* Find your sweet spot for static vs dynamic libraries
	* Too many static library, build/debug cycle slowed down
	* Too many dynamic libraries, launch time is slow
	* Re-evaluate with new linker
* Use chained fixups (target iOS 13.4 or later)

# New tools
## `dyld_usage`
Available in macoS 13
comand line tool that logs dyld ops
similar to `fs_usage`
Uses same mechanism as instruments.app to trace
can use with macos catalyst as well

## `dyld_info`
AVailable in macOS 12
CLI tool
similar to `nm` or `otool`
Can show info about mach-o and dylib files in cache

`dyld_info -fixups /usr/bin/zip`

`dyld_info -exports` shows all exported symbols in the dylib and the offset of each symbol from the start of dylib.

here we found a code in the cache, even though there is no file on disk.

# wrap up
* review your app's use of static and dynamic libraries
* Try new linker by building in xcode 14
* try the linker options discussed
* Try chained ifxups by targeting iOS 13.4 or later
* Try page-in in linking by running app built with chained fixups on iOS 16
