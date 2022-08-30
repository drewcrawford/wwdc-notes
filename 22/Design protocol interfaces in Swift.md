#swift 

I'm going to pick up where [[Embrace Swift generics]] left off.

# Understand type erasure
we have a data model with a pair of protocols nad 4 concrete types.  2 canimals, 2 foods.

each produces a related type.  To abstract over food production, add `produce` to `Animal`  [[Embrace Swift generics]]

Best way to abstract over return types, use associated types.  This preserves the type relationship.

Protocol `Self` type stands in for the `Animal`.  Associated type for food.

Let's say we have a heterogeneous array.  In [[Embrace Swift generics]] we saw how `any Animal` ahs a box representation which can be any animal dynamically.  This strategy is called type erasure.

`animal.produce()`.  Why does this typecheck?  

```swift
animals.map { animal in //any Animal
	animal.produce()
} //[any Food]
```

type erasure replaces associated types with existential types that have equivalent constraints.

`any Food` is called the *upper bound* of the associated comodity type.  Since `produce` is called on `any Animal`, return type is type erased, giving us `any Food`.

## Trype erasure semantics
Associated types appearing in the *result* of a function declaration are in *producing position*.  Calling the method will produce a value of that type.

When we call on `any Animal` we don't know the result at compile time, but it's some subset of the upper bound.  The actual concrete type that is returned can safely convert to the *upper bound*.  

Associated types apeparing in the *parameter list* of a function declaration are in *consuming position*.

```swift
func eat(_ : FeedType)
```

Need to pass this in.  Since the conversion goes in the other direction, type erasure cannot be performed.  The upper bound existential type does nto convert to the concrete type, becuase it is unknown.

Type erasure does not allow us to work with associated types in consuming position.  instead we must unbox the existential any type by passing to function with opaque `some Type`.

Existing behavior with protocol requirements returning `Self`

```swift
protocol Cloneable: AnyObject {
	func clone() -> Self
}
```

when you call this on `any Cloneable`, the result type itself is type-erased to upper bound, e.g. the protocol itself.  So we get back a new `any Cloneable.`

* `any` now supports protocol with associated types
* Associated types in *producting* position are type erased to their upper bound

# Hide implementation details
How to abstract away concrete result types to separate inferface from details.

add `isHungry` and `feedAnimals()` (on Farm).

```swift
extension Farm {
	var hungryAnimals: [any Animal] {
		animals.filter(\.isHungry)
	}

	func feedAnimals() {
		for animal in hungryAnimals {
			
		}
	}
}
```

Note that this creates temporary arrays, which is inefficient... so...

```swift
extension Farm {
	var hungryAnimals: LazyFilterSequence<[any Animal]> {
		animals.lazy.filter(\.isHungry)
	}

	func feedAnimals() {
		for animal in hungryAnimals {
			
		}
	}
}
```
However, now the type must be decared as this rather complex concrete type.  And we'd rather erase it.  So we can say `some Collection`.  Now clients calling `hungryAnimals` only know they're getting some collection.

However, as written, this hides too much, because we don't know the element type.  We can strike the right balance, by using a constrained opaque result.
`some Collection<any Animal>`

Collection protocol has a single element type.  Once declared with constsrained, opaque result type, the fact that it is actually a lazy sequence is hidden.  But client still has the knowledge that it's a collection, and the element is `any Animal`.  This is precisely the interface that we want.

this works because Collection is

```swift
protocol Collection<Element>: Sequence {
	associatedtype Element
}
```

A **primary associated type**.  The ones that work best, are those that are usually provided by the caller, as opposed to implementation details such as the collection's iterator type.

Often you will see corresopndance between PATs and generic parameter concrete types conforming to the protocol, that is `Array<Element>` `Set<Element>` `Collection<Element>`.

Can be used with both `some` and `any`.  Before Swift 5.7, you would have needed to write your own datatype to write an existential with a specific generic argument.  We build this into the language with constrained existential types.

Let's say we want to let the function decide dynamically between lazy and not.  Using opaque result type produces an error, because it returns two different underlying type.  We can instead return `any`, signaling this API can return different types across calls.  The ability to constraint PATs gives opaque types and existential types a new level of expressivity.  Can be used with various standard library protocols such as `Collection`, or your own protocols.
# Identify type relationships
How to identify and guarantee necessary type relationships between multiple abstract types using related protocol.

```swift
protocol Animal {
	var isHungry: Bool { get }

	associatedtype FeedType: AnimalFeed
	func eat(_ : FeedType)
}
```

```swift
let cow: Cow = ...
let alfalfa = Hay.grow()
let hay = alfalfa.harvest()

cow.eat(hay)
```

similar example with chicken, millet, etc.

I want to abstract over these two sets of related concrete types, so I can implement this once nad have it feed both cows and chickens as well as any new types i might adopt in the future.

`feedAnimal(some Animal)`.  This unboxes to an opaque type.

```swift
protocol AnimalFeed {
	associatedtype CropType: Crop
	static func grow() -> Croptype
}

protocol Crop {
	associatedtype FeedType: AnimalFeed
	func harvest() -> FeedType
}
```

In fact, this back and forth continues forever with an infinite nesting of associated types between `AnimalFeed` and `Crop`.

1.  Self: Crop
2. associated type: feed
3. nested associated crop type
4. etc

```swift
func feedAnimal(_ animal: some Animal) {
	let crop = type(of: animal).FeedType.grow() // (some Animal).FeedType.CropType
	let feed = crop.harvest() //(some Animal).FeedType.CropType.FeedType
	animal.eat(feed) //expects: (some Animal).feedType
}
```

These protcol definitions are too general.  They don't modify the desired relationships.

1.  Hay => alfalfa
2. alfalfa=>Hay

Imagine I'm refactoring my code, and i change the return type here.  After this accidental change, the concrete still satisfy the protocol, even though we violate our desired invariant.  

We have too many distinct associated types.  Two fo these are actually the same concrete type.  This prevents incorrectly written concrete types from conforming to our protocols.

express relationship with `where`.

```swift
protocol AniamlFeed {
	associatedtype CropType: Crop where CropType.FeedType == Self
	static func grow() -> CropType
}
```

static guarantee that two different, possibly nested associated types must be the same concrete type.  This imposes a restriction of the concrete types.

Instead of an infinite tower of nested associated types, I've collapsed all relationships down to a single pair of related types.

Here, crop's feedtype has collapsed down as  well.  We still have one too many associated type.  We want to say that the crop's feedtype's croptype is the same as the one we started with.
```swift
protocol Crop {
	associatedtype FeedType: AnimalFeed where FeedType.cropType == Self
	func harvest() -> FeedType
}
```

Let's look at an associated type diagram which pulls together everything we've seen so far.

![[Screen Shot 2022-06-15 at 6.10.37 PM.png]]

By understanding your data model, we can use same-type requirements to define equivalences between these different nested associated types.  Generic code can rely on these relationships when chaining together multiple ca..s

# Wrap up
* understand when you need type relationships
* Balance API with implementation details
* Identify type relationships across protocols
