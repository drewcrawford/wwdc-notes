> You'd think for a task this improtant, we'd be processing well-structured data.

> Either two or more spaces, or a tab.  For a very important technical reason that no one involved can remember.

# Processing collections
* element-based: `map`, `filter`, `split`
* Low-level index-based: `index(after:)` `firstIndex(of:)`, slicing subscript

Try to use element-based.  But the field separator being either tab or spaces, makes this difficult.  Whitespace alone desont' cut it.

```swift
let transaction = "DEBIT     03/05/2022    Doug's Dugout Dogs         $33.27"

let fragments = transaction.split(whereSeparator: \.isWhitespace)
// ["DEBIT", "03/05/2022", "Doug\'s", "Dugout", "Dogs", "$33.27"]
```

Drop down to low-level index manipulation code.  Hard to do right, and takes a lot of code:

```swift
var slice = transaction[...]

// Extract a field, advancing `slice` to the start of the next field
func extractField() -> Substring {
  let endIdx = {
    var start = slice.startIndex
    while true {
      // Position of next whitespace (including tabs)
      guard let spaceIdx = slice[start...].firstIndex(where: \.isWhitespace) else {
        return slice.endIndex
      }

      // Tab suffices
      if slice[spaceIdx] == "\t" {
        return spaceIdx
      }

      // Otherwise check for a second whitespace character
      let afterSpaceIdx = slice.index(after: spaceIdx)
      if afterSpaceIdx == slice.endIndex || slice[afterSpaceIdx].isWhitespace {
        return spaceIdx
      }

      // Skip over the single space and try again
      start = afterSpaceIdx
    }
  }()
  defer { slice = slice[endIdx...].drop(while: \.isWhitespace) }
  return slice[..<endIdx]
}

let kind = extractField()
let date = try Date(String(extractField()), strategy:  Date.FormatStyle(date: .numeric))
let account = extractField()
let amount = try Decimal(String(extractField()), format: .currency(code: "USD"))
```

The reason `split` doesn't work is beacuse it's element based, while the separator is a more complex pattern.  A solution is to write a regular expression.  

Lexical analysis, compilers, etc.  Take regex beyodn theoretical roots and extract things beyodn portions of the input.  Add expressive power, etc.  Regex.

`struct Regex<Outpput>` .  Can create one with a literal.  Compatible with Perl, Python, Ruby, Java, NSRegularExpression and many others.

```swift
// Regex literals
let digits = /\d+/
// digits: Regex<Substring>
```

You'll get syntax highlighting, compile-time errors, and even strongly-typed captures.  Can create at runtime.
```swift
// Run-time construction
let runtimeString = #"\d+"#
let digits = try Regex(runtimeString)
// digits: Regex<AnyRegexOutput>
```

Output type is an existential `AnyRegexOutput` because the types and number of captures are unknown at compile time.

Can also be used with a builder.

```swift
// Regex builders
let digits = OneOrMore(.digit)
// digits: Regex<Substring>
```

exampel from earlier.  "two or more occurrences of any whitespace character."  Or single tab.  Pipe = choice between alternatives.

```swift
let transaction = "DEBIT     03/05/2022    Doug's Dugout Dogs         $33.27"

let fragments = transaction.split(separator: /\s{2,}|\t/)
// ["DEBIT", "03/05/2022", "Doug's Dugout Dogs", "$33.27"]
```

Now that our fields are split, let's make a contribution to civlization by normalizing to a tab.

We could call join, but there's a better algorithm.  `replacing` is superior.

```swift
let transaction = "DEBIT     03/05/2022    Doug's Dugout Dogs         $33.27"

let normalized = transaction.replacing(/\s{2,}|\t/, with: "\t")
// DEBIT»03/05/2022»Doug's Dugout Dogs»$33.27
```

If you are familiar with regex, you may know if their mixed reputation.
> I had a problem so I wrote a regular expression.  Now i have two problems

Swift advances the art in four key areas.
* concise or cryptic?
	* Literals are concise; builders provide structure
* Data formats need a real parser
	* Interweave real parsers as part of regex
	* Library-extensible fashion
* World is Unicode, not just ASCII
	* Regex does Unicode
* Uncertain or unconstrained execution
	* Predictable execution, prominent controls
# Builder ex

```swift
// CREDIT    03/02/2022    Payroll from employer         $200.23
// CREDIT    03/03/2022    Suspect A                     $2,000,000.00
// DEBIT     03/03/2022    Ted's Pet Rock Sanctuary      $2,000,000.00
// DEBIT     03/05/2022    Doug's Dugout Dogs            $33.27

import RegexBuilder
let fieldSeparator = /\s{2,}|\t/
let transactionMatcher = Regex {
  /CREDIT|DEBIT/
  fieldSeparator
  One(.date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt))
  fieldSeparator
  OneOrMore {
    NegativeLookahead { fieldSeparator }
    CharacterClass.any
  }
  fieldSeparator
  One(.localizedCurrency(code: "USD").locale(Locale(identifier: "en_US")))
}
```

