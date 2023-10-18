#swift #async 
It's all about the task tree: Find out how structured concurrency can help your apps manage automatic task cancellation, task priority propagation, and useful task-local value patterns. Learn how to manage resources in your app with useful patterns and the latest task group APIs. We'll show you how you can leverage the power of the task tree and task-local values to gain insight into distributed systems. Before watching, review the basics of Swift Concurrency and structured concurrency by checking out “Swift concurrency: Behind the scenes” and “Explore structured concurrency in Swift” from WWDC21. 

see [[Explore Structured Concurrency in Swift]]
[[Swift concurrency Behind the scenes]]

* helps you reason bout concurrent code
* automatically implements advanced behaviors

Not all tasks are structured.
* async let
* task groups
unstructured:
* Task
* Task.detached

Benefits do not always apply to unstructured tasks.

## Soup preparation hierarchies.
consists of multiple steps.  
some dependencies, etc.

###  Unstructured concurrency - 2:27
```swift
func makeSoup(order: Order) async throws -> Soup {
    let boilingPot = Task { try await stove.boilBroth() }
    let choppedIngredients = Task { try await chopIngredients(order.ingredients) }
    let meat = Task { await marinate(meat: .chicken) }
    let soup = await Soup(meat: meat.value, ingredients: choppedIngredients.value)
    return await stove.cook(pot: boilingPot.value, soup: soup, duration: .minutes(10))
}
```

while this expresses which tasks can run concurrently, it's not ideal.

How to express using structured concurrency!

###  Structured concurrency - 2:42
```swift
func makeSoup(order: Order) async throws -> Soup {
    async let pot = stove.boilBroth()
    async let choppedIngredients = chopIngredients(order.ingredients)
    async let meat = marinate(meat: .chicken)
    let soup = try await Soup(meat: meat, ingredients: choppedIngredients)
    return try await stove.cook(pot: pot, soup: soup, duration: .minutes(10))
}
```

Structured rrelationship with their parent task.  
###  Structured concurrency - 3:00
```swift
func chopIngredients(_ ingredients: [any Ingredient]) async -> [any ChoppedIngredient] {
    return await withTaskGroup(of: (ChoppedIngredient?).self,
                               returning: [any ChoppedIngredient].self) { group in
         // Concurrently chop ingredients
         for ingredient in ingredients {
             group.addTask { await chop(ingredient) }
         }
         // Collect chopped vegetables
         var choppedIngredients: [any ChoppedIngredient] = []
         for await choppedIngredient in group {
             if choppedIngredient != nil {
                choppedIngredients.append(choppedIngredient!)
             }
         }
         return choppedIngredients
    }
}
```

uses a task group to chop all of them concurrently.




# Task hierarchy
Arrows point from parent task to child task.  makeSup has 3 child tasks for chopping ingredients, marinating chicken, and boiling broth.

chopIngredients creates a task group.

Parent/child hierarchy forms a tree, the task tree.


# Task cancellation
what causes?
* structured tasks are cancelled implicitly when they go out of scope
* you can call cancelAll to cancel all active children and any future child tasks.

unstructured tasks are cancelled explicitly.  Cancelling parent cancels all children.

Cancellation is cooperative.
###  Task cancellation - 4:32
```swift
func makeSoup(order: Order) async throws -> Soup {
    async let pot = stove.boilBroth()

    guard !Task.isCancelled else {
        throw SoupCancellationError()
    }

    async let choppedIngredients = chopIngredients(order.ingredients)
    async let meat = marinate(meat: .chicken)
    let soup = try await Soup(meat: meat, ingredients: choppedIngredients)
    return try await stove.cook(pot: pot, soup: soup, duration: .minutes(10))
}
```

this simply sets `isCancelled` on the task. Acting on the cancellation is done in your code.  Cancellation is a race, if it's performed before the check vs after.

If we're throwing a cancellationError, use `Task.checkCancellation()`.
###  Task cancellation - 4:58
```swift
func chopIngredients(_ ingredients: [any Ingredient]) async throws -> [any ChoppedIngredient] {
    return try await withThrowingTaskGroup(of: (ChoppedIngredient?).self,
                                   returning: [any ChoppedIngredient].self) { group in
        try Task.checkCancellation()
        
        // Concurrently chop ingredients
        for ingredient in ingredients {
            group.addTask { await chop(ingredient) }
        }

        // Collect chopped vegetables
        var choppedIngredients: [any ChoppedIngredient] = []
        for try await choppedIngredient in group {
            if let choppedIngredient {
                choppedIngredients.append(choppedIngredient)
            }
        }
        return choppedIngredients
    }
}
```

