#swift 


# What's a DSL?
* Programming language which builds in an assumption about the problem space
* Your code only specifies the custom parts; language adds implicit behavior
* Often declarative

## Embedded DSLs
* Apply custom language rules to code written in a normal language
* Much easier to implement than standalone DSLs
* Interoperates with non-DSL code
* Lets you use existing tools
* Less for clients to learn
* Swift has embedded DSL features

```swift
struct FavoriteSmoothies: View {
    @EnvironmentObject
    private var model: FrutaModel

    var body: some View {
        SmoothieList(smoothies: model.favoriteSmoothies)
            .overlay(
                Group {
                    if model.favoriteSmoothies.isEmpty {
                        Text("Add some smoothies!")
                            .foregroundColor(.secondary)
                            .frame(maxWidth: .infinity,
                                   maxHeight: .infinity) 
                    }
                }
            )
            .navigationTitle("Favorites")
    }
}
```

vs non-DSL
```swift
// Hypothetical code--not actually supported by SwiftUI

struct FavoriteSmoothies: View {
    @EnvironmentObject
    private var model: FrutaModel

    var body: some View {
        var list = SmoothieList(smoothies: model.favoriteSmoothies)
        
        let overlay: View
        if model.favoriteSmoothies.isEmpty {
            var text = Text("Add some smoothies!")
            text.foregroundColor = .secondary

            var frame = Frame(subview: text)
            frame.maxWidth = .infinity
            frame.maxHeight = .infinity
            overlay = frame
        } else {
            overlay = EmptyView()
        }
        
        list.addOverlay(overlay)
        list.navigationTitle = "Favorites"
        
        return list
    }
}
```

## Should you use one?
* When the code is overly cluttered
* When you're describing something, not doing it
* When specialists will use the DSL
* When you'll use the DSL often - like a library
* Remember, languages have a learning curve

* property wrappers
* trailing closure
* result builders
* modifier-style methods

Property wrappers: [[Modern Swift API design - 19]]

# How result builders work
SE-0289
* Help to implement your DSL's domain-specific assumed behavior
* Applied to a function, method, getter, or closure
* Wrap statements in implicit method calls so you can use the results
* Take over function's return value
* Compile-time feature - os version doesn't matter

```swift
VStack /* .init(content: */ {
    Text("Title").font(.title)
    Text("Contents")
} /* ) */


struct VStack<Content: View>: View {
    …
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
}
```

```swift
VStack /* .init(content: */ {
    Text("Title").font(.title)
    Text("Contents")
    /* return // TODO: build results using ‘ViewBuilder’ */
} /* ) */

struct VStack<Content: View>: View {
    …
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
}

@resultBuilder enum ViewBuilder {
    static func buildBlock(_: View...) -> some View { … }
}
```

```swift
VStack /* .init(content: */ {
    /* let v0 = */ Text("Title").font(.title)
    /* let v1 = */ Text("Contents")
    /* return ViewBuilder.buildBlock(v0, v1) */
} /* ) */

struct VStack<Content: View>: View {
    …
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
}

@resultBuilder enum ViewBuilder {
    static func buildBlock(_: View...) -> some View { … }
}
```

Modifier changes the result before result builder sees it

We tried to balance powerful feature vs predictable behavior.

* Syntax in a result builder function is the same
* Looks for names in the normal places
* Result builders disable some language keywords
	* catch, break, etc.
* Some features are enabled conditionally
* If a keyword is permitted, it will work as usual


# Designing a DSL
DSLs are like APIs, only more so
Decide how to express ideas in Swift
Decisions are subjective
Aim for clarity at the point of use
Skills transfer in both directions

