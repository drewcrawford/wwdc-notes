Porting an existing application to use #swift #async 

# CoffeeTracker overview
* UI layer
	* Views
	* complication datasource
* Model layer
	* UI model.  Hold data to be displayed by the UI layer.  `@ObservableObject`.
	* Main thread.
	* Might not be the full model of all your application's data.  Might just be a projection of the data model.  Just waht you need to display on the UI at this moment.
* Backend layer.  Probably in the background, network, etc.  In our case, that's HealthKit controller.

When we layer on concurrency, we get a much messier picture.

Main queue
Arbitrary queue

Simple architectures ignore a lot of hidden complexity about how they handle concurrency.

We put our UIViews and model on the Main Actor.  New actors to operate in the background.  Values to pass between.

We want our *concurrency* architecture to be as clean and easy to describe as our *type* architecture.

[[Meet asyncawait in Swift]]
[[Protect mutable state with Swift actors]]

Let's dive into the code.

# HealthKit controller
ctrl-6: Functions in this file

This code accesses variables.  It looks like we're only reading variables.  Not ok if other code is writing.  Need more information than just the function.  Whatever makes this threadsafe must be elsewhere.  But I can't know this just by looking at this function.

What we call "local reasoning".  Important goal for swift.  e.g., value types you don't have to worry about mutation elsewhere in your program.  Giving you more opportunity to reason locally in your code.

Completion handler with success or error.  But we have to remember to check the status and unwrap.  This isn't normally how we handle errors in swift.  Now we have async functions that can throw.  Reminds us that it's an async function and the code may suspend.  Now we don't have to remember to check the optional error.  

Notice that save no longer returns a value.  Returning success/failure is duplicative with the error.  

We're calling async in a function that does not support concurrency.  To do this, they have a separate way of handling stack frame that isn't compatible with sync functions.  One option is to infect the parent.

New async task.  Will be allowed to call async functions.  Very similar to calling async from a global dispatch queue.  Block executes concurrently.  In this case we're just calling `save` which doesn't return a value.

Need to be careful that you're not touching global state that might get mutated simultaneously from multiple threads.  

Now that we put it inside `async`, this compiles and we finished our first async/await.

# Completion handlers
cmd-shift-A => "add async alternative".  This adds a second version of the fn, which replaces original compleetion with code that creates a new async task and pulls into async version of the function.

Added a deprecation warning.  This will help code refactor.

Inside `requestAuthorization`, callback can happen on an arbitrary thread.  `completionnHandler` might not be threadsafe, but requires global reasoning to find out.

Just because we converted this function to async does not mean we're free from race conditions.  Be aware of the risk of race conditions if you only refactor to introduce async functions.

Note that previously, we forgot to call completion handler through all codepaths through the function.  This was probably a bug and caller was left hanging.  Async catches this type of behavior.  With async functions, you must return a value so this is impossible.  Replace with `return false`.

# Using continuations
Moving to async version.  Optional completion handler analagous to `discardableResult`.

We hit a snag.  Completion handler is on the query type.  But really what we want to await is the *execution* of the query.  I want to create a single async function that both creates the query and executes it.  

Using a continuation.  Query healthkit, async, returns useful values.  Take my querying code and move up into helper function.  Take execution of the query from the bottom.  Continuation helps us invert a completion handler back into an async function.

Use the continuation to replace the completion handlers.  Throw error in case of failure, or resume returning useful values.

Finally, we need to address closure that's still using the completion handler.  Here we use `DispatchQueue.main.async` to get back onto the main thread.  But we ditched out completion handler so tehre's no way of relaying info back to caller.

Global actor called `MainActor` that coordinates all operations on the main thread.  Call `MainActor.run` instead.  Run is an async function so we need to await it.  Awaiting is necessary because the function may need to suspend until the MT is ready to process the operation.  Now we can return. 

Because closures in swift capture variables by reference, capturing a mutable variable creates the possiblity for shared mutable state.  Need to ensure you're making an immutable copy first.

