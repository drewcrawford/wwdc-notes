More than ever, it's crucial that text can be read.

175 app store regions

Strings are everywhere.  

> Think about all the strings in your app as movie subtitles.  

# How strings come to life
* UX
* Foundation
* Strings files





# Create your user interface

Displaying localized strings is easy.  

Two types of visible strings.

1.  In a SwiftuI view or storyboard view
2.  anywhere else.  e.g. variable, etc.

In all the cases of 2, you can use `NSLocalizedString`.
Now, you can use `String(localized:)`

## SwiftUI

Views constructed with `LocalizedStringKey` are automatically localizable.

Avoid unnecessary work

[[Localize your SwiftUI App]]

How to make the string more dynamic?

It's not obvious how many tickets I'm ordering.  So let's improve the button with a count.  Interpolation works, evidently.

Common pitfall.  `String(format:)` is not intended to be used for localized strings.

e.g. `%d` is not ok for arabic with a different digit system.

`String(localized: "Order\(count) Tickets")`

Gluing strings together is discouraged.  e.g. "Order Now" might be different than "Order Later".  Safer to use two separate strings.

```swift
// Recommended for all languages
String(localized: "Order Now")
String(localized: "Order Later")
```

Make sure to use comments to help with localization.  Always define a comment.

Don't forget storboard files, they have a comment field.

1.  Explain where the string is visible.  e.g. what interface element.  Action vs statement is critical.
2.  Explain context.  Order: Completing a transaction?  Sorting a list.
3.  What each variable is.  They don't know what your variables mean.  What does the number represent?  May be gender conformance issues as well.

# Define your targets
How does foundation ensure your code loads the correct strings file?

Project settings.  Where the UI elements live, they are shared across languages.  e.g. a storyboard file is shared.  Make sure you click "Localized" button in IB inspector for all shared assets.

But strings belong to one language.  

## Language
Test your app using scheme settings, test plans, and swiftUI previews

Foundation handles language fallbacks for you

Always request server strings in the user's language, for strings that come from a server.

```swift
Bundle.preferredLocalizations(from: allServerLanguages).first
```

Strings organized into files called tables

## Table
Advanced feature to help you organize your strings files.

Localizable.strings
Localizable.stringsdict

Means all strings are stored in this file(s).

```swift
Text("\(ticketCount) Ordered",
     tableName: "UserProfile",
     comment: "Profile subtitle: total number of tickets ordered")
```

## Bundle
Usually `.main`.  In your app extension, `.main` refers to the extension.

Let's say you want to share strings between app and extension.  You need to provide bundle of the main app.

Can tap directly into a framework.  

Framework may provide the bundle to foundation, and the variable to your app.  

```swift
/* —-----------—------------—-—---- In TicketKit Framework —---------—------------—-—---- */

// TicketKit/OrderStatus.swift
public enum OrderStatus {
    case pending, processing, complete, canceled, invalid(Error)

    var displayName: String {
        switch self {
        case .complete: return String(localized: "Complete",
                                      bundle: Bundle(for: AnyClassInTicketKit.self),
                                      comment: "Standalone ticket status: order finalized")
          
/* —-----------—-----------—---—---       In Host App      —---------—------------—-—---- */

import TicketKit
Text(OrderStatus.complete.displayName)
```


# Generate all strings
You don't need to create strings files, xcode figures it out.

Product=>Export localizations.  

This year, in xc13, we've added compier support for swift strings extraction.  Workspaces are fully supported.

Your logic vs translations.

Localization exports include:

> Be aware if you have custom code that wraps these APIs, it will not work.

If you need to use a macro you can add them to your build settings.

* STatic strings in code calling localized strings API
* Localizable Info.plist entries (app name, privacy descriptions)
* Localized assets (inspector)
* Existing strings and stringsdict files
	* convert at your own pace
* Screenshots from UI tests are included for string context

New in xcode 13, you can view/edit localization catalogs, directly in xcode.  

Filter, screenshots, translate, etc.

```swift
xcodebuild -exportLocalizations -workspace VacationPlanet.xcworkspace -localizationPath ~/Documents
xcodebuild -importLocalizations -workspace VacationPlanet.xcworkspace -localizationPath ~/Documents/de.xcloc
```

[[Localize your SwiftUI App]]
[[New Localization Workflows in Xcode 10 - 18]]


# Format advanced strings
## Attributed strings
```swift
AttributedString(localized: "Your order is **complete**!",
                 comment: "Ticket order confirmation title")
```

[[What's new in Foundation]]

1 string => 1 translation
=> multiple translations?

e.g. plurals.  

First, need to create a stringsdict.  Unlike strings file.

Inside, define the actual value.  Search-replace mechanism

* `Order %d Ticket(s)`
	* NSStringLocalizedFormatKey `%#@tickets@`
	* tickets
		* NSStringFormatSpecTypeKey => NSStringPLuralRuleType
		* NSStringFormatValueTypeKey => `d`
		* zero => `No Selection`
		* one => `Order %d Ticket`
		* other => `Order %d Tickets`

Without a number?
* Stringsdict `one` case is a category
* In russian it's used for 1,21,31,41...
* In chinese, `other` is used for every number

Here, using stringsdict would not be correct as you just want equals to 1 only.

Use separate strings for generic singular/pair/plural distinction.

* Plural cases
* Device type
* Available width

[[Creating Great Localized Experiences with Xcode 11 - 19]]
[[Localizing with Xcode 9 - 17]]

## Automatic grammar agreement

```swift
AttributedString(localized: "Order ^[\(ticketsCount) Ticket](inflect: true)")
```

> In select languages

## Term of address

[[What's new in Foundation]]

Can also be defined in translations.

```
^[Bienvenida](inflect:true, inflectionAlternative: 'Te damas la bienvenida')
```

## Format data in strings

Dates, times, measurements, precentages, names, and lists should use a formatter

```swift
["pop", "rock", "electronic"].formatted(.list(type: .or)) // pop, rock, or electronic

Text("Total: \(price, format: .currency(code: "USD"))", // Total: $9.41
     comment: "Order subtitle: total price of all tickets")
```

[[What's new in Foundation]]
[[Formatters Make Data Human-Friendly]]

# Wrap up
Declare localizable strings in UI
Let Xcode manage them
Organize strings in bundles
Leverage new grammar and format APIs
Test!