```swift
// Fruta’s Smoothie lists

extension Smoothie {
    static let berryBlue = Smoothie(
        id: "berry-blue",
        title: "Berry Blue",
        description: "Filling and refreshing, this smoothie will fill you with joy!",
        measuredIngredients: [
            MeasuredIngredient(.orange, measurement: Measurement(value: 1.5, unit: .cups)),
            MeasuredIngredient(.blueberry, measurement: Measurement(value: 1, unit: .cups)),
            MeasuredIngredient(.avocado, measurement: Measurement(value: 0.2, unit: .cups))
        ],
        hasFreeRecipe: true
    )

    static let carrotChops = Smoothie(…)
    static let crazyColada = Smoothie(…)
    // Plus 12 more…
}

extension Smoothie {
    private static let allSmoothies: [Smoothie] = [
        .berryBlue,
        .carrotChops,
        .crazyColada,
        // Plus 12 more…
    ]

    static func all(includingPaid: Bool = true) -> [Smoothie] {
        if includingPaid {
            return allSmoothies
        }
        
        logger.log("Free smoothies only")
        return allSmoothies.filter { $0.hasFreeRecipe }
    }
}
```

* Eliminate `hasFreeRecipe` property
* merge smoothie definitions into the list
	* This ensures you don't forget to use a temporary variable
* Make ingredient list entries more concise
* Take `description` and `measuredIngredients` out of smoothie argument list

```swift
// DSL top-level design

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) -> [Smoothie] {
    Smoothie(
        // TODO: Change these parameters
        id: "berry-blue",
        title: "Berry Blue",
        description: "Filling and refreshing, this smoothie will fill you with joy!",
        measuredIngredients: [
            Ingredient.orange.measured(with: .cups).scaled(by: 1.5),
            Ingredient.blueberry.measured(with: .cups),
            Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
        ]
    )
  
    Smoothie(…)
  
    if includingPaid {
        Smoothie(…)
       
        Smoothie(…)
    } else {
        logger.log("Free smoothies only")
    }
}
```

.scaled(by:) modifier lets you modify the amounts
Why is this a sepearate modifier?
I realized that I could use the modifier on the whole group.  

```swift
// Possible DSL description/ingredient designs

Smoothie(
    id: "berry-blue",
    title: "Berry Blue",
    description: "Filling and refreshing, this smoothie will fill you with joy!",
    measuredIngredients: [
        Ingredient.orange.measured(with: .cups).scaled(by: 1.5),
        Ingredient.blueberry.measured(with: .cups),
        Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    ]
)
```

```swift
// Possible DSL description/ingredient designs

Smoothie(id: "berry-blue", title: "Berry Blue")
    .description("Filling and refreshing, this smoothie will fill you with joy!")
    .ingredient(Ingredient.orange.measured(with: .cups).scaled(by: 1.5))
    .ingredient(Ingredient.blueberry.measured(with: .cups))
    .ingredient(Ingredient.avocado.measured(with: .cups).scaled(by: 0.2))
```
Would work but wordy.  Easy to forget elements


```swift
// Possible DSL description/ingredient designs

Smoothie {
    ID("berry-blue")
    Title("Berry Blue")
    Description("Filling and refreshing, this smoothie will fill you with joy!")

    Recipe(
        Ingredient.orange.measured(with: .cups).scaled(by: 1.5),
        Ingredient.blueberry.measured(with: .cups),
        Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    )
}
```

This puts ID and title on their own lines which we want to avoid

```swift
// Possible DSL description/ingredient designs

Smoothie(id: "berry-blue", title: "Berry Blue") {
    Description("Filling and refreshing, this smoothie will fill you with joy!")

    Recipe(
        Ingredient.orange.measured(with: .cups).scaled(by: 1.5),
        Ingredient.blueberry.measured(with: .cups),
        Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    )
}
```

Still more ceremony than we need.

Recipes are always presented in a specific order.  Title, description, ingredient list.  We don't bother labeling title and description, it's inherent in the visual hierarchy.

```swift
// Possible DSL description/ingredient designs

Smoothie(id: "berry-blue", title: "Berry Blue") {
    "Filling and refreshing, this smoothie will fill you with joy!"

    Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
    Ingredient.blueberry.measured(with: .cups)
    Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
}
```

Description being below and indented, contains the information of the description string.

## Design tips

* Start with goals
* Look for precedents and inspirations
* Think about fit with the whole
* Try to define away mistakes
* Evaluate many possiblities
* Pick the one you think best


# Writing a result builder
```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildBlock(_ components: Smoothie...) -> [Smoothie] {
    return components
  }
}
```

