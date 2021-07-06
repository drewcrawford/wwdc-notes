# What is symbolication
Mapping application runtime to sourcecode.

```swift
func selectMagicNumber(choices: [Int]) -> Int {
    return choices[MAGIC_CHOICE]
}

func randomValue() -> Int {
    return Int.random(in: 1...100)
}

func numberChoices() -> [Int] {
    var choices = [Int]()
    for _ in 1...10 {
        choices.append(randomValue())
    }
    return choices
}

func generateMagicNumber() -> Int {
    let numbers = numberChoices()
    let magic = selectMagicNumber(choices: numbers)
    return magic
}

print("The magic number is: \(generateMagicNumber())")
```

Call into `numberChoices` to get an array of randomly-generated numbers.

What if I can't reproduce?

Not a viable way to diagnose the problem.  With the help of symbolication, ew don't have to debug from assembly.

Xcode applies symbolication to show sourcecode in the backtrace.

Profiling my app to deliver the fastest user experience.  App cycles through periods of high and low utilization.  In low utilization periods, the app is writing file content.

My instruments trace is only partially symbolicated.  Can locate dsym in instruments.  

## Follow up questions
8 how does it work?
Where else can I apply?
Is it all about dsyms?

## Deeper dive
* Numerous tool build upon symbolciation
* atos
* What do the flags mean?
* Understanding incomplete symbolication
* Utilizing the build settings

1.  Going back to the file
2.  Consult debug information

## Going back to the file
Translate runtime address to file address

Your app has address space on disk.
On-disk space differs from runtime (memory) space
Figure out what the differences are

## On disk address space
* Assigned by linker
* Linker will group file into segments
* Each segment contains related properties, including addresses
* e.g. `__TEXT`.  `__DATA`.
* Info recorded in Mach-O Header
* Load commands for the kernel
* Loads segments into memory
* One header per architecture for Universal 2

`otool -l App | grep L_SEGMENT -A8`

`segname __TEXT
vmaddr <start address>
vmsize <length>
`

Kernel uses ASLR slide.

1.  Generate random value S, the ASLR slide
2.  Segments are added to each address in load commands
3.  Rather than loading text segment at A, it loads at A + S. B+S, etc.

There are known as "Load Addresses"

Go back to linker address by subtracting the ASLR slide.
ASLR slide equals laod aaddress minus linker address
 S = L - A
 
 Go back to file address space once we know slide.
 
 ## Addresses
 
 Query the load commands to know the linker address via otool.
 
 File address comes from the linker
 
 * Load address comes from the running process
 * Reflected in the Binary Images list.
 * Displayed by vmmap


S = 0x10045c00 - 0x100000000
S = 0x45c000
Every runtime address is 0x45c000 bytes away from the linker address.

Subtract slide from backtrace.  Can inspect app to see what resides there.

Can use otool to see instructions.

`otool -tV MyExec -arch arm64`

`brk` signals exception or problem.

atos uses the `-o` -`l` flags for ASLR slide
File segment addresses from `-o`
Load address from `-l`

`vmmap MagicNumbers | grep __TEXT`

 X = 0x4d14000
 
 ## Important lessons
 * Getting the ssame file address is no coincidence
 * Any runtime address resolvable to the file
 * The file addresses are used for debug information

## Recap
* Ap binaries and frameworks are mach-o
* Kernel adds ASLR slide to determine load address
* ASLR slide equation

# Debug information
* Created during build
* Either embedded directly or a separate file
* Varying levels of detail

|                    | Function addresses | Function names | File and line numbers | Optimizations |
|--------------------|--------------------|----------------|-----------------------|---------------|
| Function starts    | yes                |                |                       |               |
| Nlist symbol table | yes                | yes            |                       |               |
| DWARF              | yes                | yes            | yes                   | yes           |

## Function starts
Least sourcecode detail.  Only tells us about first address or start of function.

Encodes list of addreses in `__LINKEDIT` segment.

Described with `_LC_FUNCTION_STARTS`

`symbols -onlyFunctionStartsData -arch arm64 ExecutableName`

Not the most descriptive, but it allows an update to the crashlog.  Can view file addresses as offsets to a function.

First, return to file by subrtracting ASLR slide.  Then, find function starts that could contain file address.  Here, only the first valuecould contain the address, since all the other values are 

Debugger can understand how many bytes into the function you are, what registers were modified etc.

If you encounter a crash log that lacks function names, you're dealing with lowest level fo detail information
* Many opportunities to enrich crashlog with better debug information

## Nlist symbol tables
* Also encodes a list of information in `__LINKEDIT`
* Described by LC_SYMTAB
* These are structs instead of addresses

```swift
struct nlist_64 {
    union {
        uint32_t  n_strx;
    } n_un;
    uint8_t n_type;
    uint8_t n_sect;
    uint16_t n_desc;
    uint64_t n_value; 
};
```

* Functions and methods defined in your code
* Have a name and address in the nlist_64 struct


These are represented by a particular bit pattern for `n_type`.  0x????111?

Also known as `N_SECT`.

`nm -arch arm64 -defined-only --numeric-sort Executable`

NM walks through defined symbols and listed them in address order.

Names appear cryptic.  But names stored in symbol table are mangled.  Help the compiler and linker identify a function.

`| xcrun swift-demangle`

`symbols -arch arm64 -onlyNListData Executable`

