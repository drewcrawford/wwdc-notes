String processing.  Regex type.  New type in stdlib.  Language built-in regex literal syntax, which makes this powerfula nd familiar concetp more first-class.

```swift
Regex {
    "Hi, WWDC"
    Repeat(.digit, count: 2)
    "!"
}
```

Builder API.  DSL that takes advantage of the syntactic simplicity of result builders and pushes regex to a whole new level.

[[Meet Swift Regex]]


```swift
let input = "name:  John Appleseed,  user_id:  100"

let regex = try Regex(#"user_id:\s*(\d+)"#)

if let match = input.firstMatch(of: regex) {
    print("Matched: \(match[0])")
    print("User ID: \(match[1])")
}
```



```swift
let input = "name:  John Appleseed,  user_id:  100"

let regex = /user_id:\s*(\d+)/

if let match = input.firstMatch(of: regex) {
    print("Matched: \(match.0)")
    print("User ID: \(match.1)")
}
```

For ultimate redability and customization,c an use the DSL

```swift
import RegexBuilder

let input = "name:  John Appleseed,  user_id:  100"

let regex = Regex {
    "user_id:"
    OneOrMore(.whitespace)
    Capture(.localizedInteger)
}

if let match = input.firstMatch(of: regex) {
    print("Matched: \(match.0)")
    print("User ID: \(match.1)")
}
```

# How does Regex work?
Regex engine takes an input string and performs matching from the start to the end.
```swift
let regex = Regex {
    OneOrMore("a")
    OneOrMore(.digit)
}

let match = "aaa12".wholeMatch(of: regex)
```
Use one of the matching algorithms, `wholeMatch` to match input.

1.  Match first character.  
2. Match as many as possible
3. Stop when we fail
4. Move to the next pattern
5. When we reach the end, matching succeeds.

More aobut the execution model.

Builder DSL and algorithms expand the power and expressivity of regex.  Collection-based API that provides the most common operations

```swift
let input = "name:  John Appleseed,  user_id:  100"

let regex = /user_id:\s*(\d+)/

input.firstMatch(of: regex)           // Regex.Match<(Substring, Substring)>
input.wholeMatch(of: regex)           // nil
input.prefixMatch(of: regex)          // nil

input.starts(with: regex)             // false
input.replacing(regex, with: "456")   // "name:  John Appleseed,  456"
input.trimmingPrefix(regex)           // "name:  John Appleseed,  user_id:  100"
input.split(separator: /\s*,\s*/)     // ["name:  John Appleseed", "user_id:  100"]

switch "abc" {
case /\w+/:
    print("It's a word!")
}
```

can be used in pattern match, etc.  

We also have regex support in Foundation.