We choose an enum because it's impossible for someone to use an enum that has no cases.

Recall,

```swift
// How ‘buildBlock(…)’ works

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) {
    /* let v0 = */ Smoothie(id: "berry-blue", title: "Berry Blue") { … }

    /* let v1 = */ Smoothie(id: "carrot-chops", title: "Carrot Chops") { … }

    // …more smoothies…

    /* return SmoothieArrayBuilder.buildBlock(v0, v1, …) */
}
```

So
```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildBlock(_ components: Smoothie...) -> [Smoothie] {
    return components
  }
}
```

New initializer.

```swift
extension Smoothie {
  init(id: Smoothie.ID, title: String, /* FIXME */ _ makeIngredients: () -> (String, [MeasuredIngredient])) {
    let (description, ingredients) = makeIngredients()
    self.init(id: id, title: title, description: description, measuredIngredients: ingredients)
  }
}
```

This is misleading.  Because of another error, swift isn't checking the inside of this closure.  We'll fix this later.

Implement method to add `if` statement.

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    return component ?? []
  }
  
  static func buildBlock(_ components: Smoothie...) -> [Smoothie] {
    return components
  }
}
```


Recall

```swift
// How ‘if’ statements work with ‘buildOptional(_:)’

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) {
    /* let v0 = */ Smoothie(id: "berry-blue", …) { … }
    /* let v1 = */ Smoothie(id: "carrot-chops", …) { … }

    /* let v2: [Smoothie] */
    if includingPaid {
        /* let v2_0 = */ Smoothie(id: "crazy-colada", …) { … }
        /* let v2_1 = */ Smoothie(id: "hulking-lemonade", …) { … }
        /* let v2_block = SmoothieArrayBuilder.buildBlock(v2_0, v2_1)
           v2 = SmoothieArrayBuilder.buildOptional(v2_block) */
    }
    /* else {
        v2 = SmoothieArrayBuilder.buildOptional(nil)
    } */
    
    /* return SmoothieArrayBuilder.buildBlock(v0, v1, v2) */
}
```

Problem here is that if statement produces `[Smoothie]`.  buildBlock doesn't want array of smoothie, it wants single smoothies (variadic).

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    return component ?? []
  }
  
  static func buildBlock(_ components: [Smoothie]...) -> [Smoothie] {
    return components.flatMap { $0 }
  }
}
```

Now we can't use single elements.  

So we need a component type.  buildblock's result type should be an acceptable argument.

Two ways to fix

* Make the parameter type match both `Smoothie` and `[Smoothie]`.  This is how SwiftUI ViewBuilder works.  Everything is a View, including groups etc.
* Convert `Smoothie`s returned by statements into `[Smoothie]`.

Method called `buildExpression`
```swift
// The ‘buildExpression(_:)’ method

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) {
    /* let v0 = SmoothieArrayBuilder.buildExpression( */ Smoothie(id: "berry-blue", …) { … } /* ) */
    /* let v1 = SmoothieArrayBuilder.buildExpression( */ Smoothie(id: "carrot-chops", …) { … } /* ) */

    /* let v2: [Smoothie] */
    if includingPaid {
        /* let v2_0 = SmoothieArrayBuilder.buildExpression( */ Smoothie(id: "crazy-colada", …) { … } /* ) */
        /* let v2_1 = SmoothieArrayBuilder.buildExpression( */ Smoothie(id: "hulking-lemonade", …) { … } /* ) */
        /* let v2_block = SmoothieArrayBuilder.buildBlock(v2_0, v2_1)
           v2 = SmoothieArrayBuilder.buildOptional(v2_block) */
    }
    /* else {
        v2 = SmoothieArrayBuilder.buildOptional(nil)
    } */
    
    /* return SmoothieArrayBuilder.buildBlock(v0, v1, v2) */
}
```

