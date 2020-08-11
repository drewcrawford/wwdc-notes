#swift 


# What is type inference?

Omit details

coerce via lvalue `:String` or rvalue `"" as String`

# Leveraging type inference
Example.  Interesting use of `@ViewBuilder`, which is some kind of wrapper for `SwiftUI.View` somehow involved in the DSL of #swiftui.

# How type inference works
This is a logic puzzle basically.

## How does the compiler deal with the case where all the clues don't fit?

New: integrated error tracking. Partially in 11.4, fully in 12.0

* records information about errors in source code
* Attempts to fix errors to continue the type inference algorithm
* Provides actionable error messages based on collected information


# Using Swift and xcode to fix errors

**Xcode behaviors -> automatically show issue navigator when build fails!!!**


Compiler notes now include info about type inference the compiler was trying when it failed.

command-enter: close canvas
command-option-enter: reopen canvas
option-shift -> destination chooser