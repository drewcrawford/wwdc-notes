The techniques that apple frameworks use to improve how they're imported in Swift.

#swift 

# Looking at generated interface.

## for free
NSDate->Date, etc.
init methods ->initializer
rewrote method names
add throwing method

However
## bad
* IUOs
* enums
* lots of stuff
* vanished member
* UInts, etc.
* others I missed


# Provide richer typed information

## Describe nullability to control optionals
Note that ObjC (or C) pointers e.g. `type *` can have a valid value, or they can be null/nil.  However in swift me model this differently.

by default, Swift imports as IUO.

Can specify `nonnull` or `nullable` to indicate if it's optional or not.

```objc
@property (readonly, nullable) NSString *name;
-(nonnull instancetype) initWithName:(NSString*) name;
```

Add `NS_ASSUME_NONNULL_BEGIN` and end once you've worked through the file.  Then delete all the nonull.  This cleans up the file.

Use methods/properties `nonnull` or `nullable`, but in the case of "any pointer", use `_Nonnull` or `_Nullable`.

Careful to get these correctly.

Let's say you have a `nonnull` property that is secretly `nil`.  Well, in string / array case, we generate an empty object.  e.g. `assert(capsule.isEmpty)`.  However if we then pass that object out to an ObjC method again, 

In general, this is UB.

Consider using the static analyzer to catch this.  Analyzer knows about nullable/ nonnullable.

Can also use `Null_unspecified` if you really don't know.  It's like an explicit behavior of a default behavior.

# Use generics to bridge NSArray

# Use Int for numbers
Many objC apis return e.g. UInt for count.  

Swift uses unsigned types for bits.  The main reason for `NSUInteger` is to configure that the value is never negative.  Swift requires you to explicitly convert unsigned types to signed.  This complicates mixing unsigned/signed in the way you would in objc.  Idiomatic swift apis use `Int` even in cases when they are negative.

In Foundation, we apply a blanket rule to use `NSInteger` regardless of whether a number can be negative or not.  We recommend you do this as well.

# Strengthen stringly-typed constants
Use of `NS_STRING_ENUM` means we import as a struct.
```swift
typedef NSString *SKRocket NS_STRING_ENUM;
SKRocket const Myconst;
SKRocket const Myconst2;

NSInteger SKRocketStageCount(SKrocket);
```

# Specify initializer behavior
Note that we get a `()` initiazer from NSObject.

use `NS_DESIGNATED_INITIALIZER`.

* Subclasses must override designated initalizer
* Can override convenience initializers

Note that Swift and objc's idea of initializer is similar.

Note that in ObjC, it's a convention, whereas in Swift, it's a language rule
Some classes follows in bojc, all non-final classes follow in Swift

Which initializer should be desiganted?

Designated call `[super init]`
convenience inits call `[self init]`

In some cases, you get warnings – not immediately clear why but maybe some superclass requires an initializer you did not implement?

Implement with `[self doesNotRecognizeSelector:]` and declare the initializer `NS_UNAVAILABLE`.

# Errors

ObjC developers frequently misunderstand the error convention.  We use false to signal that an error occurred, and *optionally* an `error` to indicate the error.

Note that `error` can be nil, when an error occurs!  **If a function returns false, an error occurred.**  We don't recommend leaving error nil, because it might confuse the caller.

Swift generates code like this

```swift
var error: NSError?
if !func(args, error: &error) {
    throw error ?? Foundation._GeenericObjCError.nilError
}
```

To handle the case that `error` was nil.

Can use `NS_SWIFT_NOTHROW` to indicate that you're not following the convention.

# Consider writing overlays

Mixed targets, umbrella headers, etc.

Don't import the generated header in your umbrella header, it creates a circular dependency.

## Bool digressions
Note that on some platforms, swift's bool and ObjC bool have a different memory representations.  Ordinarily Swift can "just fix it", but for inout we have to do it ourselves.

## `NS_REFINED_FOR_SWIFT`

Mostly used to hide objc methods that are wrapped in an overlay.

Adds `__` to method name.  May also hide in code complete, unclear if it's just low-priority or actually hidden.

# Addressing missing APIs
Swift can't import
* C-style variadic parameters
* Flexible array members (struct that declares C array of unknown size)
* Forward declarations that are never fully defined
* Declarations involving un-importable types
* Invalid redeclarations (often skip over both of them, rather than pick 1 at random)
* XC12 is stricter
* Complicated macros

## Macros
Swift kind of guesses how to import macros.  In particular, it looks for macros that follow common patterns for declaring constants.  But the arbitrary language of C macros is turing-complete, so there are considerably more macros than this.

# Improve ergonomics in Swift
## `NS_SWIFT_NAME`

Note that `NS_SWIFT_NAME` can't add a `CustomStringConvertible` conformance, but you can do that on your own.

## Custom swift code can
* Wrap NSView or UIView in swiftUI View
* Return a Combine `Publisher` for the completion handler
* Integrate with Swift-only frameworks in other ways

[[Integrating SwiftUI]]
[[Combine in practice]]

## Improve error code enums

Typically

```swift
const NSString *SKErrorDomain;
typedef NS_ENUM(NSInteger, SKErrorCode) {
	SKErrorFoo = 1,
	SKErrorBar,
};
```

But this is hard to catch

```swift
catch /* SKErrorCode.errorFoo*/

//e.g.

catch let error as NSError where error.domain == foo && error.domain == bar
```

Use `NS_ERROR_ENUM` instead of `NS_ENUM`.  This will convert to swift error directly.

Implements `~=` for a match operator, that matches if domain and code are equal.

# Focus on use sites

Generated interfaces don't tell the full story.

# Resources

developer.apple.com/documentation/swift#2984801 (seems to be actually https://developer.apple.com/documentation/swift/objective-c_and_c_code_customization)

swift.org/documentation/api-design-guidelines

[[Behind the scenes of the Xcode build process - 18]]