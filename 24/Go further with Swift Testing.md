Learn how to write a sweet set of (test) suites using Swift Testing's baked-in features. Discover how to take the building blocks further and use them to help expand tests to cover more scenarios, organize your tests across different suites, and optimize your tests to run in parallel.

[[Meet Swift Testing]]

Testing is a crucial step in development.  Surcvace issues before code reaches your users, gives you ocnfidence that you're shipping a quality product.  May run into challenges as you maintain your product's tests.

* readability
* code coverage
* organization
* fragility

# Expressive code

## expectations
expect macro is powerful.

Error handling is less tested but important part of experience.  Ensure your code fails gracefully in the face of invalid input and unexpected conditions.  

### Successful throwing function - 0:01

```swift
// Expecting errors
import Testing

@Test
func brewTeaSuccessfully() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    let cupOfTea = try teaLeaves.brew(forMinutes: 3)
}
```

### Validating a successful throwing function - 0:02

```swift
import Testing

@Test
func brewTeaSuccessfully() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    let cupOfTea = try teaLeaves.brew(forMinutes: 3)
    #expect(cupOfTea.quality == .perfect)
}
```

### Validating an error is thrown with do-catch (not recommended) - 0:03

```swift
import Testing

@Test
func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 3)
    do {
        try teaLeaves.brew(forMinutes: 100)
    } catch is BrewingError {
        // This is the code path we
are expecting
    } catch {
        Issue.record("Unexpected Error")
    }
}
```

### Validating a general error is thrown - 0:04

```swift
import Testing

@Test
func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect(throws: (any Error).self) {
        try teaLeaves.brew(forMinutes: 200) // We don't want this to fail the test!
    }
}
```

by using expect(throws:) we solve this.  




### Validating a type of error - 0:05

```swift
import Testing

@Test
func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect(throws: BrewingError.self) {
        try teaLeaves.brew(forMinutes: 200) // We don't want this to fail the test!
    }
}
```

### Validating a specific error - 0:06

```swift
import Testing

@Test
func brewTeaError() throws {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect(throws: BrewingError.oversteeped) {
        try teaLeaves.brew(forMinutes: 200) // We don't want this to fail the test!
    }
}
```

### Complicated validations - 0:07

```swift
import Testing

@Test
func brewTea() {
    let teaLeaves = TeaLeaves(name: "EarlGrey", optimalBrewTime: 4)
    #expect {
        try teaLeaves.brew(forMinutes: 3)
    } throws: { error in
        guard let error = error as? BrewingError,
              case let .needsMoreTime(optimalBrewTime) = error else {
            return false
        }
        return optimalBrewTime == 4
    }
}
```

### Throwing expectation - 0:08

```swift
import Testing

@Test
func brewAllGreenTeas() {
    #expect(throws: BrewingError.self) {
        brewMultipleTeas(teaLeaves: ["Sencha", "EarlGrey", "Jasmine"], time: 2)
    }
}
```


## required expectations
Adding a required expectation lets you end your test early.

### Required expectations - 0:09

```swift
import Testing

@Test
func brewAllGreenTeas() throws {
    try #require(throws: BrewingError.self) {
        brewMultipleTeas(teaLeaves: ["Sencha", "EarlGrey", "Jasmine"], time: 2)
    }
}
```
### Control flow of validating an optional value (not recommended) - 0:10

```swift
import Testing

struct TeaLeaves {
    let name: String
    let optimalBrewTime: Int
    
    func brew(forMinutes minutes: Int) throws -> Tea {
        // ...
    }
}

@Test
func brewTea() throws {
    let teaLeaves = TeaLeaves(name: "Sencha", optimalBrewTime: 2)
    let brewedTea = try teaLeaves.brew(forMinutes: 100)
    
    guard let color = brewedTea.color else {
        Issue.record("Tea color was not available!")
        return
    }
    
    #expect(color == .green)
}
```

## known issues

