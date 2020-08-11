# Universal Links
URLs that represent your content both on the web and in your app
Add the associated-domains entitlement. 
Recommended over custom URL schemes.
[[What's new in universal links - 19]]

# New Platform Support

#watchOS 
Same functionaltiy as on iOS, tvOS, and macOS

API differences between UIKit and watchkit
Add the associated domains entitlement to your watchkit extension, *not to the containing app*.

Use `WKExtensionDelegate` `handle`
And to open in another app, use `WKEXtension.shared().openSystemURL()`

If you open a universal link on an app that's not installed, it will fail to open.

Safari isn't available on watch, so we will present UI to the user.

Different for appkit, etc.  

#SwiftUI is adding support this year.

```swift
//handle universal links
.onOpenUrl { url in /* ... */}

//open in another app
@Environment(\.openURL) var openURL
let url = /* */
openURL(url)
```

[[what's new in swiftui]]

# Enhanced pattern-matching
`"*"` matches zero or more characters greedily
`"?"` matches any one character.
`"?*"` matches *at least one* character

See [[What's new in universal links - 19]] for more

## Case-insensitive patterns

`"components": [{"/": "/sourdough?*", "caseSensitive": false}]`

Available in recent OS releases

## Unicode patterns
need `"percentEncoded": false`

Big Sur, iOS 14

## defaults

Can now add `"defaults": {"percentEncoded": false, "caseSensitive": false},` to the top.

Available in iOS 14 etc.

## Pattern-matching in practice
`https://www.example.com/en_US/burrito/`

`"components":[{"/": "/??_??/?*"}]`
Too broad

Hard-code regions?  Products?  It's combinatorial!

**Pattern matching has exponential complexity**

## Substitution variables
Named lists of possible substrings to match against

Names can contain any character expect `"$"`, `(` or `)`
Values can contain `?` and `*` but not other variables
Case-sensitive by default, but will respect your setting

`$(alpha)*`: All upper-case and lower-case variables
`$(upper),$(lower)` uppercase or lowercase
`$(alnum)` 
`$(digit), $(xdigit)` digit, hexdigit

These are all equivalent to the C standard library `isalpha`, `isupper`, etc.

`$(region)`: "CA","UK","US", etc.  Note, this doesn't work for domain names.
`$(lang)`: Every iso language code.

Also has `exclude` patterns.

Substitution variables are available today in recent OS releases.

# associated domains: what if something goes wrong?
Inconsistent state: app is installed, but associated domains are not available.

Going forward, your associated domains will be getting requests proxied through apple.

[[What's new in managing apple devices]]

Opt into developer mode from Developer Settings in system settings.  Allows any valid certificate to be used, not necessary one in the trust-store.  Self-signed I guess?

Only takes effect for apps that are signed for `Development`.  Not testflight, notarized mac apps, etc.

Must use `.well-known` directory.

Separate entilement entries for developer mode
`?mode=developer`
`?mode=developer+managed`
