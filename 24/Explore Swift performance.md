

Discover how Swift balances abstraction and performance. Learn what elements of performance to consider and how the Swift optimizer affects them. Explore the different features of Swift and how they're implemented to further understand the tradeoffs available that can impact performance.

* heap allocation is explicit call to malloc
* local variables are on the stack.

###  An example C function, with self-evident allocation - 0:24
```c
int main(int argc, char **argv) {
  int count = argc - 1;
  int *arr = malloc(count * sizeof(int));
  int i;
  for (i = 0; i < count; ++i) {
    arr[i] = atoi(argv[i + 1]);
  }
  free(arr);
}
```

Swift is not always so simple.  Partly due to increased safety.  But we also have tons of abstraction.

###  An example Swift function, with a lot of implicit abstraction - 0:50
```swift
func main(args: [String]) {
  let arr = args.map { Int($0) ?? 0 }
}
```

They  have nontrivial implementations with costs that aren't as visible as malloc.  that doesn't mean that you can't develop a similar intuition for how your code actually runs.

Let's explore the low-level performance of swift.

# What is performance?

It'd be nice if we just get a number.  That's not how it works.  Performance is multidimensional.

Macroscopic goals
* reduce latency
* reduce power consumption
* stay within memory limits

Often fixed with algorithmic improvements

But sometimes you do need to dig into low level performance.  
Why are we spending so long in this function
why does this allocate?

Principles of microscopic performance
* function calls
* Memory layout
* Memory allocation
* Value copying

Swift comes with a fairly powerful optimizer
Removes a lot of costs you may not even be aware of

Optimization can only do so much
What optimizations are possible?

Monitor your performance
* automated benchmarks
* regular profiling
Catch regressions, no matter where they came from

# Low level principles

## function calls
###  An example of a function call - 4:39
```swift
URLSession.shared.data(for: request)
```

1.  Set up arguments
2. Resolve the address
3. Allocate space for local state
4. Optimization restrictions
\
### argument passing
We have to put arguments in the right place.  On modern processors they're hidden by register renaming so don't make much difference in practice.
But highlevel, may need to match ownership semantics of functions.  Extra retain/releases.

### function resolution, space
Do we know at compile time which function we're calling?  Static vs dynamic dispatch.

Better at processor level but more importantly we can inline and specialize if the compiler can see the function definition.  But DD enables polymorphism.  

In swift only specific kinds of calls use dynamic dispatch.
###  A Swift function that calls a method on a value of protocol type - 6:30
```swift
func updateAll(models: [any DataModel], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

If it's declared on main body of protocol, call uses dynamic dispatch.
###  A declaration of the method where it's a protocol requirement using dynamic dispatch - 6:40
```swift
protocol DataModel {
    func update(from source: DataSource)
}
```

If it's declared in protocol extension, static dispatch.

###  A declaration of the method where it's a protocol extension method using static dispatch - 6:50
```swift
protocol DataModel {
    func update(from source: DataSource, quickly: Bool)
}