**It's important to check the task cancellation status before starting any expensive work to verify that the result is still necessary.**

Cancellation checking is synchronous.  Anyone that should react to cancellation should check the status before continuing.

Polling for cancellation is fine.  But you may need to respond when the task is suspended and no code is running, *like when implementing an AsyncSequence*.  This is where you want `withTaskCancellationHandler`.

###  Cancellation and async sequences - 5:47
```swift
actor Cook {
    func handleShift<Orders>(orders: Orders) async throws
       where Orders: AsyncSequence,
             Orders.Element == Order {

        for try await order in orders {
            let soup = try await makeSoup(order)
            // ...
        }
    }
}
```

Here we have to use the cancellation handler to detect the event.

###  Cancellation and async sequences - 6:41
```swift
public func next() async -> Order? {
    return await withTaskCancellationHandler {
        let result = await kitchen.generateOrder()
        guard state.isRunning else {
            return nil
        }
        return result
    } onCancel: {
        state.cancel()
    }
}
```

many sequences are implemented with a state machine, which we use to stop the sequence.  Once the task is cancelled, we need to indicate that the sequence is done.

We do this by synchronously calling the cancel function on our state machine.  Because cancellation runs immediately, state is shared mutable state.  We'll need to protect our state machine.  While actors are great for protecting encapsulated state, we want to modify individual properties on our state machine, so this isn't ideal.

We also can't guarantee ordering, to ensure our cancellation will run first.

###  AsyncSequence state machine - 7:40
```swift
private final class OrderState: Sendable {
    let protectedIsRunning = ManagedAtomic<Bool>(true)
    var isRunning: Bool {
        get { protectedIsRunning.load(ordering: .acquiring) }
        set { protectedIsRunning.store(newValue, ordering: .relaxed) }
    }
    func cancel() { isRunning = false }
}
```

can use locks, etc.  Let's try atomics.

* propagates through the task tree
* seamlessly integrates with throwing errors
* event-driven cancellation handlers or explicit polling

# Task priority 
First, what is priority and why do we care?

Your way to communicate to the system how urgent a given task is.  Certain tasks need to run immediately or the app will appear frozen.  Meanwhile other tasks like prefetching content can run in the bg without anyone noticing.

What is an inversion?  HP waits on LP.  By default, child task inherit their priority from their parent...

when HP comes in, we propagate new HP to all child tasks

* lower priority tasks escalate priority when awaited on by a higher priority task
* escalation propagates through the task tree
* priority remains escalated until the task completes
* *not possible to undo a priority escalation*

# Task group patterns
We're creating a lot of chopping tasks.  How to manage concurrency?

If we chop too many things simulatneouls,y we'll run out of space.  Limit the number of ingredients.

###  Limiting concurrency with TaskGroups - 10:55
```swift
func chopIngredients(_ ingredients: [any Ingredient]) async -> [any ChoppedIngredient] {
    return await withTaskGroup(of: (ChoppedIngredient?).self,
                               returning: [any ChoppedIngredient].self) { group in
        // Concurrently chop ingredients
        for ingredient in ingredients {
            group.addTask { await chop(ingredient) }
        }

        // Collect chopped vegetables
        var choppedIngredients: [any ChoppedIngredient] = []
        for await choppedIngredient in group {
            if let choppedIngredient {
                choppedIngredients.append(choppedIngredient)
            }
        }
        return choppedIngredients
    }
}
```

###  Limiting concurrency with TaskGroups - 11:01
```swift
func chopIngredients(_ ingredients: [any Ingredient]) async -> [any ChoppedIngredient] {
    return await withTaskGroup(of: (ChoppedIngredient?).self,
                               returning: [any ChoppedIngredient].self) { group in
        // Concurrently chop ingredients
        let maxChopTasks = min(3, ingredients.count)
        for ingredientIndex in 0..<maxChopTasks {
            group.addTask { await chop(ingredients[ingredientIndex]) }
        }

        // Collect chopped vegetables
        var choppedIngredients: [any ChoppedIngredient] = []
        for await choppedIngredient in group {
            if let choppedIngredient {
                choppedIngredients.append(choppedIngredient)
            }
        }
        return choppedIngredients
    }
}
```
###  Limiting concurrency with TaskGroups - 11:17
```swift
func chopIngredients(_ ingredients: [any Ingredient]) async -> [any ChoppedIngredient] {
    return await withTaskGroup(of: (ChoppedIngredient?).self,
                               returning: [any ChoppedIngredient].self) { group in
        // Concurrently chop ingredients
        let maxChopTasks = min(3, ingredients.count)
        for ingredientIndex in 0..<maxChopTasks {
            group.addTask { await chop(ingredients[ingredientIndex]) }
        }

        // Collect chopped vegetables
        var choppedIngredients: [any ChoppedIngredient] = []
        var nextIngredientIndex = maxChopTasks
        for await choppedIngredient in group {
            if nextIngredientIndex < ingredients.count {
                group.addTask { await chop(ingredients[nextIngredientIndex]) }
                nextIngredientIndex += 1
            }
            if let choppedIngredient {
                choppedIngredients.append(choppedIngredient)
            }
        }
        return choppedIngredients
    }
}
```