Your first intuition may be disabled.  But `withKnownIssue` is better, because we know aobut compilation errors?  Failures are expected, so the test will be surfaced as an expected failure.

You'll be notified when issue is fixed, remove `withKnownIssue`.  Test may also perform multiple checks.  just wrap the failing function in `withKnownIssue` and let lest valid



### Failing test with a throwing function - 0:11

```swift
import Testing

@Test
func softServeIceCreamInCone() throws {
    try softServeMachine.makeSoftServe(in: .cone)
}
```

### Disabling a test with a throwing function (not recommended) - 0:12

```swift
import Testing

@Test(.disabled)
func softServeIceCreamInCone() throws {
    try softServeMachine.makeSoftServe(in: .cone)
}
```

### Wrapping a failing test in withKnownIssue - 0:13

```swift
import Testing

@Test
func softServeIceCreamInCone() throws {
    withKnownIssue {
        try softServeMachine.makeSoftServe(in: .cone)
    }
}
```

### Wrap just the failing section in withKnownIssue - 0:14

```swift
import Testing

@Test
func softServeIceCreamInCone() throws {
    let iceCreamBatter = IceCreamBatter(flavor: .chocolate)
    try #require(iceCreamBatter != nil)
    #expect(iceCreamBatter.flavor == .chocolate)
    withKnownIssue {
        try softServeMachine.makeSoftServe(in: .cone)
    }
}
```
## custom test descriptions

In the real world, tests fail.  See at a glance what's going on inside a test.



### Simple enumerations - 0:15

```swift
import Testing

enum SoftServe {
    case vanilla, chocolate, pineapple
}
```

### Complex types - 0:16

```swift
import Testing

struct SoftServe {
    let flavor: Flavor
    let container: Container
    let toppings: [Topping]
}

@Test(arguments: [
    SoftServe(flavor: .vanilla, container: .cone, toppings: [.sprinkles]),
    SoftServe(flavor: .chocolate, container: .cone, toppings: [.sprinkles]),
    SoftServe(flavor: .pineapple, container: .cup, toppings: [.whippedCream])
])
func softServeFlavors(_ softServe: SoftServe) {
    /*...*/
}
```

### Conforming to CustomTestStringConvertible - 0:17

```swift
import Testing

struct SoftServe: CustomTestStringConvertible {
    let flavor: Flavor
    let container: Container
    let toppings: [Topping]
    
    var testDescription: String {
        "\(flavor) in a \(container)"
    }
}

@Test(arguments: [
    SoftServe(flavor: .vanilla, container: .cone, toppings: [.sprinkles]),
    SoftServe(flavor: .chocolate, container: .cone, toppings: [.sprinkles]),
    SoftServe(flavor: .pineapple, container: .cup, toppings: [.whippedCream])
])
func softServeFlavors(_ softServe: SoftServe) {
    /*...*/
}
```

# Parameterized testing

Writing tests for every variation is a maintenance nightmare.  Now you can run a test function with many different arguments.


### An enumeration with a computed property - 0:18

```swift
extension IceCream {
    enum Flavor {
        case vanilla, chocolate, strawberry, mintChip, rockyRoad, pistachio
        
        var containsNuts: Bool {
            switch self {
            case .rockyRoad, .pistachio:
                return true
            default:
                return false
            }
        }
    }
}
```

### A test function for a specific case of an enumeration - 0:19

```swift
import Testing

@Test
func doesVanillaContainNuts() throws {
    try #require(!IceCream.Flavor.vanilla.containsNuts)
}
```

you could write a separate function...
### Separate test functions for all cases of an enumeration - 0:20

```swift
import Testing

@Test
func doesVanillaContainNuts() throws {
    try #require(!IceCream.Flavor.vanilla.containsNuts)
}

@Test
func doesChocolateContainNuts() throws {
    try #require(!IceCream.Flavor.chocolate.containsNuts)
}

@Test
func doesStrawberryContainNuts() throws {
    try #require(!IceCream.Flavor.strawberry.containsNuts)
}

@Test
func doesMintChipContainNuts() throws {
    try #require(!IceCream.Flavor.mintChip.containsNuts)
}

@Test
func doesRockyRoadContainNuts() throws {
    try #require(!IceCream.Flavor.rockyRoad.containsNuts)
}
```

