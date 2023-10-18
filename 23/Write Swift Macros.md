#swift 
Discover how you can use Swift macros to make your codebase more expressive and easier to read. Code along as we explore how macros can help you avoid writing repetitive code and find out how to use them in your app. We'll share the building blocks of a macro, show you how to test it, and take you through how you can emit compilation errors from macros.

# Overview
##  Build a new trait collection instance from scratch - 1:51
```swift
import WWDC

let a = 17
let b = 25

let (result, code) = #stringify(a + b)

print("The value \(result) was produced by the code \"\(code)\"")
```

unlike C macros that are evaluated at the preprocessor stage.  Before typechecking.

freestanding expression - use with expresisons.  Indicated with the hashtag stringify.

To perform the expansion, the macro defines an expansion.  Compiler will send the sourcecode of the macro expansion to that plugin.  Plugin will parse the sourcecode into a swift AST.  This tree is a source-accurate structural representation of the macro.

This generates a tuple.  This is serialized into sourcecode and sends it to the compiler.

# Create a macro

New macro template defines this method we've just seen.

File->New->Package.  Choose swift macro template.
##  Declaration of the stringify macro - 6:31
```swift
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(module: "WWDCMacros", type: "StringifyMacro")
```

tells the compiler that to perform the expansion, it needs to look at the StringifyMacro type in another module.  

How is that defined?


##  Implementation of the stringify macro - 7:10
```swift
public struct StringifyMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) -> ExprSyntax {
        guard let argument = node.argumentList.first?.expression else {
            fatalError("compiler bug: the macro does not have any arguments")
        }

        return "(\(argument), \(literal: argument.description))"
    }
}
```

Conforms to protocol.  It knows that this argument exists, because stringify takes a single parameter.  All arguments 

string interpolation to create syntax tree of the type. First element is the argument itself, and teh second is the string literal.

Note that the function is not returning a string, it's returning an `ExprSyntax`.  Swift can automatically figure this out.

We'll make sure the literal's contents are properly escaped.

Nobody likes bugs, but even worse are bugs we don't see.  Make sure your macros are well-tested. Great way to test them is to write unit tests.

##  Tests for the stringify Macro - 9:12
```swift
final class WWDCTests: XCTestCase {
    func testMacro() {
        assertMacroExpansion(
            """
            #stringify(a + b)
            """,
            expandedSource: """
            (a + b, "a + b")
            """,
            macros: testMacros
        )
    }
}

let testMacros: [String: Macro.Type] = [
    "stringify": StringifyMacro.self
]
```

verify that the stringify macro expands correctly.

It produces a tuple, of the literal value and the string literal.  To tell the test case how to expan dthe macros, we specify the macros.

macro declaration defines the macro's signature
Implementation operates on SwiftSyntax trees
Easy to test



# Macro roles

In which other situations can we use macros?


| role                         | explanation                                                               |
| ---------------------------- | ------------------------------------------------------------------------- |
| `@freestanding(expression)`  | creates a piece of code that returns a value                              |
| `@freestanding(declaration)` | Creates one or more declarations                                          |
| `@attached(peer)`            | Adds new declarations alongside the declaration it's applied to           |
| `@attached(accessor)`        | adds accessors to a property                                              |
| `@attached(memberAttribute)` | adds attributes to the declarations in the type/extension it's applied to |
| `@attached(member)`          | Adds new declarations inside the type/extension it's applied to           |
| `@attached(conformance)`     | Adds conformances to the type/extension applied to                                                                          |

[[Expand on Swift macros]]

## the plan
##  Slope and EasySlope - 12:05
```swift
/// Slopes in my favorite ski resort.
enum Slope {
    case beginnersParadise
    case practiceRun
    case livingRoom
    case olympicRun
    case blackBeauty
}

/// Slopes suitable for beginners. Subset of `Slopes`.
enum EasySlope {
    case beginnersParadise
    case practiceRun

    init?(_ slope: Slope) {
        switch slope {
        case .beginnersParadise: self = .beginnersParadise
        case .practiceRun: self = .practiceRun
        default: return nil
        }
    }

    var slope: Slope {
        switch self {
        case .beginnersParadise: return .beginnersParadise
        case .practiceRun: return .practiceRun
        }
    }
}
```

* declare attached member macro
* Create empty macro implementation
* Create test case
* Write macro implementation
* Integrate macro into app
##  Declare SlopeSubset - 14:16
```swift
/// Defines a subset of the `Slope` enum
///
/// Generates two members:
///  - An initializer that converts a `Slope` to this type if the slope is
///    declared in this subset, otherwise returns `nil`
///  - A computed property `slope` to convert this type to a `Slope`
///
/// - Important: All enum cases declared in this macro must also exist in the
///              `Slope` enum.
@attached(member, names: named(init))
public macro SlopeSubset() = #externalMacro(module: "WWDCMacros", type: "SlopeSubsetMacro")
```


