#swift 
Swift parameter packs are a powerful tool to expand what is possible in your generic code while also enabling you to simplify common generic patterns. We'll show you how to abstract over types as well as the number of arguments in generic code and simplify common generic patterns to avoid overloads. To get the most out of this session, we recommend first checking out “Embrace Swift generics" from WWDC22.

[[Embrace Swift generics]]

# What parameter packs solve

values, types

```swift
func radians(from degrees: Double) -> Double
```

```swift
struct Array<Element>
```

Most generic code abstracts over types and values.

```swift
func query<Payload>(_ item: Request<Payload>) -> Payload
```

variadic parameters
```swift
func query(_ item: Request...)
```

limitations:
* what's the return value?  No way to declare return type that is based on argument length.
* can't accept varying types without type erasure, can't preserve static type information of each type

so we want to preserve that type information

can do it with overloading
```swift
func query<Payload>(_ item: Request<Payload>) -> Payload
```

...but this forces you to choose an upper bound on the number of arguments you support...

```swift
func query<Payload>(
  _ item: Request<Payload>
) -> Payload

func query<Payload1, Payload2>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>
) -> (Payload1, Payload2)

func query<Payload1, Payload2, Payload3>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>,
  _ item3: Request<Payload3>
) -> (Payload1, Payload2, Payload3)

func query<Payload1, Payload2, Payload3, Payload4>(
  _ item1: Request<Payload1>,
  _ item2: Request<Payload2>,
  _ item3: Request<Payload3>,
  _ item4: Request<Payload4>
) -> (Payload1, Payload2, Payload3, Payload4)

let _ = query(r1, r2, r3, r4, r5)
```

Pervasive across APIs that might handle varying number of type parameters.

New in Swift 5.9
# How to read parameter packs

Holds any quantity of types or values, and passes them together.

"Type pack" can hold bool, int, string (3 items)
individual values are the "value pack".  ex `true`, `10`, `""`.

these are used together.  type pack provides the types, value packs provides the value.

Write 1 piece of generic code that works with every element in a pack.

```swift
for request in requests {
  evaluate(request)
}
```

```swift
func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload)
```

repetition pattern (`repeat`).  Indicates that the pattern type will be repeated for each element in the argument path.  the `each` will be passed each version.

ex lets' say our `Payload` is `Bool, Int, String`.  Then we get `Request<Bool>`, `Request<Int>`, `Request<String>`.

note you can only use repetition patterns in positions that accept comma separated lists.
* wrapped in parantheses
	* tuple types
	* single type
* function parameter lists
* generic argument lists

```swift
func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload)

let result = query(Request<Int>())
```

since function parameter and return types, depend on `each Payload`, we know the length of output matches input.

```swift
func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload)

let result = query(Request<Int>())

let results = query(Request<Int>(), Request<String>(), Request<Bool>())
```

conformances

```swift
func query<each Payload: Equatable>(
  _ item: repeat Request<each Payload>
) -> (repeat each Payload)
```

can also use where

```swift
func query<each Payload>(
  _ item: repeat Request<each Payload>
) -> (repeat each Payload)
  where repeat each Payload: Equatable
```

how to require a minimum length?  We want at least 1.  For this we use `FirstPayload` the regular parameter.

Note we want `Equatable` on both.

```swift
func query<FirstPayload, each Payload>(
  _ first: Request<FirstPayload>, _ item: repeat Request<each Payload>
) -> (FirstPayload, repeat each Payload) 
  where FirstPayload: Equatable, repeat each Payload: Equatable
```


# Using parameter packs

here we use `repeat` at the value level (ex in rvalue position)

```swift
struct Request<Payload> {
  func evaluate() -> Payload
}

func query<each Payload>(_ item: repeat Request<each Payload>) -> (repeat each Payload) {
  return (repeat (each item).evaluate())
}
```

some details about this
* if input is empty pack, the result is `()`
* if input is one value, the reuslt will be that element (not in tuple?)
* multiple elements -> tuple

## Enhancing the server query
* add stored state for the query
* differ the input and output types
* manage control flow during parameter pack evaluation

Here we use `each` in type position.  Now we can store the request as a property.

```swift
protocol RequestProtocol {
  associatedtype Input
  associatedtype Output
  func evaluate(_ input: Input) -> Output
}

struct Evaluator<each Request: RequestProtocol> {
  var item: (repeat each Request)

  func query(_ input: repeat (each Request).Input) -> (repeat (each Request).Output) {
    return (repeat (each item).evaluate(each input))
  }
}
```
some do catch thing

```swift
protocol RequestProtocol {
  associatedtype Input
  associatedtype Output
  func evaluate(_ input: Input) throws -> Output
}

struct Evaluator<each Request: RequestProtocol> {
  var item: (repeat each Request)

  func query(_ input: repeat (each Request).Input) -> (repeat (each Request).Output)? {
    do {
      return (repeat try (each item).evaluate(each input))
    } catch {
      return nil
    }
 }
}
```

now any individual query's operation can halt if needed

# wrap up
* abstract over types as well as number of arguments in generic code
* simplify and remove limitation sin your generic code
* iterate parameter packs and perform operations with their elements


# Resources
[[Design protocol interfaces in Swift]]
[[Embrace Swift generics]]

