#swiftui 

Declarative UI framework.  Always be moments that something you don't expect happens.

Helps to understand a bit more about what it's doing behind the scenes.

What does swiftui see?

1.  Identity
2.  Lifetime
3.  Dependencies

# Identity
Elements as the same or distinct across multiple updates

Are these two different dogs?  Or are these two pictures of the same dog?

How swiftui understands your app.

* Two different views? => fade in/out?
* Same view? => transition between states?

Connecting views is important.  How swiftui understands how ot transition.

Key concept behind "View identity".

Views that share the same identity represent differents tate of the same element.

Distinct element: different identity.

Practical impacts of view identity on data and update cycle.

## Types of identity
* Explicit.  Custom, data-driven identifiers
* Structural identity.  By their type, position in the view hierarchy.

### Explicit
Dog names.

Powerful and flexible.  Requires that someone keeps track of the names.

Pointer identity.  Used in UIKit and appkit.  Not used by SwiftUI, but learning it will help you understand how/why it works differently.

Pointer is a natural source of explicit identity.  Refer to individual views using their pointer.  If two views share the sam epointer, we can guarantee they are the same view.

SwiftUI views are value types.  We discussed why value types instead of classes.

* Not allocated, no pointers
* Efficient memory representation
* Supports small, single-purpose components

[[swiftui essentials - 19]]

Relies on other forms of identity.  e.g., `ForEach(a, id: id)`.  

Explicitly identifies the corresopnding view.  SwiftUI can use those ids to understand what exactly changed.

`.id` modifier.  Pass that identifier to the proxy's `scrollTo` method.

Don't have to explicitly identify ever view, just the ones we need.

Just because their identity isn't explicit, doesn't mean they have no identity.

**Every view has identity even if not explicit.**

## Structural identity.

Implicit view identities.

We can identify them based on where they're sitting?

Relative arrangement of subjects to distinguish.  Structural identity.

Structure of conditional statement gives us a clear way to identify each view.

`true` view vs `false` view.

However this only works because SwiftUI statically guarantees.

Type structure of the view hierarchy.  SwiftUI sees the generic types.

`_ConditionalContent<True,False>`.

Powered by `@ViewBuilder`, e.g. a result builder.  Implicitly wraps the body property.  Putting this in the typesystem guarantees that the trueview will always be X and vice versa.

By default, try to preserve identity and provide more fluid transition.  Preserve your view's lifetime and state.

## AnyView

Each conditional branch returns a different view, so I wrapped them all in `AnyView` to get the same returntype fo rthe function.

Unfortunately, SwiftUI can't see the conditional structure and just sees `AnyView`.  This is because anyview is a type-erasing wrapper.

SwiftUI requires a single return type.  

`body` is special, because it's implicitly wrapped in `ViewBuilder`.  Swift odes not infer helper functions to be viewbuilders by default, but we can opt in with `@ViewBuilder` attribute.

Allows us to remove return statement.

If we look at the type signature, it exactly replicates the tree of our code.

Now it's easier to quicly understand the cases of our view.  Our view's typesignature is exactly the same.

We recommend avoiding `AnyView`

* Makes code harder to understand
* Hides static type information, which prevents diagnostic errors
* Worse performance when not needed

Learned how SwiftUI can identify based on type/position.

# Lifetime
Tracks the existence of views and data over time

How identity ties into lifetime of views and data?  Help you better understand how SwiftUI works.

Essence of connecting identity over time.

Just like Theseus can be in different state at different moments in time, our views can be at differents tates throughout their lifetime.

Each state is a different view value.  Identity connects this different value as a single entity/view over time.

These are two distinct values from the same view.  SwiftUI will keep around a copy of the value.. and know if the view has changed.  But after that, the value is destroyed.

*What's important is that the view value is different than the view identity.*  Do not rely on view lifetime.

1.  onAppear (assign identity)
2.  New values are created
3.  From SwiftUI perspective, these represent the same view
4.  Once an entity changes, things end

*A view's lifetime is the duration of the identity*


Let's bring state/StateObject

When SwiftUI sees `@State` or `@StateObject`, it knows to persist that data throughout view lifetime.

This is the persistent storage associated with view's identity.

At the beginning of view's identity, SwiftUI will allocate storage/memory for these.  Using their initial values.

SwiftUI will persist the storage as it gets mutated and the view's body is re-evaluated.


Consider this evaluated first with `true` and then with `false`

```swift
if dayTime {
	//allocates storage
	CatRecorder()
}
else {
	//swiftUI knows this is a different view, so creates new storage
	//storage for 'true view' is deallocated
	CatRecorder().nightTimeStyle()
}
```

If we go back to true branch, that's a new view.

**State lifetime = view lifetime**

Clearly separate the essence of a view (its state) and tie that to its identity.  Everything else is derived from it.  Your data... 

has a set of data-driven constructs that use the identity as a form of explicit identity for views.
* `ForEach`
* `confirmationDialog` / `alert()`
* `List` / `Table` / `OutlineGroup`

## ForEach
```swift
ForEach(0..<5)
```

SwiftUI uses offset here as the identity.  By requiring a constant range, we guarantee that identities are stable for the view lifetime.

In fact, it is an error to use this initializer with a dynamic range.  New this year, you will see a warning with non-constant range.

```swift
ForEach(rescueCats, id: \.tagId)
```

id must be hashable since we use this to assign identity to views generated from collection.

Later, will show you some example of how choosing a stable identity affects performance/correctness.

Providing a stable identity is so important that stdlib defines a protocol `Identifiable`

Take advantage of typesystem to precisely describe the constraint of the problem that we are solving.  

We constrain element of collection to be `Identifiable`.  So that can provide a stable notion of identity.

Very similar to concept of identity/lfietime we discussed earlier.

## Lifetime
A view's value is short-lived
Don't rely on their lifetime!
Lifetime is the duration of its identity
Persistence of state is tied to lifetime
Clearly scope the lfietime of state
Provide a stable identity for your data

# Dependencies
How SwiftUI understand when your interface needs to be updated and why.

How SwiftUI updates the UI.  Goal is to give you a better mental model for how to structure SwiftUI code.

Properties are *dependencies* of the view e.g. *input* to view.  View is required ot produce a new body.

Bod is where you build the hierarchy fo the view.

Actions trigger changes to a view's dependencies.

[[Data Essentials in SwiftUI]]

Notice how our views form a tree.  Dependencies are a tree as well.  However, the dogview is not the only view with dependencies.  In swiftui, each view can have dependencies.  

But multiple views might depend on the same data.  Actually a graph, not a tree.  In fact, we call this the *dependency graph*.

Allows SwiftUI to efficiently update views that require a new body.  

Because views are value types, swiftui can efficiently compare them.  

A view's value is short-lived.  struct vaue is just used for comparison, view itself has longer lifetime.  Avoid generating a new body.

An identity is the backbone of the graph.

Efficiently updates the UI.

Value comparison reduces body generation

## Kinds of dependencies
* Properties of the struct (@Binding, `@Environment`, `@State`, `@StateObject`, `@ObservedObject`, `@EnvironmentObject`)

## Stability
* Directly impacts lifetime
* Helps performance
* Minimizes dependency churn
* Avoids state loss

Indices are *not stable*.  Any persistent identifier is a great choice.

Stability isn't the only property we need.  

## Identifier uniqueness

Each identifer should map to a single view

* Improves animations
* Helps performance
* Correctly reflects dependencies

Need a serial number, etc.

When swiftUI needs an identifier, it needs your help

* Excercise cuation with random identifiers
* Use stable identifiers
* Ensure uniqueness, one identifier per item

Branches are a form of structural identity.  Two copies of the content instead of the same modified copy.

```swift
if date < .now {
	content.opacity(0.3)
}
else {
	content
}
```

Note that in general, you might have branches "across files"

Fold the branches together and move condition insde opacity

`content.opacity(date < .now ? 0.3 : 1.0)`

Inert modifiers: don't affect the rendered result.

Because there's no resulting visual effect, framework can prune the modifier, reducing its cost.

* Avoid unnecessary branches
* Tightly scope dependent code
* Prefer inert modifiers
	* `opacity(1)`
	* `padding`
	* `transformEnvironment`

# Wrap up

* Explicit and structural identity
* Lifetime controls associated storage and durations
* Dependencies represented by a graph

Tips and trick sto avoid bugs and improve performance.