buildOptional only works for plain-if, not else or switch.  

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildEither(first component: [Smoothie]) -> [Smoothie] {
    return component
  }
  
  static func buildEither(second component: [Smoothie]) -> [Smoothie] {
    return component
  }
  
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    return component ?? []
  }
  
  static func buildBlock(_ components: [Smoothie]...) -> [Smoothie] {
    return components.flatMap { $0 }
  }
  
  static func buildExpression(_ expression: Smoothie) -> [Smoothie] {
    return [expression]
  }
}
```
How if-else works

```swift
// How ‘if’-‘else’ statements work with ‘buildEither(…)’

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) -> [Smoothie] {
    /* let v0: [Smoothie] */
    if includingPaid {
        /* let v0_0 = SmoothieArrayBuilder.buildExpression( */ Smoothie(…) { … } /* ) */
        /* let v0_block = SmoothieArrayBuilder.buildBlock(v0_0)
           v0 = SmoothieArrayBuilder.buildEither(first: v0_block) */
    }
    else {
        /* let v0_0 = SmoothieArrayBuilder.buildExpression( */ logger.log("Only got free smoothies!") /* ) */
        /* let v0_block = SmoothieArrayBuilder.buildBlock(v0_0)
           v0 = SmoothieArrayBuilder.buildEither(second: v0_block) */
    }
    
    /* return SmoothieArrayBuilder.buildBlock(v0) */
}
```

Branch that "returns" void `()`.  Need `buildExpression` for that.

```swift
@resultBuilder
enum SmoothieArrayBuilder {
  static func buildEither(first component: [Smoothie]) -> [Smoothie] {
    return component
  }
  
  static func buildEither(second component: [Smoothie]) -> [Smoothie] {
    return component
  }
  
  static func buildOptional(_ component: [Smoothie]?) -> [Smoothie] {
    return component ?? []
  }
  
  static func buildBlock(_ components: [Smoothie]...) -> [Smoothie] {
    return components.flatMap { $0 }
  }
  
  static func buildExpression(_ expression: Smoothie) -> [Smoothie] {
    return [expression]
  }
  
  static func buildExpression(_ expression: Void) -> [Smoothie] {
    return []
  }
}
```

Hooray errors.

Modifiers

```swift
extension Ingredient {
  func measured(with unit: UnitVolume) -> MeasuredIngredient {
    MeasuredIngredient(self, measurement: Measurement(value: 1, unit: unit))
  }
}

extension MeasuredIngredient {
  func scaled(by scale: Double) -> MeasuredIngredient {
    return MeasuredIngredient(ingredient, measurement: measurement * scale)
  }
}
```

```swift
// Closures and result builders

@SmoothieArrayBuilder
static func all(includingPaid: Bool = true) -> [Smoothie] {
    /* let v0 = SmoothieArrayBuilder.buildExpression( */ Smoothie(…) {
        "Filling and refreshing, this smoothie will fill you with joy!"
        
        Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
        Ingredient.blueberry.measured(with: .cups)
        Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    } /* ) */

    /* let v1 = SmoothieArrayBuilder.buildExpression( */ Smoothie(…) {
        "Packed with vitamin A and C, Carrot Chops is a great way to start your day!"
        
        Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
        Ingredient.carrot.measured(with: .cups).scaled(by: 0.5)
        Ingredient.mango.measured(with: .cups).scaled(by: 0.5)
    } /* ) */

    /* return SmoothieArrayBuilder.buildBlock(v0, v1) */
}
```

Resuilt builders only apply to one function
* Not nested functions or closures
* Must apply it to them in some other way
	* Write attribute explicitly
	* Inferring from protocol requirement (`SwiftUI.View.body`)
	* Inferring from closure parameter
	* Disabled by an explicit `return` statement

```swift
extension Smoothie {
  init(id: Smoothie.ID, title: String, @SmoothieBuilder _ makeIngredients: () -> (String, [MeasuredIngredient])) {
    let (description, ingredients) = makeIngredients()
    self.init(id: id, title: title, description: description, measuredIngredients: ingredients)
  }
}

@resultBuilder
enum SmoothieBuilder {
  static func buildBlock(_ description: String, components: MeasuredIngredient...) -> (String, [MeasuredIngredient]) {
    return (description, components)
  }
}
```

```swift
// Accepting different types