NegativeLookahead lets you precisely control how a regex matches its components.

We use `Capture` to extract some data out.  Portions of our input for alter processing.  By convention, the 0 capture is the entire input.  Each explicit capture will follow.

For dates, we capture the strongly-typed value that was parsed out,w ithout needed to post-process the text.  

```swift
let fieldSeparator = /\s{2,}|\t/
let transactionMatcher = Regex {
  Capture { /CREDIT|DEBIT/ }
  fieldSeparator

  Capture { One(.date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt)) }
  fieldSeparator

  Capture {
    OneOrMore {
      NegativeLookahead { fieldSeparator }
      CharacterClass.any
    }
  }
  fieldSeparator
  Capture { One(.localizedCurrency(code: "USD").locale(Locale(identifier: "en_US"))) }
}
// transactionMatcher: Regex<(Substring, Substring, Date, Substring, Decimal)>
```

We recommend you dump the data into a real database.  But they want to ekep everything as strings.  Good news for this talk because you see even more of swift regex.

ex - date type depends on currency used?

# Plot twist
US dollars is MDY, whereas british pounds are DMY.

```swift
private let ledger = """
KIND      DATE          INSTITUTION                AMOUNT
----------------------------------------------------------------
CREDIT    03/01/2022    Payroll from employer      $200.23
CREDIT    03/03/2022    Suspect A                  $2,000,000.00
DEBIT     03/03/2022    Ted's Pet Rock Sanctuary   $2,000,000.00
DEBIT     03/05/2022    Doug's Dugout Dogs         $33.27
DEBIT     06/03/2022    Oxford Comma Supply Ltd.   £57.33
"""
// 😱
```

How to disabiguate?  sed-like script.  Have slashes inside without having to escape them.  An extended syntax mode where whitespace is ignored.  Use whitespace for readability, like normal code

Named captures.  Show up in output like tuple.  Handle specific symbols in application logic.

```swift
let regex = #/
  (?<date>     \d{2} / \d{2} / \d{4})
  (?<middle>   \P{currencySymbol}+)
  (?<currency> \p{currencySymbol})
/#
// Regex<(Substring, date: Substring, middle: Substring, currency: Substring)>
```


All our assupmtions are explicit in code, making it easy to adapt and evolve.
```swift
let regex = #/
  (?<date>     \d{2} / \d{2} / \d{4})
  (?<middle>   \P{currencySymbol}+)
  (?<currency> \p{currencySymbol})
/#
// Regex<(Substring, date: Substring, middle: Substring, currency: Substring)>

func pickStrategy(_ currency: Substring) -> Date.ParseStrategy {
  switch currency {
  case "$": return .date(.numeric, locale: Locale(identifier: "en_US"), timeZone: .gmt)
  case "£": return .date(.numeric, locale: Locale(identifier: "en_GB"), timeZone: .gmt)
  default: fatalError("We found another one!")
  }
}
```

Use our regex with a find/replace algorithm.

```swift
let regex = #/
  (?<date>     \d{2} / \d{2} / \d{4})
  (?<middle>   \P{currencySymbol}+)
  (?<currency> \p{currencySymbol})
/#
// Regex<(Substring, date: Substring, middle: Substring, currency: Substring)>

func pickStrategy(_ currency: Substring) -> Date.ParseStrategy { … }

ledger.replace(regex) { match -> String in
  let date = try! Date(String(match.date), strategy: pickStrategy(match.currency))

  // ISO 8601, it's the only way to be sure
  let newDate = date.formatted(.iso8601.year().month().day())

  return newDate + match.middle + match.currency
}
```

More adaptable to changing requirements.  Helps us evolve that much quicker.  a regex declares an algorithm over some model of string.  Swift string presents multiple models for working with unicode.

```swift
let aZombieLoveStory = "🧟‍♀️💖🧠"
// Characters: 🧟‍♀️, 💖, 🧠
```

characters are complex entities formally called Unicode extended grapheme cluster.
Single character: unicode scalar.
```swift
aZombieLoveStory.unicodeScalars
// Unicode scalar values: U+1F9DF, U+200D, U+2640, U+FE0F, U+1F496, U+1F9E0
```

Advanced usage as well as compatability with other systems.

zombie consists of 4 scalars.
* zombie
* female
* zero-width join
* selector 16


```swift
aZombieLoveStory.utf8
// UTF-8 code units: F0 9F A7 9F E2 80 8D E2 99 80 EF B8 8F F0 9F 92 96 F0 9F A7 A0
```