extension DataModel {
    func update(from source: DataSource) {
        self.update(from: source, quickly: true)
    }
}
```


Local allocation happens on the C stack, just subtract the stack pointer.

###  The same function as before, which we're now talking about the local state within - 7:00
```swift
func updateAll(models: [any DataModel], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

###  Partial assembly code for that function, showing instructions to adjust the stack pointer - 7:18
```asm
_$s4main9updateAll6models4fromySayAA9DataModel_pG_AA0F6SourceCtF:
    sub   sp, sp, #208
    stp   x29, x30, [sp, #192]
    …
    ldp   x29, x30, [sp, #192]
    add   sp, sp, #208
    ret
```

Call frame - gives function space to execute.  Run body of function.  Add/subtract 208 bytes.

Call frame has a layout like a C struct.  Ideally all local state in the function becomes fields in the callframe.  

###  A C struct showing one possible layout of the function's call frame - 7:59
```c
// sizeof(CallFrame) == 208
struct CallFrame {
    Array<AnyDataModel> models;
    DataSource source;
    AnyDataModel model;
    ArrayIterator iterator;
    ...
    void *savedX29;
    void *savedX30;
};
```

Subtracting a larger constant doesn't take any longer.  Allocating memory in the callframe is as close as it gets to free.
## memory allocation

Three kinds of memory
* global
* stack
* heap
now in the computer they're the same, but we use them in distinct patterns.

allocated/initialized when program loaded.  Almost free.  Only usable for a fixed amount of memory that will never be freed.

* lets and vars declared at global scope
* static stored properties

stack memory
* allocated and freed via C stack pointer
* almost free
* Only useful for memory that does not need to outlive current scope
* local lets/vars
* parameters
* other temporary values

Heap memory
Allocated and freed 
Substantially more expensive

Used for:
class and actor instances
whenever we can't prove a scope restriction is ok

Reference counting
Heap memory often has shared ownership
Managed with reference counting
`retain` means incrementing reference count, release, etc.
## memory layout

"value" is a high-level concept of information content.


###  A line of code containing a single variable initialization - 10:50
```swift
var array = [1.0, 2.0]
```

Inline representation - just the port that you get without following any pointers.

inline of array -> single buffer reference.  Memory layout just doesinline representation.  

###  Using the MemoryLayout type to examine a type's inline representation - 11:44
```swift
MemoryLayout.size(ofValue: array) == 8
```

Every value in swift is logically contained into some context
* a local scope (e.g. local variables, intermediate reuslts of expressions)
* an instance context (e.g. non-static stored properties)
* a global context (e.g. global variables, static stored properties)
* a dynamic context (e.g. buffers managed by Array and Dictionary)


Types and representations
* every value in swift has a static type
* the rules of the type dictate the representation of the value
* the rules of the context provide the memory to hold the inline representation


###  The variable initialization from before, now placed within a function - 12:48
```swift
func makeArray() {
    var array = [1.0, 2.0]
}
```

what is the inline representation of array of double?
well it is the representation of all its stored properties.  At the end of the day, array has a single stored property, a class reference.  

Inline vs out of line storage
Structs, enums, tuples use inline storage
includes the inline representations of all stored properties
classes/actors use out of line storage.

## copying
###  Initializing a second variable with the contents of the first - 15:42
```swift
func makeArray() {
    var array = [1.0, 2.0]
    var array2 = array
}
```

References are managed with reference counting.  So when a container has ownership of array, is that there's an invariant that the array buffer has been retained as part of storing the value into the container.

Balance retain with release.  If nothing else, that has to happen when container goes away.  Container is in local scope, object is released.

In the user of a value or variable, we can consume it, mutate it, borrow it.

Consuming value takes ownership fo the representation.  Assign a value into storage.  Passing a value to a consuming parameter.
Initializing a variable transfers ownership of the value into the variable.

Initialize an existing value -> need to transfer ownership of a value into the new variable
Now the intiail value expression does not produce a new value: refers to existing variable.  Can't just steal the value out of the variable.

We have to copy the value of the old variable, since the value is an array, copying means retaining its buffer.  This is frequently optimized.  If the compiler can see there aren't any more uses, it can transfer instead.

`consume` operator requests this explicitly.  If you try to use the variable after `consume`, it will complain.  

###  Taking the value of an existing variable with the consume operator - 16:27
```swift
func makeArray() {
    var array = [1.0, 2.0]
    var array2 = consume array
}
```

###  A call to a mutating method - 16:58
```swift
func makeArray() {
    var array = [1.0, 2.0]
    array.append(3.0)
}
```

variable still expects to have ownership of the value afterwards.  We transfer ownership of value inside the method.  When the method is done, it transfers ownership of new value back to its variable.

Borrowing.

###  Passing an argument that should be borrowable - 17:40
```swift
func makeArray() {
    var array = [1.0, 2.0]
    print(array)
}
```

ex asserting that nobody else can consume or mutate it.  Passing an argument is the most common situation for borrowing.


Some situations we need to defensively copy arguments instead of borrow them.  We have to prove that there aren't any simultaneous attempts to mutate or consume it.  But in more complex exmamples, we struggle.  ex, class property.

###  Passing an argument that will likely have to be defensively copied - 18:10
```swift
func makeArray(object: MyClass) {
    object.array = [1.0, 2.0]
    print(object.array)
}
```

Swift is actively evolving improvements, both with optimizer and features.

Copying a value means copying the inline representation.

For types using out of line storage eg classes, copy retains the object reference
for types with inline storage (structs) recursively copies the inline representation of all stored properties.

Inline storage
* avoids heap allocations
* great for small types
* copies get more expensive the more properties you have
**No hard and fast rule for optimal performance!**

Large structs


###  Part of a large struct type - 19:27
```swift
struct Person {
    var name: String
    var birthday: Date
    var address: String
    var relationships: [Relationship]
    ...
}
```

some of these fields are object references that have to be retained.
Each copy needs its own storage.  So if we expect to copy it around, we may end up using a lot more memory.  Out of line storage may be preferred.

Out-of-line mutable storage naturally has reference semantics
Mutations to one value are visible in copies of it
Challenging in multithreaded environments


You can still get value semantics using copy on write
Wrap a class reference with a struct
In mutations, copy the object if the reference isn't unique


# Putting it together

## Dynamically-sized types
Many value types reserve the right to add/change their stored properties in a future OS update.  Foundation's URL.  So the layout is unknown at compile time.

A type parameter of generic type can usually be replaced with any type.  

###  A GenericConnection struct that contains a property of an unknown type parameter type - 21:40
```swift
struct GenericConnection<T> {
    var username: String
    var address: T
    var options: [String: String]
}
```

###  The same GenericConnection struct, except with a class constraint on the type parameter - 21:51
```swift
struct GenericConnection<T> where T: AnyObject {
    var username: String
    var address: T
    var options: [String: String]
}
```

When the type parameter is explained to be a class, it has to have a class type.  This can create more efficient code.

###  The same Connection struct as before - 22:27
```swift
struct Connection {
    var username: String
    var address: URL
    var options: [String: String]
}
```

Most containers can adapt dynamically, eg when the value stored is a property of a struct or class
Layout is deferred to runtime
If URL ends up being 24 bytes, the connection will be laid out at runtime, it will layout the same way but we can't use constants.

Somecontainers however must have constant size.
* global memory
* call frames
In that case we must allocate this memory separately from the main memory for the container!

###  A Connection struct that contains a property of the dynamically-sized URL type - 21:22
```swift
struct Connection {
    var username: String
    var address: URL
    var options: [String: String]
}
```

###  A global variable of URL type - 23:23
```swift
var address = URL(string: "...")
```

Compiler creates a global variable pointer type.  As part of its initializer, it will lazily allocate heap space.

###  A local variable of URL type - 23:42
```swift
func workWithAddress() {
    var address = URL(string: "...")
}
```

because callframes must also have a constant size, we need to dynamically do this as well.

Since local variables are scoped, this can still be done on the C stack.  We allocate the callframe as normal, but when variable comes into scope, we subtract from sp again.



## async functions


###  An async function - 25:02
```swift
func awaitAll(tasks: [Task<Int, Never>]) async -> [Int] {
    var results = [Int]()
    for task in tasks {
        results.append(await task.value)
    }
    return results
}
```

Designed to be able to abandon a c thread when they need to suspend
keep local state on a special stack
split functions into partial functions that run between suspensions

the key idea is C threads are a precious resource and we don't want to use them just to block.

async functions conceptually work the same way but they don't allocate out of a large contiguous stack.  Tasks hold onto one or more slabs of memory.  When async fn wants to allocate memory, it asks tasks.  Task tries to satisfy from current slab.  Task will mark that part of the slab as used.

If most of the slab is occupied however, that might not fit.  Task has to allocate a new slab from malloc.  Allocation comes out of that.

In iether case, deallocation just hands the memory back to the task where it is marked as unused.  Because this allocator is only used by a single task and uses a stack discipline, it's much faster than malloc.  Similar performance to synchronous functions, but higher overhead for calls.

Async functions are split into partials.

Only at most one partial function on the C stack.  Run like an ordinary C function until the next suspension point.  Truly local storage can stay in C callframe.

Then we tailcall the next partial.  If it needs to actually suspend, it returns normally, which heads into the concurrency runtime.  

## closures
But how do closures work?

Passed around as values of function type.

###  A function that takes an argument of function type - 28:21
```swift
func sumTwice(f: () -> Int) -> Int {
  return f() + f()
}
```

* function pointer
* context pointer
###  A C function roughly corresponding to the Swift function - 28:30
```c
Int sumTwice(Int (*fFunction)(void *), void *fContext) {
  return fFunction(fContext) + fFunction(fContext);
}
```

we pass the function pointer as an implicit extra argument.

###  A function call that passes a closure expression as a function argument - 28:47
```swift
func sumTwice(f: () -> Int) -> Int {
  return f() + f()
}

func puzzle(n: Int) -> Int {
  return sumTwice { n + 1 }
}
```

here, function is nonescaping function.  As a result, we know that the fn value will not be used after call completes.  So we can allocate context with scoped allocation.

###  C code roughly corresponding to the emission of the non-escaping closure - 29:15
```c
struct puzzle_context {
  Int n;
};

Int puzzle(Int n) {
  struct puzzle_context context = { n };
  return sumTwice(&puzzle_closure, &context);
}

Int puzzle_closure(void *_context) {
  struct puzzle_context *context = (struct puzzle_context *) _context;
  return _context->n + 1;
}
```

In closure function, weknow the type of the context and can pull the content we need.


###  The function and its caller again, now taking an escaping function as its parameter - 29:34
```swift
func sumTwice(f: @escaping () -> Int) -> Int {
  return f() + f()
}

func puzzle(n: Int) -> Int {
  return sumTwice { n + 1 }
}
```

for escaping, we need heap allocation for context, and retain/release.  It's an instance of an anonymous swift class.

In swift when we refer to a local var, we capture by reference.

###  A closure that captures a local variable by reference - 29:53
```swift
func sumTwice(f: () -> Int) -> Int {
  return f() + f()
}

func puzzle(n: Int) -> Int {
  var addend = 0
  return sumTwice {
    addend += 1
    return n + addend
  }
}
```

if var is captured by nonescaping closure, they can just capture a pointer to variable allocation.  But if we capture by escaping closure, the lifetime can be extended for as long as the closure is alive.  So var has to be heap allocated and contain a reference to that object.

###  Swift types roughly approximating how escaping variables and closures are handled - 30:30
```swift
class Box<T> {
  let value: T
}

class puzzle_context {
  let n: Int
  let addend: Box<Int>
}
```
## generics
###  A generic function that calls a protocol requirement - 30:40
```swift
protocol DataModel {
    func update(from source: DataSource)
}

func updateAll<Model: DataModel>(models: [Model], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

Swift protocols are represented at runtime with a table of fps.  One for each requirement in the protocol.


###  A C struct roughly approximating a protocol witness table - 31:03
```c
struct DataModelWitnessTable {
    ConformanceDescriptor *identity;
    void (*update)(DataSource source, TypeMetadata *Self);
};
```

In a generic function, type and witness tables become hidden extra parameters.  
###  A C function signature roughly approximating how generic functions receive generic parameters - 31:20
```c
void updateAll(Array<Model> models, DataSource source, TypeMetadata *Model, DataModelWitnessTable *Model_is_DataModel);
```

When we work with values of protocol type, it's different.  `any`

###  A function that receives an array of values of protocol type - 31:36
```swift
protocol DataModel {
    func update(from source: DataSource)
}

func updateAll(models: [any DataModel], from source: DataSource)
```


###  A C struct roughly approximating the layout of the Swift type `any DataModel` - 31:49
```c
struct AnyDataModel {
    OpaqueValueStorage value;
    TypeMetadata *valueType;
    DataModelWitnessTable *value_is_DataModel;
};

struct OpaqueValueStorage {
    void *storage[3];
};
```

We have storage for value, and fields for value's types and conformances.  But this has to be a fixed size type, it can't change sizes, no matter how large we make the value.  There is potentialyl a data model that won't fit.

Swift uses an arbitrary buffer size of 3 pointer.  If value can fit there, swift puts it there.  Otherwise we allocate space on the heap and just store in the buffer.

First function takes a homogenous array of data models.  Those models are efficiently packed into the array.

Function can be specialized if it knows the type we're calling it with.  Array with known type, optimizer can inline, or produce a specialized version.

###  A contrast of the two Swift function signatures from before - 31:50
```swift
protocol DataModel {
    func update(from source: DataSource)
}

func updateAll<Model: DataModel>(models: [Model], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}

func updateAll(models: [any DataModel], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

###  Specialization of a generic function for known type parameters - 32:57
```swift
func updateAll<Model: DataModel>(models: [Model], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}

var myModels: [MyDataModel]
updateAll(models: myModels, from: source)

// Implicitly generated by the optimizer
func updateAll_specialized(models: [MyDataModel], from source: DataSource) {
    for model in models {
        model.update(from: source)
    }
}
```

Each element has its own dynamic type, values are no longer densely packed.  So you have less help from compiler.

Balancing flexibility and performance
* abstraction is a powerful tool
* ti has costs
* be thoughtful and find the right balance for your code
[[getting started with instruments]]
[[Understanding Swift Performance - 16]]



# Resources
* [Forum: Programming Languages](https://developer.apple.com/forums/topics/programming-languages-topic?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10217/5/8228D59A-1164-48DA-86CD-79F2191061DC/downloads/wwdc2024-10217_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10217/5/8228D59A-1164-48DA-86CD-79F2191061DC/downloads/wwdc2024-10217_sd.mp4?dl=1)