# Using the main actor
`assert(Thread.current == main)`
Limitations
* Forget to put an assert everywhere it's needed
* Can't assert on access to stored properties

Annotate a function with `@MainActor`.  Requires that the caller switch to the main actor before the function is run.  

It's a bit like optionals.  We used to have values you had to check for nil, but easy to forget.  Here we do a similar thing except instead of nil checks, it's enforcing the actor.

If you're outside of an actor, you can run functions on the actor by `await`ing them.

At each await, your function may suspend and other code may run instead.  Sometimes you do want to run multiple calls on the MT.  You might not want the main runloop to turn inbetween operations.  In that case you would want to use MainActor.run to group together calls to the main actor, to ensure you run without suspensions inbetween.  

What about code that works with local variables?  One way is to put the entire HealthKit controller on the main actor.  If I write MainActor, that will protect every method and property of this type, coordinated on the main thread.  For a simple app that might be an ok choice.  But that also seems wrong  This controller is really the backend of the app, it seems unnecessary to do this work on the main thread.

Actor types can be instantiated multiple times.  ex., each room in a chat server can be its own actor.  

Errors are guiding you to places you need to update.  When you get these errors, make sure you understand what they're telling you.  Resist the temptation to mash fixit when you don't understand how or why they fix the issue.

Error cascade.  Instead of feeling overwhelmed, use techniques to keep the change isolated and done one step at a time, compiling and running inbetween.  Add shims to keep code working, even though you might want to delete them later.

First I converted method to async and then actor.  I recommend doing it this way.   

Actor won't let functions not on the actor access that state.  The only state on the actor is the model object.  Everything else is passed in as function arguments.  This suggests that the fn belongs on the model.  

`nonisolated`.  This tells the compiler that the fn isn't part of the actor's protected state so they can be called directly.  The compiler will check that this claim is true.  If I try to access some state of the actor... it won't compile.

Any updates to SwiftUI properties (e.g. `@Published`) must be done on the main queue.  

Consider moving dispatch queues into actors.

Actors relay information by passing values between each other.  So consider taking arguments that are value types.

 1.  handle(backgroundTasks:)
 2.  async loadNewDataFromhealthKit
 3.  sync updateModel
 4.  sync didSet
 5.  sync save
 6.  try to call completion handler in 1

synchronous parts force us to perform synchronous IO on the main thread.  

Completion handlers requires us to visit each method and add a completion handler.  But you can't do that with `didSet` which has no arguments.

Need to make sure our update of fetching a value, awaiting it, reading it back.  Is reentrant.

 Load operation.  Here the logic is split between code that needs to run in the bg, and code that needs to run back on the main thread.  We'll lift the top part of the function into the actor.
 
 `saveValue` is mutated on the main queue, but the save operation is on the background queue.  The kind of assumption that can break in subtle ways.  
 
 We need a way to pass back loaded values, e.g. by returning a value from the `load` function.
 
 Creating a new async task runs on an arbitrary thread.  We shouldn't mutate shared state from an arbitrary thread.  One way to resolve this is to put this task on the main actor.  But there's a better way.
 
 # UI models on the main actor.
 
 The way we do this is to go through our app and add @MainActor to our model type.  
 
 CoffeeData is an ObservableObject, so we want it to use the MainActor.  (e.g. published property).  That must only be updated on the main thread.
 
 You may have noticed to protecting this type on the main actor.
 
 What is it that determines a SwiftUI view is on the main actor?  It's inferreed from the `@EnvironmentObject`.  Anything that has `@EnvironmentObject` or `@ObservedObject` *will always be on the main actor*.  
 
 Elsewhere, we'll also access our model from this extension delegate.  Since this is guaranteed to be called on the main thread, it's annotated by watchkit as running on the main actor.   
 
 
[[Explore Structured Concurrency in Swift]]
[[Swift concurrency Behind the scenes]]

If while you watch this talk you're wondering exactly how this works, check out our under-the-hood talk.