Now that we can associate function name to address, we can observe that our fofset expression matches an entry from direct symbols.



`main + 264`.  Not a full view into the sourcecode
Some functions were missing

Why?  Symbol table only has direct symbol entries.

Reserved for functions involved in linking
* Shared across project components
* Exported from a framework
* Necessary data for `dladdr` / `dlsym`.

Gaps in information
* Local, static functions are not represented
* Can omit implementation functions
* Stripped in Release configuration
* Primary app executables left practically empty

Control how stripped
* Strip Linked Product => Should this product be stripped?  If so, we use Strip Style
* Strip Style
	* All symbols = Most invasive; preserve essentials.  Removes direct symbols that are not exported.
	* Non Globals => Remove locally shared functions
	* Debugging symbols => Remove debug nlist entry, keep direct symbols
Strip Swift Symbols



`nm -arch arm64 --defined-only --numeric-sort MyFramework`
Strip non-globals => Only left with interfaces.  Not shared implementation function usedw ithinf ramework.

 stripping all symbols still leaves interface.
 
 ```
 symbols -arch arm64 --onlyNListData MyFramework
 ```
 
 Function starts may include starts for symbols that may have been stripped.  They seem to be missing the function name?
 
 We can determine when working with direct symbols.  Function names, but no line numbers.  Or mix of function names and start addresses.
 
 ## Indirect symbols
 
 The second type of `nlist` struct is an *indirect* symbol.  Signifified with bitfield 0x1 (1 in lsb).  N_EXT.
 
 Using from other libraries, such as print.
 
 `nm -m -arch arm64 --undefined-only --numeric-sort MagicNumbers`
 
 List of addreses – offsets.  
 Nlist symbol tables – entire structs of information, add names to addreses
	 Direct and indirect symbols
	 Reserved for functions involved in linking
	 Stripped build settings influence which symbols are available
	 Both are embedded in `__LINKEDIT`.
	 
 How to get file names and line numbers?
 
 ## DWARF
 
 * Highly detailed and expressive
 * Adds another dimension - relationships
 * Primarily found in dSYM bundles
 * Unlocks the most powerful aspects of symbolication.

## DWARF binary
* Data streams in the `__DWARF` segment
* `debug_info` raw debug data
* `debug_abbrev` structure of the raw data
* `debug_line` file names, line numbers

## Compile unit

Single sourcefile that built the product.  e.g. each swift file.
Assigned attributes and properties.  File, SDKs, text segment ranges

## Subprogram
Represents a defined fucntion or method
Not limited to exported or shared functions
name and `__TEXT` segment range

## Relationships
Subprograms are defined in compile units

Chidlren are the subprograms.

`dwarfdump -v -debug-info -arch arm64 Executable.dSYM`

* Parsed from debug_line
* Defines a "line table program" not a tree
* Generates a sorted list of addresses and source locations

## Traversing the tree with atos

`atos -o Exec.dSYM/Contents/Resources/DWARF/MagicNumbers -arch arm64 -l 0x10045c000 0x10045fb70`

## Inlined functions
Common compiler optimization
Substitutes a function call with the function body

## Inlined subroutine
DWARF type fore presenting inline functions
Subprogram that was inlined into another subprogram
Expressed as a child of the surrounding function

`dwarfdump -v -debug-info -arch arm64 Executable.dSYM`

If there are many inlined, copies, the common properties are stored in the node called the "abstract origin" to avoid duplication.

Callsite => Location in our sourcecode where we wrote the call and it was replaced.  Lets us update our tree with new.

Instructs the tool to consider them during symbolication.  

## Static libraries and object files
* Linked symbols generate "Debugging symbols" nlist entries
* The nlist entries refer to the originating file
* Libraries built with debug information contain DWARF

> If the library was built with debug information, then nlist entry can point us to (library's) DWARF

`dsymutil -deump-debug-map -arch arm64 Executable`

## DWARF recap
* Function and file relationships express detail
* Inlined functions impact the quality of symbolication
* DWARF can be found in dSYMs and static libraries

# Tools and tips
## Build settings
Make sure release is set to "DWARF with dSYM file".

## Finding local dSYMS

`mdfind "com_apple_xcode_dsym_uuids = XXX"

symbols -uuid Foo.dSYM
`


```
dwarfdump -verify Foo.dSYM
```

> If there are any reported errors, please file a bug


DWARF data imited to 4GB
If your dSYM's data exceeds 4GB, consider splitting into components

Compare UUIDs via

`symbols -uuid MagicNumbers
symbols -uuid MagicNumbers.dsym
`
 
 
Tags in square brackets tell you the information source.

If you are confident you have a dsym, but aren't getting symbolicaton, check codesign or entitlements.

`codesign --display -v --entitlements :- MyApp.app`


Specifically, verify that you have a proper codesignature.
Also verify `get-task-allow` entitelement.  Grants permissions to e.g. instruments to symbolicate your app.

If you don't have it enabled, check "Code Sgining Inject Base Entitlements" build setting.

For universal 2 apps, specify the architecture to the tool with `-arch ARCH`.

# Wrap up
* UUID and file addresses are relaible
* Use dSYMs whenever possible
* Tools - atos, nm, otool, symbols, dwarfdump, dsymutil, vmmap

[[Optimizing app startup time - 18]]
[[App startup time past, present, and future - 18]]







# Tools and tips