# Comparison
Same exact character can soemtimes be represented with different scalars.  ex.
```swift
"café".elementsEqual("cafe\u{301}")
// true
```

Can be represented as either a single scalar.

```swift
"café".elementsEqual("cafe\u{301}")
// true

"café".unicodeScalars.elementsEqual("cafe\u{301}".unicodeScalars)
// false

"café".utf8.elementsEqual("cafe\u{301}".utf8)
// false
```

or ascii e, followed by "combining acute accent".

Unicode canonical equivalence.  From the perspective of uincode scalar or utf-8, these are different.   Just like string, this is "correct by default".


```swift
switch ("🧟‍♀️💖🧠", "The Brain Cafe\u{301}") {
case (/.\N{SPARKLING HEART}./, /.*café/.ignoresCase()):
  print("Oh no! 🧟‍♀️💖🧠, but 🧠💖☕️!")
default:
  print("No conflicts found")
}
```

`.` matches any character, any unicode grapheme cluster.

You can match with `.unicodeScalar` semantics.  here the `.` matches single unicode scalar value.  

```swift
let input = "Oh no! 🧟‍♀️💖🧠, but 🧠💖☕️!"

input.firstMatch(of: /.\N{SPARKLING HEART}./)
// 🧟‍♀️💖🧠

input.firstMatch(of: /.\N{SPARKLING HEART}./.matchingSemantics(.unicodeScalar))
// ️💖🧠
```

Instead of a date, they have a precise timetamp.  Regular expression that matches just fine.


Filter transactions against runtime compiled regex.  Because this is live, they want to bail early on any uninteresting transactions.

Fields are still separated by 2 or more spaces or a tab.


```swift
let timestamp = Regex { ... } // proprietary
let details = try Regex(inputString)
let amountMatcher = /[\d.]+/

// CREDIT    <proprietary>      <redacted>        200.23        A1B34EFF     ...
let fieldSeparator = /\s{2,}|\t/
let transactionMatcher = Regex {
  Capture { /CREDIT|DEBIT/ }
  fieldSeparator

  Capture { timestamp }
  fieldSeparator

  Capture { details }
  fieldSeparator

  // ...
}
```

> Everything technically works

Timestamp and details regexes match more of the input than their fields.  Ideally these are constrained to only run over a single field.  We handled a similar issue with `NegativeLookahead`.


```swift
let field = OneOrMore {
  NegativeLookahead { fieldSeparator }
  CharacterClass.any
}
```

Bail early with `TryCapture`.  Passes matched field to our closure where we test against the detailed regexes.  

Return `nil` will signal matching fails.  Closure participates in matching, which is exactly what we need.

With this we've solved a major scaling issue.  But it can take a long time to exit.


```swift
// CREDIT    <proprietary>      <redacted>        200.23        A1B34EFF     ...
let fieldSeparator = /\s{2,}|\t/
let field = OneOrMore {
  NegativeLookahead { fieldSeparator }
  CharacterClass.any
}
let transactionMatcher = Regex {
  Capture { /CREDIT|DEBIT/ }
  fieldSeparator

  TryCapture(field) { timestamp ~= $0 ? $0 : nil }
  fieldSeparator

  TryCapture(field) { details ~= $0 ? $0 : nil }
  fieldSeparator

  // ...
}
```

If the regex later fails it will backup and only match 7 whitespace characters.  Then it will back up again, etc. etc.  Only after trying all alternatives does matching fail.  This backing up in order tot ry alternatives is called *global backtracking*.  Or in formal logics, the *Kleene closure*.  Gives the characteristic power, but opens up a broad search space.  Here we want to match all whitespace.

General tool – place fieldSeparator in a local space.
`Local` - if contained regex matches, any untried alternatives are discarded.  Even if our transaction matcher fails later on, we don't go back to try consuming fewer spaces.  Global backtracking is great for search and fuzzy matching.  Local is useful to matching precisely specified tokens. Field separator is precise.

Local is known elsewhere as an *atomic non-capturing group*.  This contains the search space.

```swift
// CREDIT    <proprietary>      <redacted>        200.23        A1B34EFF     ...
let fieldSeparator = Local { /\s{2,}|\t/ } 
let field = OneOrMore {
  NegativeLookahead { fieldSeparator }
  CharacterClass.any
}
let transactionMatcher = Regex {
  Capture { /CREDIT|DEBIT/ }
  fieldSeparator

  TryCapture(field) { timestamp ~= $0 ? $0 : nil }
  fieldSeparator

  TryCapture(field) { details ~= $0 ? $0 : nil }
  fieldSeparator

  // ...
}
```

# Wrap up
[[Swift Regex Beyond the Basics]]

* Regex builders vs literals is subjective
* Use real parsers whenever possible
* the Unicode Way, by default
* Think globally, act `Local`ly.


```swift

```