distilled:
1.  create up to the max number
2. once max number is running, we wait for one to finish
3. after finishing and haven't stopped, new task to keep making progress
4. limits the number of concurrent tasks in the group.
###  Limiting concurrency with TaskGroups - 11:26
```swift
withTaskGroup(of: Something.self) { group in
    for _ in 0..<maxConcurrentTasks {
        group.addTask { }
    }
    while let <partial result> = await group.next() {
        if !shouldStop { 
            group.addTask { }
        }
    }
}
```

handle shifts.

###  Kitchen Service - 11:56
```swift
func run() async throws {
    try await withThrowingTaskGroup(of: Void.self) { group in
        for cook in staff.keys {
            group.addTask { try await cook.handleShift() }
        }

        group.addTask {
            // keep the restaurant going until closing time
            try await Task.sleep(for: shiftDuration)
        }

        try await group.next()
        // cancel all ongoing shifts
        group.cancelAll()
    }
}
```

New in Swift 5.9, we have withDiscardingTaskGroup.  Discarding task groups odn't hold onto the results of completed. child tasks.  Resources used by tasks are freed immediately after finishing.


###  Introducing DiscardingTaskGroups - 12:41
```swift
func run() async throws {
    try await withThrowingDiscardingTaskGroup { group in
        for cook in staff.keys {
            group.addTask { try await cook.handleShift() }
        }

        group.addTask { // keep the restaurant going until closing time
            try await Task.sleep(for: shiftDuration)
            throw TimeToCloseError()
        }
    }
}
```

these automatically clean up their children, no need to explicitly cancel the group and clean up.  automatic sibling cancellation.  If any child tasks throw an error, all remaining tasks are automatically cancelled.

* DiscardingTaskGroup reelesases resources immediately on task completion
	* unlike normal task groups where you have to collect the result
	* reduces memory when you don't need to return anything
* Use the completion group of one task to signal the creation of the next in a TaskGroup if you don't want a task explosion.

# Task-local values
Like a global variables, but bound to the current task hierarchy.

`@TaskLocal` static var

###  TaskLocal values - 14:10
```swift
actor Kitchen {
    @TaskLocal static var orderID: Int?
    @TaskLocal static var cook: String?
    func logStatus() {
        print("Current cook: \(Kitchen.cook ?? "none")")
    }
}

let kitchen = Kitchen()
await kitchen.logStatus()
await Kitchen.$cook.withValue("Sakura") {
    await kitchen.logStatus()
}
await kitchen.logStatus()
```

good idea to make this optional, any task that does not have this set needs a default value, so nil is nice for that.

Binding tasks for the duration of the scope, and reverts back at the end of the scope.


Looking for the value bound to a TLV involves recursively walking each parent until we find the value.  If we find that, the TLV will assume that value.  If not bound, we get the original default value.

Swift runtime is optimized to run these queries faster, instead of tree-walking, we have a direct reference.

Suppose we want to track the current step.  We can bind the step variable to soup, and then rebind to chop.  Kinda like a stack basically.

Observe orders to ensure they're being completed in a timely manner.  Server environment handles many requests concurrently.

###  Logging - 16:17
```swift
func makeSoup(order: Order) async throws -> Soup {
     log.debug("Preparing dinner", [
       "cook": "\(self.name)",
       "order-id": "\(order.id)",
       "vegetable": "\(vegetable)",
     ])
     // ... 
}

 func chopVegetables(order: Order) async throws -> [Vegetable] {
     log.debug("Chopping ingredients", [
       "cook": "\(self.name)",
       "order-id": "\(order.id)",
       "vegetable": "\(vegetable)",
     ])
     
     async let choppedCarrot = try chop(.carrot)
     async let choppedPotato = try chop(.potato)
     return try await [choppedCarrot, choppedPotato]
}

func chop(_ vegetable: Vegetable, order: Order) async throws -> Vegetable {
    log.debug("Chopping vegetable", [
      "cook": "\(self.name)",
      "order-id": "\(order)",
      "vegetable": "\(vegetable)",
    ])
    // ...
}
```