##  Write empty implementation for SlopeSubset - 15:24
```swift
/// Implementation of the `SlopeSubset` macro.
public struct SlopeSubsetMacro: MemberMacro {
    public static func expansion(
        of attribute: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        return []
    }
}
```

##  Register SlopeSubsetMacro in the compiler plugin - 16:23
```swift
@main
struct WWDCPlugin: CompilerPlugin {
    let providingMacros: [Macro.Type] = [
        SlopeSubsetMacro.self
    ]
}
```

##  Test SlopeSubset - 18:41
```swift
let testMacros: [String: Macro.Type] = [
    "SlopeSubset" : SlopeSubsetMacro.self,
]

final class WWDCTests: XCTestCase {
    func testSlopeSubset() {
        assertMacroExpansion(
            """
            @SlopeSubset
            enum EasySlope {
                case beginnersParadise
                case practiceRun
            }
            """, 
            expandedSource: """

            enum EasySlope {
                case beginnersParadise
                case practiceRun
                init?(_ slope: Slope) {
                    switch slope {
                    case .beginnersParadise:
                        self = .beginnersParadise
                    case .practiceRun:
                        self = .practiceRun
                    default:
                        return nil
                    }
                }
            }
            """, 
            macros: testMacros
        )
    }
}
```

##  Cast declaration to an enum declaration - 19:25
```swift
guard let enumDecl = declaration.as(EnumDeclSyntax.self) else {
    // TODO: Emit an error here
    return []
}
```

**set breakpoint inside macro, run tests to stop inside macro implementation**

##  Extract enum members - 21:14
```swift
let members = enumDecl.memberBlock.members
```

##  Load enum cases - 21:32
```swift
let caseDecls = members.compactMap { $0.decl.as(EnumCaseDeclSyntax.self) }
```

##  Retrieve enum elements - 21:58
```swift
let elements = caseDecls.flatMap { $0.elements }
```

##  Generate initializer - 24:11
```swift
let initializer = try InitializerDeclSyntax("init?(_ slope: Slope)") {
    try SwitchExprSyntax("switch slope") {
        for element in elements {
            SwitchCaseSyntax(
                """
                case .\(element.identifier):
                    self = .\(element.identifier)
                """
            )
        }
        SwitchCaseSyntax("default: return nil")
    }
}
```

##  Return generated initializer - 24:19
```swift
return [DeclSyntax(initializer)]
```

##  Apply SlopeSubset to EasySlope - 25:51
```swift
/// Slopes suitable for beginners. Subset of `Slopes`.
@SlopeSubset
enum EasySlope {
    case beginnersParadise
    case practiceRun

    var slope: Slope {
        switch self {
        case .beginnersParadise: return .beginnersParadise
        case .practiceRun: return .practiceRun
        }
    }
}
```

Write amcro
* start with swift macro package template
* Use debugger to explore syntax node structure
* Develop macro based on test cases
* Add package to an Xcode project



# Diagnostics
What if your macro is used in situations that it doesn't support?  Always emit error messages rather than emit code that doesn't compile.  Don't make people read generated code to debug your macro.


##  Test that we generate an error when applying SlopeSubset to a struct - 28:00
```swift
func testSlopeSubsetOnStruct() throws {
    assertMacroExpansion(
        """
        @SlopeSubset
        struct Skier {
        }
        """,
        expandedSource: """

        struct Skier {
        }
        """,
        diagnostics: [
            DiagnosticSpec(message: "@SlopeSubset can only be applied to an enum", line: 1, column: 1)
        ],
        macros: testMacros
    )
}
```



##  Define error to emit when SlopeSubset is applied to a non-enum type - 28:48
```swift
enum SlopeSubsetError: CustomStringConvertible, Error {
    case onlyApplicableToEnum
    
    var description: String {
        switch self {
        case .onlyApplicableToEnum: return "@SlopeSubset can only be applied to an enum"
        }
    }
}
```

##  Throw error if SlopeSubset is applied to a non-enum type - 29:09
```swift
throw SlopeSubsetError.onlyApplicableToEnum
```

Let's make this generic to the superset.

##  Generalize SlopeSubset declaration to EnumSubset - 31:03
```swift
@attached(member, names: named(init))
public macro EnumSubset<Superset>() = #externalMacro(module: "WWDCMacros", type: "SlopeSubsetMacro")
```

access the argument type of the first argument.

##  Retrieve the generic parameter of EnumSubset - 31:33
```swift
guard let supersetType = attribute
    .attributeName.as(SimpleTypeIdentifierSyntax.self)?
    .genericArgumentClause?
    .arguments.first?
    .argumentType else {
    // TODO: Handle error
    return []
}
```

# Summary
* start with macro package template
* Write test cases
* Print syntax tree in debugger
* Write custom error messages
[[Expand on Swift macros]]