You really only need 1/2 test functions.  Logic of the test is the same except for a single input value.

Test cases are independent and can run in parallel.  re-run individual test cases when type of input conforms to Codable.  Retry individual failing test cases without re-running success.

You might start with a for loop, this is not recommended.

### Parameterizing a test with a for loop (not recommended) - 0:21

```swift
import Testing

extension IceCream {
    enum Flavor {
        case vanilla, chocolate, strawberry, mintChip, rockyRoad, pistachio
    }
}

@Test
func doesNotContainNuts() throws {
    for flavor in [IceCream.Flavor.vanilla, .chocolate, .strawberry, .mintChip] {
        try #require(!flavor.containsNuts)
    }
}
```


Move arguments out to `@Test` attribute.  Testing library will pass each element one at a time to the test function as its first and only argument.
### Swift testing parameterized tests - 0:22

```swift
import Testing

extension IceCream {
    enum Flavor {
        case vanilla, chocolate, strawberry, mintChip, rockyRoad, pistachio
    }
}

@Test(arguments: [IceCream.Flavor.vanilla, .chocolate, .strawberry, .mintChip])
func doesNotContainNuts(flavor: IceCream.Flavor) throws {
    try #require(!flavor.containsNuts)
}
```

now add a second function for the flavors that do contain nuts.
### 100% test coverage - 0:23

```swift
import Testing

extension IceCream {
    enum Flavor {
        case vanilla, chocolate, strawberry, mintChip, rockyRoad, pistachio
    }
}

@Test(arguments: [IceCream.Flavor.vanilla, .chocolate, .strawberry, .mintChip])
func doesNotContainNuts(flavor: IceCream.Flavor) throws {
    try #require(!flavor.containsNuts)
}

@Test(arguments: [IceCream.Flavor.rockyRoad, .pistachio])
func containNuts(flavor: IceCream.Flavor) {
    #expect(flavor.containsNuts)
}
```

any sendable collection - ranges, dictionaries, etc., can be passed.

Displayed in test navigator with each arg with own run button, and rich information view.  
### A parameterized test with one argument - 0:24

```swift
import Testing

enum Ingredient: CaseIterable {
    case rice, potato, lettuce, egg
}

@Test(arguments: Ingredient.allCases)
func cook(_ ingredient: Ingredient) async throws {
    #expect(ingredient.isFresh)
    let result = try cook(ingredient)
    try #require(result.isDelicious)
}
```

test functions can accept multiple inputs, just append after the first argument!
### Adding a second argument to a parameterized test - 0:26

```swift
import Testing

enum Ingredient: CaseIterable {
    case rice, potato, lettuce, egg
}

enum Dish: CaseIterable {
    case onigiri, fries, salad, omelette
}

@Test(arguments: Ingredient.allCases, Dish.allCases)
func cook(_ ingredient: Ingredient, into dish: Dish) async throws {
    #expect(ingredient.isFresh)
    let result = try cook(ingredient)
    try #require(result.isDelicious)
    try #require(result == dish)
}
```

Being able to test every combination is apowerful way to improve your test coverage.  The outer limits of parameterized testing.

As you add more inputs, the test cases will grow, grow, grow, exponentially.  To help control exponential growth, they support a maximum of two collections.  Also can use zip to match up pairs of inputs to go together.
### Using zip() on arguments - 0:28

```swift
import Testing

enum Ingredient: CaseIterable {
    case rice, potato, lettuce, egg
}

enum Dish: CaseIterable {
    case onigiri, fries, salad, omelette
}

@Test(arguments: zip(Ingredient.allCases, Dish.allCases))
func cook(_ ingredient: Ingredient, into dish: Dish) async throws {
    #expect(ingredient.isFresh)
    let result = try cook(ingredient)
    try #require(result.isDelicious)
    try #require(result == dish)
}
```


