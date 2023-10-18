Discover how Xcode 15 makes it easy to localize your app by managing all of your strings in one place. We'll show you how to extract, edit, export, and build strings in your project using String Catalogs. We'll also share how you can adopt String Catalogs in existing projects at your own pace by choosing which files to migrate.

Previously, you needed strings and stringsdict files.  Unlocalized strings for users, etc.

in xc15 we're introducing string catalogs.  This will supercede prior formats.  Manage all strings in one place, etc.

Be confident your content is fully localized before shipping.

All strings in swift code have been automatically extracted into localizable catalog.
right click -> vary by device.  I'm not really sure why you wouldn't do this in code, but ok.

# Extract

## 4:30 - Localizable string
```swift
String(localized: "Welcome to WWDC!")
```

## 4:42 - Localizable string with default value
```swift
String(localized: "WWDC_NOTIFICATION_TITLE",
   defaultValue: "Welcome to WWDC!")
```

## 5:05 - Localizable string with comment
```swift
String(localized: "Welcome to WWDC!",
   comment: "Notification banner title")
```

## 5:22 - Localizable string with table and comment
```swift
String(localized: "Welcome to WWDC!",
   table: "WWDCNotifications",
   comment: "Notification banner title")
```
* key - identifier, often equivalent to the string itself
* default value - explicit if desired, or otherwise the key
* comments.  We recommend dadding comments, etc.
* table - can organize how you want.  By default we pick a default table

Usually a single strings table contains strings for each language lproj directory.  All the files shown here make up the localizable string table.

Catalog contains an entire string table in a single file.  Includes all translations, and extra metadata for each localizable string in that table.

although keys are unique within their containing table, no requirement to be unique across tables.

as mentioned, xcode 15 will automatically populate string catalogs and make a best effort to keep them in sync with localizable strings.  Where does xcode find these strings?

variety of places
* swiftui views
* storyboards
* swift code
* xibs
* objc code
* app shortcut phrases
* c code
* info plists

## 7:36 - Localizable strings in SwiftUI
```swift
// Localizable strings in SwiftUI

struct ContentView: View {
var body: some View {
    VStack {
        Label("Thanks for shopping with us!", systemImage: "bag")
            .font(.title)

        HStack {
            Button("Clear Cart") { }

            Button("Checkout") { }
        }
    }
}
}
```

any parameter accepting a type of `LocalizedStringKey`.

## 8:16 - Localizable strings in SwiftUI custom view
```swift
// Localizable strings in SwiftUI

struct CardView: View {
    let title: LocalizedStringResource
    let subtitle: LocalizedStringResource

    var body: some View {
        ZStack {
            RoundedRectangle(cornerRadius: 10.0)
            VStack {
                Text(title)
                Text(subtitle)
            }
            .padding()
        }
    }
}

CardView(title: "Recent Purchases", subtitle: "Items you’ve ordered in the past week.")
```
`LocalizedStringResource` is the recommended type.  Initialize with a string literal, provide with a comment, tablename, etc.  Different. from string key.


here I have some model code that includes strings that will be represented later.  Here I use `String(localized:)`.

Can also use `LocalizedStringREsource` directly.

catalogs use powerful technology to extract strings.  "Use Compiler to Extract Swift Strings" build settings.



## 9:03 - Localizable strings in Swift displayed at runtime
```swift
// Localizable strings in Swift

import Foundation

func stringsToPresent() -> (String, AttributedString) {
    let deferredString = LocalizedStringResource("Title")

...

return (
    String(localized: deferredString),
    AttributedString(localized: "**Attributed** _Subtitle_")
)
}
```

Here we us the macro
## 9:44 - Localizable strings in Objective-C
```objc
// Localizable strings in Objective-C

#import <Foundation/Foundation.h>

- (NSString *)stringForDisplay {
    return NSLocalizedString(@"Recent Purchases", @"Button Title");
}

#define MyLocalizedString(key, comment) \
[myBundle localizedStringForKey:key value:nil table:nil]
```

CFCoypLocalizedString

## 10:04 - Localizable strings in C
```c
// Localizable strings in C

#include <CoreFoundation/CoreFoundation.h>

CFStringRef stringForDisplay(void) {
    return CFCopyLocalizedString(CFSTR("Recent Purchases"), CFSTR("Button Title"));
}

#define MyLocalizedString(key, comment) \
CFBundleCopyLocalizedString(myBundle, key, NULL, NULL)
```



Use "Localized String macro Names" build settings to specify your own macros.

IB.  Automatically treated as localizable.  Using inspector can also specify comment under "Localizer Hint" field.
Info plist files. Simply add an info plist.xcstrings file to your project, and add to the desired target.  Every time you build, xcode will add known set of info plist keys  to the catalog.


[[Spotlight your app with App Shortcuts]]


