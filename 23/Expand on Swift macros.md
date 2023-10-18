#swift 
Discover how Swift macros can help you reduce boilerplate in your codebase and adopt complex features more easily. Learn how macros can analyze code, emit rich compiler errors to guide developers towards correct usage, and generate new code that is automatically incorporated back into your project. We'll also take you through important concepts like macro roles, compiler plugins, and syntax trees.

An exciting new feature that lets you customize the swift language.

# Why macros?
## Existing features using expansions (1) - 0:50
```swift
struct Smoothie: Codable {
    var id, title, description: String
    var measuredIngredients: [MeasuredIngredient]

    static let berryBlue =
        Smoothie(id: "berry-blue", title: "Berry Blue") {
            """
            Filling and refreshing, this smoothie \
            will fill you with joy!
            """

            Ingredient.orange
                .measured(with: .cups).scaled(by: 1.5)
            Ingredient.blueberry
                .measured(with: .cups)
            Ingredient.avocado
                .measured(with: .cups).scaled(by: 0.2)
        }
}
```
Codable is kinda macro-like, in that we synthesize some stuff


## Existing features using expansions (2) - 1:11
```swift
struct Smoothie: Codable {
    var id, title, description: String
    var measuredIngredients: [MeasuredIngredient]

        // Begin expansion for Codable
        private enum CodingKeys: String, CodingKey {
            case id, title, description,
                 measuredIngredients
        }

        init(from decoder: Decoder) throws { … }

        func encode(to encoder Encoder) throws { … }
        // End expansion for Codable

    static let berryBlue =
        Smoothie(id: "berry-blue", title: "Berry Blue") {
            """
            Filling and refreshing, this smoothie \
            will fill you with joy!
            """

            Ingredient.orange
                .measured(with: .cups).scaled(by: 1.5)
            Ingredient.blueberry
                .measured(with: .cups)
            Ingredient.avocado
                .measured(with: .cups).scaled(by: 0.2)
        }
}
```

often the compiler expands things.  But what if the existing featuers can't do what you want?  Well you can add a feature to swift, but that's annoying

* eliminate boilerplate
* make tedious things easy
* share with other developers in packages


# Design philosophy
we're doing something different than C macros.
* distinctive use sites
	* freestanding starts with `#` sign.
* attribute macros
	* starts with `@` sign.

macros make existing behavior extensible.  If you don't see those, be confident there are no macros involved.

* complete, type-checked, validated
## Macros inputs are complete, type-checked, and validated - 3:16
```swift
#unwrap(1 + )    // error: expected expression after operator
//arguments have to be complete expressions


//typechecked
@AddCompletionHandler(parameterName: 42)    // error: cannot convert argument of type 'Int' to expected type 'String'
func sendRequest() async throws -> Response



//emit warnings or errors
@DictionaryStorage class Options { … }    // error: '@DictionaryStorage' can only be applied to a 'struct'
```

* inserted in predictable ways

## Macro expansions are inserted in predictable ways - 3:45
```swift
func doThingy() {
    startDoingThingy()

    #someUnknownMacro()

    finishDoingThingy()
}
```

**a macro can only add to visible code, cannot remove or change**

You can still be sure that it doesn't delete the call, or move it into a new function.

* macros are not magic

right click on a macro's use site and see what it expands into.  Set breakpoints, step into, etc.


# Translation model
## How macros work, featuring #stringify - 4:51
```swift
func printAdd(_ a: Int, _ b: Int) {
    let (result, str) = #stringify(a + b)
  
        // Begin expansion for "#stringify"
        (a + b, "a + b")
        // End expansion for "#stringify"
  
    print("\(str) = \(result)")
}

printAdd(1, 2)    // prints "a + b = 3"
```

extracts and sends to comipler plugin.  Runs in separate process in the secure sandbox.  

Swift compiler adds tehe xpansion to your program.  It works justa s through you wrote this yourself, instead of calling the macro.