Smoothie(…) /* @SmoothieBuilder */ {
    /* let v0 = */ "Filling and refreshing, this smoothie will fill you with joy!"
    /* let v1 = */ Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
    /* let v2 = */ Ingredient.blueberry.measured(with: .cups)
    /* let v3 = */ Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    
    /* return SmoothieBuilder.buildBlock(v0, v1, v2, v3) */
}
```

Maybe first parmater is string, next component is variadic ingredient?

```swift
extension Smoothie {
  init(id: Smoothie.ID, title: String, @SmoothieBuilder _ makeIngredients: () -> (String, [MeasuredIngredient])) {
    let (description, ingredients) = makeIngredients()
    self.init(id: id, title: title, description: description, measuredIngredients: ingredients)
  }
}

@resultBuilder
enum SmoothieBuilder {
  static func buildBlock(_ description: String, components: MeasuredIngredient...) -> (String, [MeasuredIngredient]) {
    return (description, components)
  }
}
```

Additional features
* `for`-`in` loops with `buildArray(_:)`
* Processing return value with `buildFinalResult(_:)`
* More information in the "Attributes" chapter of the swift book, https://docs.swift.org

# Handling invalid code
* Clients will make mistakes - pay attention to the error experience
* Swift will diagnose errors, but they'll be phrased in Swift's terms

```swift
// SmoothieBuilder without the string

Smoothie(…) /* @SmoothieBuilder */ {
    // "Filling and refreshing, this smoothie will fill you with joy!"
    /* let v0 = */ Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
    /* let v1 = */ Ingredient.blueberry.measured(with: .cups)
    /* let v2 = */ Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    
    /* return SmoothieBuilder.buildBlock(v0, v1, v2) */
}

extension SmoothieBuilder {
    static func buildBlock(_ description: String, _ ingredients: ManagedIngredients...) -> (String, [ManagedIngredients]) { … }
}
```

Swift thinks of this error as "you're trying to pass measuredingredient to string".  Not technically wrong, but not helpful.

We make the compiler support the invalid thing, but produce an error when you do it.

```swift
// How Swift improves diagnostics

//values here can be one of
func fn0() throws {}
func fn1() rethrows {}
func fn2() {}

//here we guess you wanted another statement
//Consecutive statements on a line must be separated by `;`
func fn3() deinit {}

//But if you write `try` speicifically, you get a different error.  Compiler suggests `throws`
// Expected throwing speicifier; did you mean `throws`?  Replace `try` with `throws`
func fn4() try {}
```
We special-cased this to emit some undocumented extension.

* function-signature=>parameter-clause `throws` (opt) function-result(opt)
* function-signature =>parameter-clause `rethrows` function-result(opt)
* function-signature=>parameter-clause `try` function-result(opt) **error**
* function-signature =>parameter-clause `throw` function-result(opt) **error**

In oru case, you can create an overload that's unavailable, specifying an error message to use.

```swift
// SmoothieBuilder without the string

Smoothie(…) /* @SmoothieBuilder */ {
    // "Filling and refreshing, this smoothie will fill you with joy!"
    /* let v0 = */ Ingredient.orange.measured(with: .cups).scaled(by: 1.5)
    /* let v1 = */ Ingredient.blueberry.measured(with: .cups)
    /* let v2 = */ Ingredient.avocado.measured(with: .cups).scaled(by: 0.2)
    
    /* return SmoothieBuilder.buildBlock(v0, v1, v2) */
}

extension SmoothieBuilder {
    static func buildBlock(_ description: String, _ ingredients: ManagedIngredients...)
        -> (String, [ManagedIngredients]) { … }

    @available(*, unavailable, message: "missing ‘description’ field")
    static func buildBlock(_ ingredients: ManagedIngredients...)
        -> (String, [ManagedIngredients]) { fatalError() }
}
```

# Wrap up
* DSLs can make complex Swift code much cleaner
* Resuilt bulders capture statment results for a DSL to use
* Modifier-style methods work well with result builders
* Clients will have to learn your language, so use DSLs judiciously

https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html#ID630
https://docs.swift.org/swift-book/ReferenceManual/Attributes.html
https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui
