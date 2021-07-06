#swiftui 

# Strings
Text() with a string literal automatically looks up localized strings.

```swift
Button(action: done) {
   Text("Done", comment: "Button title to dismiss rewards sheet")
}
```

* String interpolation "just works"

Format types are automatically inferred in Xcode 13

More control for tables:
```swift
// RecipeView.swift
Text("Ingredients.recipe", tableName: "Ingredients", comment: "Ingredients in a recipe. For languages that have different words for \"Ingredient\" based on semantic context.")
Text("Ingredients.menu", tableName: "Ingredients", comment: "Ingredients in a smoothie. For languages that have different words for \"Ingredient\" based on semantic context.")
```

Russian requires words to be translated fiferently based on context

[[Streamline your localized strings]]

First arg to text is `LocalizedStringKey`.  If you have custom views/methods that accept string literals, consider using this type

```swift
struct Card: View {
    var title: LocalizedStringKey
    var subtitle: LocalizedStringKey
    
    var body: some View {
        Circle()
            .fill(BackgroundStyle())
            .overlay(
                VStack(spacing: 16) {
                    Text(title)
                    Text(subtitle)
                }
            )
    }
}

Card(
    title: "Thank you for your order!",
    subtitle: "We will notify you when your order is ready."
 )
 ```
 
 Alternatively, have your views accept a `Text` argument instead.
 
 `LocalizedStringKey` allows preview to look up localizations.  Change language in scheme editor.
 
 `Text` and `LocalizedStringKey` now work together with compiler.  Xcode can do a better job of finding localizable content and extracting it.
 
 Parsed correctly:
 
 ```swift
 Text("""
        A delicious blend of tropical fruits and blueberries will
        have you sambaing around like you never knew you could!
        """,
        comment: "Tropical Blue smoothie description")
```
 
 
# Layout
Localized layouts in swiftUI
* Default behaviors will handle most cases
* Wrapping is appropriately handled per control

Smoothee name is wrapped to a second line in russian.

* Default behaviors will handle most cases
* Most controls wrap text by default
* Support for RTL languages works out of the box

```swift
VStack(alignment: .leading) {
    Text(smoothie.title)
        .font(.headline)
    Text(ingredients)
}
```


# Styling
Introduced markdown for localized strings

```swift
// Smoothie.swift

Text("A refreshing blend that's a *real kick*!", comment: "Lemonberry smoothie description")
```

Arabic has no italics.  e.g., can use strong emphasis.

SwiftUI makes it easy to take advantage of by passing styled strings directly to text for display

[[What's new in Foundation]]

[[21/What's new in SwiftUI]]


# Formatting

New formatting API works with Text and TextField for more declarative code.

```swift
let calories = Measurement<UnitEnergy>(
    value: nutritionFact.kilocalories, unit: .kilocalories)

Text(calories.formatted(.measurement(width: .wide, usage: .food)))

Text("Energy: \(calories, format: .measurement(width: .wide, usage: .food))")
```

[[What's new in Foundation]]


# Keyboard shortcuts
Keyboard shortcuts are automatically adjusted for active layout

```swift
struct SmoothieCommands: Commands {

    var body: some Commands {
        CommandMenu(Text("Smoothie", comment: "Menu title for smoothie-related actions")) {
            SmoothieFavoriteButton(smoothie)
                .keyboardShortcut("+")
        }
    }
}
```

However, on lithnian keyboard, reaching + is not so easy.  Must press backtick first.  Also not typable while holding down cmd.

But now, changed to command-ž.  

You as the developer don't need to do anything.
# Exporting for translation
Xcode will build all targets in projects and extract localized string keys.

use 2 languages in swiftUI previews to see which strings are localizable and which i've missed.

Scheme=>edit scheme=>app language.  Accented pseudolanguage adds different accent marks to source strings in UI.

Custom swiftUI view that takes a label string and passes it down to a text view.  In custom swiftUI views, need to use `LocalizedStringKey`.

Make sure it handles plurals properly.  Stringsdict can provide different translations for plural variants

[[Streamline your localized strings]]

Starting in xcode 12.5, you can export/import for products in product menu.

[[New Localization Workflows in Xcode 10 - 18]]

Let's check what was exported.  Starting with XC13, localization catalogs can be opened directly.

Super useful if you are localizing your own app, verifing strings, fix translation for specific languages.

Use `.formatted(.measurement)` to localize a unit.

Noun vs verb.  Add comments to distinguish.

# Summary
* Extract `LocalizedStringKeys`
* "Use Compiler to Extract Swift Strings"
* Internationalize your code with formatting
* Style your localized strings with markdown
* Use `Text()` to add comments for translation context
https://developer.apple.com/documentation/Xcode/localization
https://developer.apple.com/localization/
https://developer.apple.com/documentation/swiftui/fruta_building_a_feature-rich_app_with_swiftui