Important point... how did swift know that the macro exists?

Impossible to write a mcro without specifying role.




## Macro declaration for #stringify - 5:43
```swift
/// Creates a tuple containing both the result of `expr` and its source code represented as a
/// `String`.
@freestanding(expression)
macro stringify<T>(_ expr: T) -> (T, String)
```

Provides the API.  Declaration, import, etc.
# Macro roles

A role is a set of rules for a macro. Governs
* where it can be used
* what tyypes of code it expands into
* where the expansions are inserted

| role                         | explanation                                                               |
| ---------------------------- | ------------------------------------------------------------------------- |
| `@freestanding(expression)`  | creates a piece of code that returns a value                              |
| `@freestanding(declaration)` | Creates one or more declarations                                          |
| `@attached(peer)`            | Adds new declarations alongside the declaration it's applied to           |
| `@attached(accessor)`        | adds accessors to a property                                              |
| `@attached(memberAttribute)` | adds attributes to the declarations in the type/extension it's applied to |
| `@attached(member)`          | Adds new declarations inside the type/extension it's applied to           |
| `@attached(conformance)`     | Adds conformances to the type/extension applied to                                                                          |

## What’s an expression? - 7:11
```swift
let numPixels = (x + width) * (y + height)
//              ^~~~~~~~~~~~~~~~~~~~~~~~~~ This is an expression
//               ^~~~~~~~~                 But so is this
//                   ^~~~~                 And this
```

Freestanding expression macro expands into an expression.  How would you use one?

## The #unwrap expression macro: motivation - 7:34
```swift
// Some teams are nervous about this:
let image = downloadedImage!

// Alternatives are super wordy:
guard let image = downloadedImage else {
    preconditionFailure("Unexpectedly found nil: downloadedImage was already checked")
}
```

let's strike a better balance.
## The #unwrap expression macro: macro declaration - 8:03
```swift
/// Force-unwraps the optional value passed to `expr`.
/// - Parameter message: Failure message, followed by `expr` in single quotes
@freestanding(expression)
macro unwrap<Wrapped>(_ expr: Wrapped?, message: String) -> Wrapped
```

## The #unwrap expression macro: usage - 8:21
```swift
let image = #unwrap(downloadedImage, message: "was already checked")

            // Begin expansion for "#unwrap"
            { [downloadedImage] in
                guard let downloadedImage else {
                    preconditionFailure(
                        "Unexpectedly found nil: ‘downloadedImage’ " + "was already checked",
                        file: "main/ImageLoader.swift",
                        line: 42
                    )
                }
                return downloadedImage
            }()
            // End expansion for "#unwrap"
```

Let's see freestanding declaration.

You want to store elements in a flat 1d array.  1d index from the 2d indexes passed by the developer.

You might write a type like array2d.

## The #makeArrayND declaration macro: motivation - 9:09
```swift
public struct Array2D<Element>: Collection {
    public struct Index: Hashable, Comparable { var storageIndex: Int }
  
    var storage: [Element]
    var width1: Int
  
    public func makeIndex(_ i0: Int, _ i1: Int) -> Index {
        Index(storageIndex: i0 * width1 + i1)
    }
  
    public subscript (_ i0: Int, _ i1: Int) -> Element {
        get { self[makeIndex(i0, i1)] }
        set { self[makeIndex(i0, i1)] = newValue }
    }

    public subscript (_ i: Index) -> Element {
        get { storage[i.storageIndex] }
        set { storage[i.storageIndex] = newValue }
    }

    // Note: Omitted additional members needed for 'Collection' conformance
}

public struct Array3D<Element>: Collection {
    public struct Index: Hashable, Comparable { var storageIndex: Int }
  
    var storage: [Element]
    var width1, width2: Int
  
    public func makeIndex(_ i0: Int, _ i1: Int, _ i2: Int) -> Index {
        Index(storageIndex: (i0 * width1 + i1) * width2 + i2)
    }
  
    public subscript (_ i0: Int, _ i1: Int, _ i2: Int) -> Element {
        get { self[makeIndex(i0, i1, i2)] }
        set { self[makeIndex(i0, i1, i2)] = newValue }
    }
  
    public subscript (_ i: Index) -> Element {
        get { storage[i.storageIndex] }
        set { storage[i.storageIndex] = newValue }
    }
  
    // Note: Omitted additional members needed for 'Collection' conformance
}
```

but then you need a 3d array.  It's a lot like the 2d array, but a little more complex.

## The #makeArrayND declaration macro: macro declaration - 10:03
```swift
/// Declares an `n`-dimensional array type named `Array<n>D`.
/// - Parameter n: The number of dimensions in the array.
@freestanding(declaration, names: arbitrary)
macro makeArrayND(n: Int)
```

## The #makeArrayND declaration macro: usage - 10:15
```swift
#makeArrayND(n: 2)

// Begin expansion for "#makeArrayND"
public struct Array2D<Element>: Collection {
    public struct Index: Hashable, Comparable { var storageIndex: Int }
    var storage: [Element]
    var width1: Int
    public func makeIndex(_ i0: Int, _ i1: Int) -> Index {
        Index(storageIndex: i0 * width1 + i1)
    }
    public subscript (_ i0: Int, _ i1: Int) -> Element {
        get { self[makeIndex(i0, i1)] }
        set { self[makeIndex(i0, i1)] = newValue }
    }
    public subscript (_ i: Index) -> Element {
        get { storage[i.storageIndex] }
        set { storage[i.storageIndex] = newValue }
    }
}
// End expansion for "#makeArrayND"

#makeArrayND(n: 3)
#makeArrayND(n: 4)
#makeArrayND(n: 5)
```

## attached macros

not only variables, fucntions, types, but also imports and operator declarations.  If you use a top level fucntion or type, you can create new toplevel declarations.

say you have various functions, and you have callers that have not adopted async.  You want to vend stubs that can be used for legacy callers.

## The @AddCompletionHandler peer macro: motivation - 11:23
```swift
/// Fetch the avatar for the user with `username`.
func fetchAvatar(_ username: String) async -> Image? {
    ...
}

func fetchAvatar(_ username: String, onCompletion: @escaping (Image?) -> Void) {
    Task.detached { onCompletion(await fetchAvatar(username)) }
}
```



## The @AddCompletionHandler peer macro: macro declaration - 11:51
```swift
/// Overload an `async` function to add a variant that takes a completion handler closure as
/// a parameter.
@attached(peer, names: overloaded)
macro AddCompletionHandler(parameterName: String = "completionHandler")
```

## The @AddCompletionHandler peer macro: usage - 11:59
```swift
/// Fetch the avatar for the user with `username`.
@AddCompletionHandler(parameterName: "onCompletion")
func fetchAvatar(_ username: String) async -> Image? {
    ...
}

    // Begin expansion for "@AddCompletionHandler"

    /// Fetch the avatar for theuser with `username`.
    /// Equivalent to ``fetchAvatar(username:)`` with
    /// a completion handler.
    func fetchAvatar(
        _ username: String,
        onCompletion: @escaping (Image?) -> Void
    ) {
        Task.detached {
            onCompletion(await fetchAvatar(username))
        }
    }

    // End expansion for "@AddCompletionHandler"
```

## accessor macro

## The @DictionaryStorage accessor macro: motivation - 12:36
```swift
struct Person: DictionaryRepresentable {
    init(dictionary: [String: Any]) { self.dictionary = dictionary }
    var dictionary: [String: Any]
  
    var name: String {
        get { dictionary["name"]! as! String }
        set { dictionary["name"] = newValue }
    }
    var height: Measurement<UnitLength> {
        get { dictionary["height"]! as! Measurement<UnitLength> }
        set { dictionary["height"] = newValue }
    }
    var birthDate: Date? {
        get { dictionary["birth_date"] as! Date? }
        set { dictionary["birth_date"] = newValue as Any? }
    }
}
```

## The @DictionaryStorage accessor macro: declaration - 13:04
```swift
/// Adds accessors to get and set the value of the specified property in a dictionary
/// property called `storage`.
@attached(accessor)
macro DictionaryStorage(key: String? = nil)
```
macro generates accessor that you can use to look up value in the dictionary.  Can't do with property wrappers since they can only access the one property.

## The @DictionaryStorage accessor macro: usage - 13:20
```swift
struct Person: DictionaryRepresentable {
    init(dictionary: [String: Any]) { self.dictionary = dictionary }
    var dictionary: [String: Any]
    
    @DictionaryStorage var name: String
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["name"]! as! String }
            set { dictionary["name"] = newValue }
        }
        // End expansion for "@DictionaryStorage"

    @DictionaryStorage var height: Measurement<UnitLength>
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["height"]! as! Measurement<UnitLength> }
            set { dictionary["height"] = newValue }
        }
        // End expansion for "@DictionaryStorage"

    @DictionaryStorage(key: "birth_date") var birthDate: Date?
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["birth_date"] as! Date? }
            set { dictionary["birth_date"] = newValue as Any? }
        }
        // End expansion for "@DictionaryStorage"
}
```

## attached - memberAttribute
Attach to whole type, and can add attributes to the members.

* a macro may have multiple attached roles
	* compose any combination, except 2 freestanding roles, because swift wouldn't know which one to use
* swift will expand all roles applicable where the macro was used
* at least one role must be applicable

## The @DictionaryStorage member attribute macro: macro declaration - 13:56
```swift
/// Adds accessors to get and set the value of the specified property in a dictionary
/// property called `storage`.
@attached(memberAttribute)
@attached(accessor)
macro DictionaryStorage(key: String? = nil)
```

## The @DictionaryStorage member attribute macro: usage - 14:46
```swift
@DictionaryStorage
struct Person: DictionaryRepresentable {
    init(dictionary: [String: Any]) { self.dictionary = dictionary }
    var dictionary: [String: Any]
    
        // Begin expansion for "@DictionaryStorage"
        @DictionaryStorage
        // End expansion for "@DictionaryStorage"
    var name: String
    
        // Begin expansion for "@DictionaryStorage"
        @DictionaryStorage
        // End expansion for "@DictionaryStorage"
    var height: Measurement<UnitLength>

    @DictionaryStorage(key: "birth_date") var birthDate: Date?
}
```

## attached - member


## The @DictionaryStorage member macro: macro definition - 15:52
```swift
/// Adds accessors to get and set the value of the specified property in a dictionary
/// property called `storage`.
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor)
macro DictionaryStorage(key: String? = nil)
```

## The @DictionaryStorage member macro: usage - 16:26
```swift
// The @DictionaryStorage member macro

@DictionaryStorage struct Person: DictionaryRepresentable {
        // Begin expansion for "@DictionaryStorage"
        init(dictionary: [String: Any]) {
            self.dictionary = dictionary
        }
        var dictionary: [String: Any]
        // End expansion for "@DictionaryStorage"
    
    var name: String
    var height: Measurement<UnitLength>
    @DictionaryStorage(key: "birth_date") var birthDate: Date?
}
```

the initializer and stored proeprty are required by dictionaryrepresentable.

You might wonder, which one gets expanded first?  **Each one sees the original version of the declaration without seeing the expansions from the other.**

## conformance





## The @DictionaryStorage conformance macro: macro definition - 16:59
```swift
/// Adds accessors to get and set the value of the specified property in a dictionary
/// property called `storage`.
@attached(conformance)
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor)
macro DictionaryStorage(key: String? = nil)
```

## The @DictionaryStorage conformance macro: usage - 17:09
```swift
struct Person
        // Begin expansion for "@DictionaryStorage"
        : DictionaryRepresentable
        // End expansion for "@DictionaryStorage"
{
    var name: String
    var height: Measurement<UnitLength>
    @DictionaryStorage(key: "birth_date") var birthDate: Date?
}
```

## review


## @DictionaryStorage starting point - 17:28
```swift
struct Person: DictionaryRepresentable {
    init(dictionary: [String: Any]) { self.dictionary = dictionary }
    var dictionary: [String: Any]

    var name: String {
        get { dictionary["name"]! as! String }
        set { dictionary["name"] = newValue }
    }
    var height: Measurement<UnitLength> {
        get { dictionary["height"]! as! Measurement<UnitLength> }
        set { dictionary["height"] = newValue }
    }
    var birthDate: Date? {
        get { dictionary["birth_date"] as! Date? }
        set { dictionary["birth_date"] = newValue as Any? }
    }
}
```

## @DictionaryStorage ending point - 17:32
```swift
@DictionaryStorage
struct Person
        // Begin expansion for "@DictionaryStorage"
        : DictionaryRepresentable
        // End expansion for "@DictionaryStorage"
{
        // Begin expansion for "@DictionaryStorage"
        init(dictionary: [String: Any]) { self.dictionary = dictionary }
        var dictionary: [String: Any]
        // End expansion for "@DictionaryStorage"

        // Begin expansion for "@DictionaryStorage"
        @DictionaryStorage
        // End expansion for "@DictionaryStorage"
    var name: String
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["name"]! as! String }
            set { dictionary["name"] = newValue }
        }
        // End expansion for "@DictionaryStorage"

        // Begin expansion for "@DictionaryStorage"
        @DictionaryStorage
        // End expansion for "@DictionaryStorage"
    var height: Measurement<UnitLength>
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["height"]! as! Measurement<UnitLength> }
            set { dictionary["height"] = newValue }
        }
        // End expansion for "@DictionaryStorage"

    @DictionaryStorage(key: "birth_date")
    var birthDate: Date?
        // Begin expansion for "@DictionaryStorage"
        {
            get { dictionary["birth_date"] as! Date? }
            set { dictionary["birth_date"] = newValue as Any? }
        }
        // End expansion for "@DictionaryStorage"
}
```

## @DictionaryStorage ending point (without expansions) - 17:35
```swift
@DictionaryStorage
struct Person {
    var name: String
    var height: Measurement<UnitLength>

    @DictionaryStorage(key: "birth_date")
    var birthDate: Date?
}
```




# Macro implementation

How do we implement these?

## Macro implementations - 18:01
```swift
/// Creates a tuple containing both the result of `expr` and its source code represented as a
/// `String`.
@freestanding(expression)
macro stringify<T>(_ expr: T) -> (T, String) = #externalMacro(
                                                   module: "MyLibMacros",
                                                   type: "StringifyMacro"
                                               )
```


always another macro.  

Usually, you use an external macro.  This is one that's implemented by a compiler plugin.

What does this look like?

## Implementing @DictionaryStorage’s @attached(member) role (1) - 19:18
```swift
import SwiftSyntax
import SwiftSyntaxMacros
import SwiftSyntaxBuilder

struct DictionaryStorageMacro: MemberMacro {
    static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        return [
           "init(dictionary: [String: Any]) { self.dictionary = dictionary }",
           "var dictionary: [String: Any]"
        ]
    }
}
```

We import `SwiftSyntax`.  package maintained by swift project that lets you parse, inspect, manipulate, etc., swift sourcecode.  We keep it up to date, so it supports every feature the swift compiler does.

Represents sourcecode as a tree structure.  attributes, modifiers, structKeyword, identifier, memberBlock.
Tokens.  Specific piece of text in the source file.  

Some nodes, such as memberDeclBlock, are not tokens.  These have child nodes and their own properties.

ex memberClock has left brace, and right brace, but also members.

For more on SwiftSyntax, see:
[[Write Swift macros]]
documentation for SwiftSyntax.



## Code used to demonstrate SwiftSyntax trees - 19:52
```swift
@DictionaryStorage
struct Person {
    var name: String
    var height: Measurement<UnitLength>
    @DictionaryStorage(key: "birth_date")
    var birthDate: Date?
}
```

SwiftSyntaxMacros - protocols and types for writing macros
SwiftSyntaxBuilder - convenience APIs for constructing syntax trees for representing code.  

## Implementing @DictionaryStorage’s @attached(member) role (2) - 22:00
```swift
import SwiftSyntax
import SwiftSyntaxMacros
import SwiftSyntaxBuilder

struct DictionaryStorageMacro: MemberMacro {
    static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        return [
           "init(dictionary: [String: Any]) { self.dictionary = dictionary }",
           "var dictionary: [String: Any]"
        ]
    }
}
```
Each role has a protocol.  Implementation has to conform to the protocol.  We need 4 corresponding protocols.

methods are static.  All expansion methods are static, so swift doesn't create an instance.  It just uses it as a container for the methods.
Each expansion method returns an array of new members.

This string literal is being written where a DeclSyntax is expected.  So swift treats it as sourcecode, and swift parser turns it into DeclSyntax. 


## compatability
what if we add this to a type that isn't compatible?

## A type that @DictionaryStorage isn’t compatible with - 24:29
```swift
@DictionaryStorage
enum Gender {
    case other(String)
    case female
    case male

        // Begin expansion for "@DictionaryStorage"
        init(dictionary: [String:Any]) { self.dictionary = dictionary }
        var dictionary: [String: Any]
        // End expansion for "@DictionaryStorage"
}
```


Swift will produce an error.  Enums must not contain stored properties.  Great that swift will stop this from compiling, but the error is confusing.

arguments
* attribute - dictionary storage attribute the developer wrote to use the macro
* declaration: DeclGroupSyntax has the type/extension the attribute was applied to
* context: communicate with the compiler
	* emit errors/warnings

1.  Check StructDeclSyntax vs EnumDeclSyntax.
2. Easy way to throw error: just throw swift error.  Doesn't give much control over the output.
3. Create a `Diagnostic`.  This is a bit of compiler jargon.  
	4. Location of the error (syntax node)
	5. Error message and severity
		6. Conform to `DiagnosticMessage`.

Can add fix-its.  Highlights.  Notes, pointing to other locations in the code.  Etc.

## Expansion method with error checking - 25:17
```swift
import SwiftSyntax
import SwiftSyntaxMacros
import SwiftSyntaxBuilder

struct DictionaryStorageMacro: MemberMacro {
    static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        guard declaration.is(StructDeclSyntax.self) else {
            let structError = Diagnostic(
                node: attribute,
                message: MyLibDiagnostic.notAStruct
            )
            context.diagnose(structError)
            return []
        }
        return [
            "init(dictionary: [String: Any]) { self.dictionary = dictionary }",
            "var dictionary: [String: Any]"
        ]
    }
}

enum MyLibDiagnostic: String, DiagnosticMessage {
    case notAStruct

    var severity: DiagnosticSeverity { return .error }

    var message: String {
        switch self {
        case .notAStruct:
            return "'@DictionaryStorage' can only be applied to a 'struct'"
        }
    }

    var diagnosticID: MessageID {
        MessageID(domain: "MyLibMacros", id: rawValue)
    }
}
```


SwiftSyntax initializers and methods


## Parameter list for `ArrayND.makeIndex` - 29:32
```swift
FunctionParameterListSyntax {
    for dimension in 0 ..< numDimensions {
        FunctionParameterSyntax(
            firstName: .wildcardToken(),
            secondName: .identifier("i\(dimension)"),
            type: TypeSyntax("Int")
        )
    }
}
```

This can use a syntaxbuilder to generate whatever number of parameters is apporpriate for the type it's creating.

Literals with interpolations are also supported.

Let's look at interpolation features.



## The #unwrap expression macro: revisited - 30:38
```swift
let image = #unwrap(downloadedImage, message: "was already checked")

            // Begin expansion for "#unwrap"
            { [downloadedImage] in
                guard let downloadedImage else {
                    preconditionFailure(
                        "Unexpectedly found nil: ‘downloadedImage’ " + "was already checked",
                        file: "main/ImageLoader.swift",
                        line: 42
                    )
                }
                return downloadedImage
            }()
            // End expansion for "#unwrap"
```

Let's create a helper fucntion and start adding interpolations so we can parameterize this.

## Implementing the #unwrap expression macro: start - 30:57
```swift
static func makeGuardStmt() -> StmtSyntax {
    return """
        guard let downloadedImage else {
            preconditionFailure(
                "Unexpectedly found nil: ‘downloadedImage’ " + "was already checked",
                file: "main/ImageLoader.swift",
                line: 42
            )
        }
    """
}
```

one interpolation.  Ntoe that we can't add a plain string via interpolation.

## Implementing the #unwrap expression macro: the message string - 30:57
```swift
static func makeGuardStmt(message: ExprSyntax) -> StmtSyntax {
    return """
        guard let downloadedImage else {
            preconditionFailure(
                "Unexpectedly found nil: ‘downloadedImage’ " + \(message),
                file: "main/ImageLoader.swift",
                line: 42
            )
        }
    """
}
```

## Implementing the #unwrap expression macro: the variable name - 31:21
```swift
static func makeGuardStmt(wrapped: TokenSyntax, message: ExprSyntax) -> StmtSyntax {
    return """
        guard let \(wrapped) else {
            preconditionFailure(
                "Unexpectedly found nil: ‘downloadedImage’ " + \(message),
                file: "main/ImageLoader.swift",
                line: 42
            )
        }
    """
}
```

 How to create a string literal that contains a syntax node.

## Implementing the #unwrap expression macro: interpolating a string as a literal - 31:44
```swift
static func makeGuardStmt(wrapped: TokenSyntax, message: ExprSyntax) -> StmtSyntax {
    let messagePrefix = "Unexpectedly found nil: ‘downloadedImage’ "

    return """
        guard let \(wrapped) else {
            preconditionFailure(
                \(literal: messagePrefix) + \(message),
                file: "main/ImageLoader.swift",
                line: 42
            )
        }
    """
}
```

Here we use `literal:`.  SwiftSyntax will add the contents of a string, as a string literal.  Cal also use numbers, optionals, etc.



## Implementing the #unwrap expression macro: adding an expression as a string - 32:11
```swift
static func makeGuardStmt(wrapped: TokenSyntax,
                           originalWrapped: ExprSyntax,
                           message: ExprSyntax) -> StmtSyntax {
    let messagePrefix = "Unexpectedly found nil: ‘\(originalWrapped.description)’ "

    return """
        guard let \(wrapped) else {
            preconditionFailure(
                \(literal: messagePrefix) + \(message),
                file: "main/ImageLoader.swift",
                line: 42
            )
        }
    """
}
```

`literal:` will add escapes, switch to a raw literal, etc.

file/line number.  Compiler doesn't know where we are expanding into.  But the context has an API you can use to generate special syntax nodes that will turn into literals.

## Implementing the #unwrap expression macro: inserting the file and line numbers - 33:00
```swift
static func makeGuardStmt(wrapped: TokenSyntax,
                           originalWrapped: ExprSyntax,
                           message: ExprSyntax,
                           in context: some MacroExpansionContext) -> StmtSyntax {
    let messagePrefix = "Unexpectedly found nil: ‘\(originalWrapped.description)’ "
    let originalLoc = context.location(of: originalWrapped)!

    return """
        guard let \(wrapped) else {
            preconditionFailure(
                \(literal: messagePrefix) + \(message),
                file: \(originalLoc.file),
                line: \(originalLoc.line)
            )
        }
    """
}
```

Note that location will be `nil` if you created it, rather than the one the compiler passed into you.


# Writing correct macros

When we looked at unwrap before, we looked at an exmample where we unwrap a variable name.

What happens if you try to use a variable called `wrappedValue` in the message?  It finds the closer one.  So it will use that, instead of the one you actually meant.

## The #unwrap expression macro, with a name conflict - 34:05
```swift
let wrappedValue = "🎁"
let image = #unwrap(request.downloadedImage, message: "was \(wrappedValue)")

            // Begin expansion for "#unwrap"
            { [wrappedValue = request.downloadedImage] in
                guard let wrappedValue else {
                    preconditionFailure(
                        "Unexpectedly found nil: ‘request.downloadedImage’ " +
                            "was \(wrappedValue)",
                        file: "main/ImageLoader.swift",
                        line: 42
                    )
                }
                return wrappedValue
            }()
            // End expansion for "#unwrap"
```



`context.makeUniqueName()`.  Variable name that is guaranteed not to be used in user code or any other macro expansion.

* swift macros don't prevent name conflicts
	* in rust, we have hygienic macros
* Sometimes you want to access names from outside your macro
* Sometimes you want to introduce new names
	* But you need to declare the names you're adding
* We do that inside the role attributes.

## The MacroExpansion.makeUniqueName() method - 34:30
```swift
let captureVar = context.makeUniqueName()

return  """
        { [\(captureVar) = \(originalWrapped)] in
            \(makeGuardStmt(wrapped: captureVar, …))
            \(makeReturnStmt(wrapped: captureVar))
        }
        """
```

## name specifiers

| name specifier           | meaning                                                                                                                                                         |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| overloaded               | creates a declarationw ith the same base name as the declaration it's attached to (attached only)                                                               |
| prefixed(\< some prefix>) | Creates a declaration whose base name is \<some prefix> followed by the base name of the declaration it's attached to; prefix can start with `$` (attached only) |
| suffixed(< some suffix>)  | Creates a declaration whose base name is the base name of the declaration it's attached to followed by \< some suffix> (attached only)                            |
| arbitrary                | creates a declaration whose name cannot be described by any of the rules above                                                                                                                                                                |

## Declaring a macro’s names - 35:44
```swift
@attached(conformance)
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor)
macro DictionaryStorage(key: String? = nil)



@attached(peer, names: overloaded)
macro AddCompletionHandler(parameterName: String = "completionHandler")



@freestanding(declaration, names: arbitrary)
macro makeArrayND(n: Int)
```


When you can use another specified, please do so.  It makes compiler, code completion, etc., faster.

## don't use outside information

**Only use the information the compiler provides**
**otherwise tools won't know when they need to re-expand a macro**

compiler assumes these are pure functions.  Expansion can't change.  If you circumvent this, you might see inconsistent behavior
Macros don't have access to file systems or network

Sandbox can't stop you, but you still shouldn't
* insert API results like current time, process ID, random numbers
* save information in global variables between expansions

## testing macros

## Macros are testable - 38:28
```swift
import MyLibMacros
import XCTest
import SwiftSyntaxMacrosTestSupport

final class MyLibTests: XCTestCase {
    func testMacro() {
        assertMacroExpansion(
            """
            @DictionaryStorage var name: String
            """,
            expandedSource: """
            var name: String {
                get { dictionary["name"]! as! String }
                set { dictionary["name"] = newValue }
            }
            """,
            macros: ["DictionaryStorage": DictionaryStorageMacro.self])
    }
}
```

**highly recommended!!**

# Wrap up
* macros let you  create language features that 'expand' into complicated code
* Declared in a library, but implemented in a Swift program run in a sandbox
* Roles express what a macro can do
* Write macro tests with XCTest

[[Write Swift macros]]