[[What's new in Foundation]]

Formats and parsers you're already using.  Date, number, etc.  See the session.  This year, support for formatting and parsing URLs as well.

Embed the foundation parsers directly in regex builder.  ex,

```swift
let statement = """
    DSLIP    04/06/20 Paypal  $3,020.85
    CREDIT   04/03/20 Payroll $69.73
    DEBIT    04/02/20 Rent    ($38.25)
    DEBIT    03/31/20 Grocery ($27.44)
    DEBIT    03/24/20 IRS     ($52,249.98)
    """

let regex = Regex {
    Capture(.date(format: "\(month: .twoDigits)/\(day: .twoDigits)/\(year: .twoDigits)"))
    OneOrMore(.whitespace)
    OneOrMore(.word)
    OneOrMore(.whitespace)
    Capture(.currency(code: "USD").sign(strategy: .accounting))
}
```

Use the foundation-provided date parser, or currency parser.  Create regexes out of existing battle-tested parsers.

# Use a regex
Let's parse XCTest Logs.

First and last lines of the log.  

> Many of you may be familiar with the textual Regex syntax.

Literal starts and ends with `/`.  Swift infers correct strong type.  Output type, etc.

First-class regex literal is strongly typed capturing groups.  I can write a capturing group to capture two digits as the year, and gitve it a name.  Later I will show you how you can use captures.

Besides standard regex literals, swift also supports extended regex literals with `#/` and `/#`.  This allows non-semantic whitepsaces.  Can split patterns into multipline lines.

To match these options we use `ChoiceOf`.  Multiple subpatterns.
```swift
import RegexBuilder

let regex = Regex {
    "Test Suite '"
    /[a-zA-Z][a-zA-Z0-9]*/
    "' "
    ChoiceOf {
        "started"
        "passed"
        "failed"
    }
    " at "
    OneOrMore(.any)
    Optionally(".")
}
```

Match whole string.  Here I'm matching each log message and print the match content.

```swift
let testSuiteTestInputs = [
    "Test Suite 'RegexDSLTests' started at 2022-06-06 09:41:00.001",
    "Test Suite 'RegexDSLTests' failed at 2022-06-06 09:41:00.001.",
    "Test Suite 'RegexDSLTests' passed at 2022-06-06 09:41:00.001."
]

for line in testSuiteTestInputs {
    if let match = line.wholeMatch(of: regex) {
        print("Matched: \(match.output)")
    }
}
```

We also want to extract the ifnormation like name, status, timestamp.  Captures.

A capture saves a portion of the input during matching.  Can be` Capture("b")` or `(e)` in regex syntax.
Appends the matched substring to Output tuple.  
Result accessible from `Regex.Match`.  
```swift
let regex = Regex {
   "a"
   Capture("b")
   "c"
   /d(e)f/
}

if let match = "abcdef".wholeMatch(of: regex) {
    let (wholeMatch, b, e) = match.output
}
```


```swift
import RegexBuilder

let regex = Regex {
    "Test Suite '"
    Capture(/[a-zA-Z][a-zA-Z0-9]*/)
    "' "
    Capture {
        ChoiceOf {
            "started"
            "passed"
            "failed"
        }
    }
    " at "
    Capture(OneOrMore(.any))
    Optionally(".")
}
```

```swift
let testSuiteTestInputs = [
    "Test Suite 'RegexDSLTests' started at 2022-06-06 09:41:00.001",
    "Test Suite 'RegexDSLTests' failed at 2022-06-06 09:41:00.001.",
    "Test Suite 'RegexDSLTests' passed at 2022-06-06 09:41:00.001."
]

for line in testSuiteTestInputs {
    if let (whole, name, status, dateTime) = line.wholeMatch(of: regex)?.output {
        print("Matched: \"\(name)\", \"\(status)\", \"\(dateTime)\"")
    }
}
```

Printed name, status, and timestamp.  It included the period in the input as part of the capture.  ??

Can make `OneOrMore`, `.reluctant`.  It is a case of repetition behaviors.  We call these repetitions:
* `OneOrMore`
* `ZeroOrMore`
* `Optionally`
* `Repeat`

A repetition is eager by default.  Matches as many occurrences as possible.  Because we're running `wholeMatch` and both the input and pattern reach the end, matching succeeds.  

`reluctant` matches as few occurrences as possible.  Always tries to match the rest of the regex first, before assuming a repetition occurs.  When the regex doesn't match, it backtracks the repetition.

* Eager by default
	* Think about its implications on your intended match
	* Specify with `.eager` or similar.
	* Or use `.repetitionBehavior` modifier to override for a group.


```swift
import RegexBuilder

let regex = Regex {
    "Test Suite '"
    Capture(/[a-zA-Z][a-zA-Z0-9]*/)
    "' "
    Capture {
        ChoiceOf {
            "started"
            "passed"
            "failed"
        }
    }
    " at "
    Capture(OneOrMore(.any, .reluctant))
    Optionally(".")
}
```

As i use Capture to extract test status from the input, its type is substring.  but it's better to convert into something more programming-friendly like a custom datastructure.  Use a transforming cpature.

Appends transform result type to Output type.  Calls the transforming closure.  Produces the result of desired type.

```swift
Regex {
    Capture {
        OneOrMore(.digit)
    } transform: {
        Int($0)     // Int.init?(_: some StringProtocol)
    }
} // Regex<(Substring, Int?)>
```

Corresponding regex output type becomes closure's return type.  To obtain a non-ptional output, `TryCapture` can help.  It is a variant that removes optionality from tarnsform.
Backtracks when transform returns `nil`.

```swift
Regex {
    TryCapture {
        OneOrMore(.digit)
    } transform: {
        Int($0)     // Int.init?(_: some StringProtocol)
    }
} // Regex<(Substring, Int)>
```

When you transform a captuer with a failable initializer.  Let's define an enumeration.

```swift
enum TestStatus: String {
    case started = "started"
    case passed = "passed"
    case failed = "failed"
}

let regex = Regex {
    "Test Suite '"
    Capture(/[a-zA-Z][a-zA-Z0-9]*/)
    "' "
    TryCapture {
        ChoiceOf {
            "started"
            "passed"
            "failed"
        }
    } transform: {
        TestStatus(rawValue: String($0))
    }
    " at "
    Capture(OneOrMore(.any, .reluctant))
    Optionally(".")
} // Regex<(Substring, Substring, TestStatus, Substring)>
```
Enum is initializable via string.  In the regex, we use `TryCapture` with a transform.

Now the corresponding output type is `TestStatus`.  Using a custom datastructure makes the custom match output typesafe.

Additioanl improvement.  Currently, I match the timestamp using a wildcard pattern.  This produces a substring.  If my app wants to understand the timestamp, it'd have to parse the substring again into another datastructure.

Earlier I mentioend foundation supports swift regex, providing industry-strength parsers as regexers.  I can switch to foundation ISO 8601 parser.
```swift
let regex = Regex {
    "Test Suite '"
    Capture(/[a-zA-Z][a-zA-Z0-9]*/)
    "' "
    TryCapture {
        ChoiceOf {
            "started"
            "passed"
            "failed"
        }
    } transform: {
        TestStatus(rawValue: String($0))
    }
    " at "
    Capture(.iso8601(
        timeZone: .current, includingFractionalSeconds: true, dateTimeSeparator: .space))
    Optionally(".")
} // Regex<(Substring, Substring, TestStatus, Date)>
```

Now this shows that regex outputs date.  As i run `wholeMatch`, I can see we parsed into a date value.

# Reuse an existing parser
Incredibly handy in day to day tasks.  

Let's reuse a existing parser.


```swift
let input = "Test Case '-[RegexDSLTests testCharacterClass]' passed (0.001 seconds)."

let regex = Regex {
    "Test Case "
    OneOrMore(.any, .reluctant)
    "("
    Capture {
        .localizedDouble
    }
    " seconds)."
}

if let match = input.wholeMatch(of: regex) {
    print("Time: \(match.output)")
}
```

Duration is a floating point number such as `0.001`.  Let's use localized parser.

We have a function from the c stard library, `strtod`.  This returns a number parsed from the string.  This takes a string pointer, parses the underlying string, and assigns the end position of the amtch to the end pointer.

Let's parse the duration the C way.  I conform my type to `CustomConsumingRegexComponen`.  Its `RegexOutput` type is `Double`.  It calls the C standard library.
```swift
import Darwin

struct CDoubleParser: CustomConsumingRegexComponent {
    typealias RegexOutput = Double

    func consuming(
        _ input: String, startingAt index: String.Index, in bounds: Range<String.Index>
    ) throws -> (upperBound: String.Index, output: Double)? {
        input[index...].withCString { startAddress in
            var endAddress: UnsafeMutablePointer<CChar>!
            let output = strtod(startAddress, &endAddress)
            guard endAddress > startAddress else { return nil }
            let parsedLength = startAddress.distance(to: endAddress)
            let upperBound = input.utf8.index(index, offsetBy: parsedLength)
            return (upperBound, output)
        }
    }
}
```

Use it in my regex:

```swift
let input = "Test Case '-[RegexDSLTests testCharacterClass]' passed (0.001 seconds)."

let regex = Regex {
    "Test Case "
    OneOrMore(.any, .reluctant)
    "("
    Capture {
        CDoubleParser()
    }
    " seconds)."
} // Regex<(Substring, Double)>

if let match = input.wholeMatch(of: regex) {
    print("Time: \(match.1)")
}
```

# Wrap up
* string processing with Regex
* Concision vs readability
* Prefer existing parsers for common patterns
# More information
see these swift evolution proposals

* https://github.com/apple/swift-evolution/blob/main/proposals/0357-regex-string-processing-algorithms.md
* https://github.com/apple/swift-evolution/blob/main/proposals/0355-regex-syntax-run-time-construction.md
* https://github.com/apple/swift-evolution/blob/main/proposals/0354-regex-literals.md
* https://github.com/apple/swift-evolution/blob/main/proposals/0351-regex-builder.md
* https://github.com/apple/swift-evolution/blob/main/proposals/0350-regex-type-overview.md