# Organizing tests

Suites contain test functions.

### Suites - 0:29

```swift
@Suite("Various desserts")
struct DessertTests {
    @Test func applePieCrustLayers() { /* ... */ }
    @Test func lavaCakeBakingTime() { /* ... */ }
    @Test func eggWaffleFlavors() { /* ... */ }
    @Test func cheesecakeBakingStrategy() { /* ... */ }
    @Test func mangoSagoToppings() { /* ... */ }
    @Test func bananaSplitMinimumScoop() { /* ... */ }
}
```

Suites can also contain other suites.



### Nested suites - 0:30

```swift
import Testing

@Suite("Various desserts")
struct DessertTests {
    @Suite
    struct WarmDesserts {
        @Test func applePieCrustLayers() { /* ... */ }
        @Test func lavaCakeBakingTime() { /* ... */ }
        @Test func eggWaffleFlavors() { /* ... */ }
    }

    @Suite
    struct ColdDesserts {
        @Test func cheesecakeBakingStrategy() { /* ... */ }
        @Test func mangoSagoToppings() { /* ... */ }
        @Test func bananaSplitMinimumScoop() { /* ... */ }
    }
}
```

### Separate suites - 0:31

```swift
@Suite
struct DrinkTests {
    @Test func espressoExtractionTime() { /* ... */ }
    @Test func greenTeaBrewTime() { /* ... */ }
    @Test func mochaIngredientProportion() { /* ... */ }
}

@Suite
struct DessertTests {
    @Test func espressoBrownieTexture() { /* ... */ }
    @Test func bungeoppangFilling() { /* ... */ }
    @Test func fruitMochiFlavors() { /* ... */ }
}
```

### Separate suites - 0:32

```swift
// Insert code snippet.
```

Tags can share tests across suites.
### Using a tag - 0:35

```swift
import Testing

extension Tag {
    @Tag static var caffeinated: Self
}

@Suite(.tags(.caffeinated))
struct DrinkTests {
    @Test func espressoExtractionTime() { /* ... */ }
    @Test func greenTeaBrewTime() { /* ... */ }
    @Test func mochaIngredientProportion() { /* ... */ }
}

@Suite
struct DessertTests {
    @Test(.tags(.caffeinated)) func espressoBrownieTexture() { /* ... */ }
    @Test func bungeoppangFilling() { /* ... */ }
    @Test func fruitMochiFlavors() { /* ... */ }
}
```

Tags help associate tests from different files/targets that share things in common.

Declare tags with `@Tag`.
### Declare and use a second tag - 0:36

```swift
import Testing

extension Tag {
    @Tag static var caffeinated: Self
    @Tag static var chocolatey: Self
}

@Suite(.tags(.caffeinated))
struct DrinkTests {
    @Test func espressoExtractionTime() { /* ... */ }
    @Test func greenTeaBrewTime() { /* ... */ }
    @Test(.tags(.chocolatey)) func mochaIngredientProportion() { /* ... */ }
}

@Suite
struct DessertTests {
    @Test(.tags(.caffeinated, .chocolatey)) func espressoBrownieTexture() { /* ... */ }
    @Test func bungeoppangFilling() { /* ... */ }
    @Test func fruitMochiFlavors() { /* ... */ }
}
```

Can filter by tags in bottom left bar.

Can analyze results across groups.

Insights now show common tags.

# Testing in parallel

Parallel testing enabled by default.

Run parallel tests on all physical devices!

## Parallel testing basics

Advantages
1.  Execution time saved

XCTest only supports parallelization with multiple processes, each process running one test a time.  We do everything.  Can isolate to main actor when needed.

### Two tests with an unintended data dependency (not recommended) - 0:37

