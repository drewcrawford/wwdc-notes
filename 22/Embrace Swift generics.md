#swift 

Abstraction separates ideas from specific details.

ex: Factoring code out into a function or local variable.  Details are abstracted away, and code can use the idea without repeating details.

Can also abstract away concrete types.  `<T> where T: Idea`.  

# Model with concrete types
farm example.  More animals, more support for farm, etc.
Could overload `feed` to support each parameter separately.  But each overload will ahve a simple implementation.  This creates extra boilerplate as i add more types of animals.  And it's mostly repeated code.

These are so similar because different types of animals are similar in functionality.
# Identify common capabilities
All animals have the ability to eat some type of food.  Each animal eats differently, so each animal will have differences in behavior.

Allow abstract code to call `eat` and it will behave differently depending on the animal's concrete type.  

*Polymorphism allows one piece of code to have many behaviors.*

Forms of polymorphism:
* function overloading, achieve ad-hoc polymorphism
* Subtypes achieve *subtype* polymorphism
* Generics achieve *parametric* polymorphism

Exampel involving subtyping.  But a few problems, wghat type of food for the abstract class?  Shared state between animal instances?  Subclasses override methods, but might forget to do this and not caught at compile time?  

One approach on the food situation is to accept `Any`.  But this relies on subclass implementations making sure the correct type was passed at runtime.

Instead we could implement type on superclass
```swift
class Animal<Food>
```

this seems a little unnatural because although animals need food to operate, food isn't the core purpose of an animal, and most code that works with animals won't care about food.  Despite this, you need to specify the type.  This could become onerous, etc.

A class is a data type.  And we're trying to convolute a superclass to make it represent abstract ideas.  instead, we want a language construct that was designed to reflect capabilities of type.

* a specific type of food
* operation for consuming food

# Build an interface
A protocol is an abstraction tool that describes the functionality of conforming types.

A protocol separates ideas from implementation details.

an associatedtype serves as placeholder for the concrete type.  Special because they depend on a specific conforming type.  Each instance of a specific type of animal always has the same type of food.

operation for consuming food maps to a method.  Animal types will be required to implement.

Protocols are not limited to classes, so they can be used with structs, enums, and actors.  The compiler will check that the concrete type implements the protocol requirements.

where clauses etc.
Named type parameters and trailing where clauses let you write sophisticated requirements.

But most parameters you can just use ~~impl~~ `some`.  e.g., `some View`.  

An abstract type that represents a placeholder is called an *opaque type*.  The specific type substituted in is the *underlying* type.  For values with *opaque* type the underlying type is fixed.  This way, generic cocde using the value is guaranteed to get the same underlying type each time it's accessed.

opaque types can be used for both inputs and outputs.  So they can be declared in "parameter position" or "result position".  The position of an opaque type determiens wihch part of the program sees the abstract type and which part determines the concrete type.

In general, the part of the program supplying the value decides the underlying type.  The part of the program using the value sees the abstract type.

## Inferring the underlying type for `some`
local variables: underlying type inferred from the rvalue
Underlying type must be fixed for the scope of the variable, so you can't change type of lvalue.
For parameters, inferred from argument value at call value.  
Using `some` in parameter position is inew in swift 5.7.  Only fixed for the scope of that parameter.
For opaque result, udnerlying type is inferred from function implementation.  Scope of this named value is global.  Underlying return type has to be the same across all return statements.  Otherwise, compiler reports an error that the underlying return values have mismatched types.

For swiftui, DSL control flow statements can transform into same type of each branch.  

## heading
If you need the name, you can't use `some`, the type must be named.

because underlying type is fixed, comipler knows relationship between the types.  These static relationships prevent us from feeding the animal the wrong type of food.

[[Design protocol interfaces in Swift]]

## arrays
beware `[some Animal]`, this requires same type.  Instead, use ~~dyn~~  `any Animal`.

Indicates the type can store any arbitrary animal, can vary at runtime.  This is a single static type that has runtime capability to store

special representation in memory, boxed.  Sometimes, a value small enough to fit inside the box directly.  Other values are too large for the box, so the value has to be allocated elsewhere and the box stores a pointer to that value.

*existential type*.  Same representation for different types is *type erasure*.  

type erasure eliminates the type-level distinction between different values.  Different types interchangeably.

heterogeneous array of value types.

Note that we can't use `eat` because we don't know the type (it was erased).  So we can't know what type of feed this animal expects.  To rely on type relationships, you need to get back into a context where the specific type is fixed.  

Now `any Animal` is a different type than `some Animal`, but the compiler can convert them.  To open a type-erased box, (new), think of this as the compiler opening the box and taking out the value stored inside.  For the scope of `some`, it has a fixed underlying type, so we have access to its associated types.

# heading
* some holds a fixed ocncrete type
* guarantees type relationships

any
* holds an arbitrary concrete type
* erases type relationships

Write `some` first, change to `any` when you need it.

> write `some` by default

This workflow is similar to using `let` by default.

# Wrap up
* start with concrete types
* notice reptition
* generalize using a protocol
* write abstract code usign `some` and `any`
* Prefer `some` for more expressive code
[[Design protocol interfaces in Swift]]


# Write generic code

```swift
protocol AnimalFeed {
  associatedtype CropType: Crop where CropType.Feed == Self
  static func grow() -> CropType
}

protocol Crop {
  associatedtype Feed: AnimalFeed where Feed.CropType == Self
  func harvest() -> Feed
}

protocol Animal {
  associatedtype Feed: AnimalFeed
  func eat(_ food: Feed)
}

struct Farm {
  func feed(_ animal: some Animal) {
    let crop = type(of: animal).Feed.grow()
    let produce = crop.harvest()
    animal.eat(produce)
  }

  func feedAll(_ animals: [any Animal]) {
    for animal in animals {
      feed(animal)
    }
  }
}

struct Cow: Animal {
  func eat(_ food: Hay) {}
}

struct Hay: AnimalFeed {
  static func grow() -> Alfalfa {
    Alfalfa()
  }
}

struct Alfalfa: Crop {
  func harvest() -> Hay {
    Hay()
  }
}
```