## 11:23 - App Shortcut phrases
```swift
// App Shortcut phrases

struct FoodTruckShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: ShowTopDonutsIntent(),
            phrases: [
                "\(.applicationName) Trends for \(\.$timeframe)",
                "Show trending donuts for \(\.$timeframe) in \(.applicationName)",
                "Give me trends for \(\.$timeframe) in \(.applicationName)"
            ]
        )
    }
}
```

Every time you build, we discover localizable strings.

sourcecode -> act as source of truth
source strings from string catalog are kept in sync.  Xcode adds code to string catalog.  now string can be translated.

xcode can also discover when you've removed a string from code.  If not translated, xcode removes it for you.  However, if you've provided translations, xcode will instead mark it as stale.  This indicates that the string could no longer be found in code.  Can delete and translations, if you can confirm that it is no longer needed.

Can also use the inspector to tell xcode that you want to manually manage that string.  These will never be updated or removed by xcode.


# Edit
xcode will show translation progress

* new - hasn't been translated
* stale - not found in code
* needs review - may require change
	* mark as reviewed
	* can mark for review to get into this state
* translated - good to go

pluralization.

vary the stringg based on the value of the number.  Previously, solving this number would have required stringsdict.  This plist format can be tough to use correctly and introduces a fairly high barrier.

Now, the string catalog includes built-in support for string variation workflows.
vary by plural => zero, one, other

consider multiple variables.  AT runtime we could end up with several different scenarios.
string catalog editor can do this too.  Here, we have varied both arguments by plural.  Each substitution stores a dictionary of plural cases and their values.  Split by each variable individually.

we show c-style format specifiers, even for swift code.

recommend renaming substitutions so they're easier to understand.


# Export
localization catalog is a package format that contains all localizable content within a project or workspace.  For now, we'll focus on the xliff file, which contains all localizable strings and their translations.

xliff is a standard format for storing and transporting localizations.

changes to how varying strings are presented.
## 23:53 - Stringsdict in XLIFF
```xml
<!-- Stringsdict in XLIFF -->

<trans-unit id="/%lld Recent Visitors:dict/NSStringLocalizedFormatKey:dict/:string">
    <source>%#@recentVisitors@</source>
    <target>%#@recentVisitors@</target>
</trans-unit>

<trans-unit id="/%lld Recent Visitors:dict/recentVisitors:dict/one:dict/:string">
    <source>%lld Recent Visitor</source>
    <target>%lld Visitante Recente</target>
</trans-unit>

<trans-unit id="/%lld Recent Visitors:dict/recentVisitors:dict/other:dict/:string">
    <source>%lld Recent Visitors</source>
    <target>%lld Visitantes Recentes</target>

</trans-unit>
```


when we originate from catalogs, we do this instead

## 24:08 - String Catalog in XLIFF
```xml
<!-- String Catalog in XLIFF -->

<trans-unit id="%lld Recent Visitors|==|plural.one">
    <source>%lld Recent Visitor</source>
    <target>%lld Visitante Recente</target>
</trans-unit>

<trans-unit id="%lld Recent Visitors|==|plural.other">
    <source>%lld Recent Visitors</source>
    <target>%lld Visitantes Recentes</target>
</trans-unit>
```

## 24:58 - String Catalog variations in XLIFF
```xml
<!-- Overriding variation in XLIFF -->

<trans-unit id="Bird Food Shop|==|device.applewatch">
    <source>Bird Food Shop</source>
    <target>Loja de Comida</target>
</trans-unit>

<trans-unit id="Bird Food Shop|==|device.other">
    <source>Bird Food Shop</source>
    <target>Loja de Comida de Passarinho</target>
</trans-unit>
```

not only should it be easy for automation tooling, but hopefully humans can understand them as well.

can also vary strings taht weren't previously varied.  For example, we have a string that is not varied at all.  but we want a shoerter string for apple watch.  we replace the trans-unit in xliff, and then we influence the variation structure on the next import.

build setting "Localization Prefers String Catalogs" to YES.

can import with product->import localizations...

Run with run options, change app language to desired language for testing purposes.

# Build
Let's discuss the build process.

string catalogs aer designed for interactions inside xcode.  As json files, they're easily diffable.

Then, at buildtime, these compile to `strings` and `stringsdict` files.  Because these have been supported forever... back-deployable to any OS!
Does not include source strings from code!

# Migrate

How to get started with string catalogs in existing projects.

Pick which strings files and targets to migrate.

the % lets us see which strings are nonlocalized.  

Packages: add a `defaultLocalization` to specify the development language.  Can add a string catalog to the package.  Now i can see unlocalized strings.

# Wrap up
* use string catalogs for localization in xcode
* simplify managing translations in your project
* migrate or create a new file to get started



# Resources
https://developer.apple.com/documentation/Xcode/localization
https://developer.apple.com/localization/