On apple devices, continue using os-log directly.  But as parts of your app move to the cloud, you need other solutions.

SwiftLog has multiple backing implementations, so you acn drop in a backing that meets your needs.

`Logger.MetadataProvider` abstracts your logging logic to ensure you're emitting consistent information.

It uses a dictionary-like structure, mapping a name to a value.
### MetadataProvider in action - 17:33
```swift
let orderMetadataProvider = Logger.MetadataProvider {
    var metadata: Logger.Metadata =[:code]
    if let orderID = Kitchen.orderID {
        metadata["orderID"] = "\(orderID)"
    }
    return metadata
}
```

Multiple libraries may define their own `MetadataProvider` to look for library-specific values.
###  MetadataProvider in action - 17:50
```swift
let orderMetadataProvider = Logger.MetadataProvider {
    var metadata: Logger.Metadata = [:]
    if let orderID = Kitchen.orderID {
        metadata["orderID"] = "\(orderID)"
    }
    return metadata
}

let chefMetadataProvider = Logger.MetadataProvider {
    var metadata: Logger.Metadata = [:]
    if let chef = Kitchen.chef {
        metadata["chef"] = "\(chef)"
    }
    return metadata
}

let metadataProvider = Logger.MetadataProvider.multiplex([orderMetadataProvider,
                                                          chefMetadataProvider])

LoggingSystem.bootstrap(StreamLogHandler.standardOutput, metadataProvider: metadataProvider)

let logger = Logger(label: "KitchenService")
```

Multiple providers and combines into a singleobject.  Once we have a provider, we're ready to start logging.  The logs automatically include information.

###  Logging with metadata providers - 18:13
```swift
func makeSoup(order: Order) async throws -> Soup {
    logger.info("Preparing soup order")
    async let pot = stove.boilBroth()
    async let choppedIngredients = chopIngredients(order.ingredients)
    async let meat = marinate(meat: .chicken)
    let soup = try await Soup(meat: meat, ingredients: choppedIngredients)
    return try await stove.cook(pot: pot, soup: soup, duration: .minutes(10))
}
```

* attach metadata to the current task
* inherited by child tasks as well as `Task` (not detached)
* Low-level building block for context propagation

# Task traces

Local profiling using Instruments.
[[Visualize and optimize Swift concurrency]]


[[Analyze HTTP traffic in Instruments]]
only shows traces for events happening locally.  Grey box while waiting for response from a server.

Swift Distributed Tracing package.
* provides an extensible instrumentation API
* similar to Swift Log and Swift Metrics
* OpenTelemetry.  

Instead of getting a trace per function, we use `withSpan` to instrument with spans.  Assign names to regions of code that are reported in the tracing system.  Don't need to cover an entire function, provide more insight on specific pieces.

###  Profile server-side execution - 20:30
```swift
func makeSoup(order: Order) async throws -> Soup {
    try await withSpan("makeSoup(\(order.id)") { span in
        async let pot = stove.boilWater()
        async let choppedIngredients = chopIngredients(order.ingredients)
        async let meat = marinate(meat: .chicken)
        let soup = try await Soup(meat: meat, ingredients: choppedIngredients)
        return try await stove.cook(pot: pot, soup: soup, duration: .minutes(10))
    }
}
```

Allow tracing system to merge task trees into a single trace.  Enough information to provide you with insight into the task hierarchy, and runtime performance characteristics, etc.

span name with the function directive, to automatically fill the span name with the function name.
attribute to attach the current orderid.

most spans come with HTTP status codes, request/response, start/end times, etc.

for more info, check out the repo.




###  Profiling server-side execution - 21:36
```swift
func makeSoup(order: Order) async throws -> Soup {
    try await withSpan(#function) { span in
        span.attributes["kitchen.order.id"] = order.id
        async let pot = stove.boilWater()
        async let choppedIngredients = chopIngredients(order.ingredients)
        async let meat = marinate(meat: .chicken)
        let soup = try await Soup(meat: meat, ingredients: choppedIngredients)
        return try await stove.cook(pot: pot, soup: soup, duration: .minutes(10))
    }
}
```

helpful way to track down timing races, etc.

The beautify of the tracing system is that there's nothing more that needs to be done.  If we pull services out of each other, tracing system will automatically pick up the traces and relate them across multiple machines.  Indicate that the spans are running on different machines, but otherwise the same
* server ecosystem-wide effort
* use traces to understand your services
* Task tree used to propagate span metadata

# Wrap up
* automatic cancellation and priority propagation
* task local values
* tracing in distributed systems

hope that's exciting!  reach for structured tasks before unstructured ones!
* https://github.com/apple/swift-distributed-tracing
* 