```swift
import Testing

// ❌ This code is not concurrency-safe.
var cupcake: Cupcake? = nil

@Test
func bakeCupcake() async {
    cupcake = await Cupcake.bake(toppedWith: .frosting)
    // ...
}

@Test
func eatCupcake() async {
    await eat(cupcake!)
    // ...
}
```

Converting older test code, some dependencies may be baked in. Swift 6 can help you find some problems as you rewrite them.  May want to convert your code to Swift testing and address problems later.

### Serialized trait - 0:38

```swift
import Testing

@Suite("Cupcake tests", .serialized)
struct CupcakeTests {
    var cupcake: Cupcake?
    
    @Test func mixingIngredients() { /* ... */ }
    @Test func baking() { /* ... */ }
    @Test func decorating() { /* ... */ }
    @Test func eating() { /* ... */ }
}
```

Indicates test needs to run serially.  

### Serialized trait with nested suites - 0:39

```swift
import Testing

@Suite("Cupcake tests", .serialized)
struct CupcakeTests {
    var cupcake: Cupcake?
    
    @Suite("Mini birthday cupcake tests")
    struct MiniBirthdayCupcakeTests {
        // ...
    }
    
    @Test(arguments: [...])
    func mixing(ingredient: Food) { /* ... */ }
    @Test func baking() { /* ... */ }
    @Test func decorating() { /* ... */ }
    @Test func eating() { /* ... */ }
}
```

We inherit serialized to nested suites.

Swift can still run unrelated tests in parallel with the serialized tests.  We recommend refactoring serial test logic to run in parallel.
On by default
Faster test execution
Swift 6 can help.

## asynchronous conditions
### Using async/await in a test - 0:40

```swift
// Insert code snippet.
```

[[Meet asyncawait in Swift]]
### Using a function with a completion handler in a test (not recommended) - 0:41

```swift
import Testing

@Test
func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    // ❌ This code will run after the test function returns.
    eat(cookies, with: .milk) { result, error in
        #expect(result != nil)
    }
}
```

### Replacing a completion handler with an asynchronous function call - 0:42

```swift
// Insert code snippet.
```

### Using withCheckedThrowingContinuation - 0:43

```swift
import Testing

@Test
func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    try await withCheckedThrowingContinuation { continuation in
        eat(cookies, with: .milk) { result, error in
            if let result {
                continuation.resume(returning: result)
            } else {
                continuation.resume(throwing: error)
            }
        }
    }
}
```


### Callback that invokes more than once (not recommended) - 0:44

```swift
import Testing

@Test
func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    // ❌ This code is not concurrency-safe.
    var cookiesEaten = 0
    
    try await eat(cookies, with: .milk) { cookie, crumbs in
        #expect(!crumbs.in(.milk))
        cookiesEaten += 1
    }
    
    #expect(cookiesEaten == 10)
}
```

confirmation - expected to occur exactly once, but you can specify a different `expectedCount`.
### Confirmations on callbacks that invoke more than once - 0:45

```swift
import Testing

@Test
func bakeCookies() async throws {
    let cookies = await Cookie.bake(count: 10)
    
    try await confirmation("Ate cookies", expectedCount: 10) { ateCookie in
        try await eat(cookies, with: .milk) { cookie, crumbs in
            #expect(!crumbs.in(.milk))
            ateCookie()
        }
    }
}
```

can specify 0 if it shouldn't occur during testing.

### Confirmation that occurs 0 times - 0:46

```swift
// Insert code snippet.
```


# Wrap up
* expressive tests
* Run tests with multiple arguments
* Document and organize tests
* Improve performance and identify dependencies

# Resources
* https://developer.apple.com/documentation/Xcode/adding-tests-to-your-xcode-project
* https://developer.apple.com/documentation/Xcode/organizing-tests-to-improve-feedback
* https://developer.apple.com/documentation/Xcode/running-tests-and-interpreting-results
* https://developer.apple.com/documentation/Testing
* https://github.com/apple/swift-testing
* https://github.com/apple/swift-testing/blob/main/Documentation/Vision.md
