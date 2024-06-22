Ensure your app works properly and effectively for multilingual users. Learn best practices for text input, display, search, and formatting. Get details on typing in multiple languages without switching between keyboards. And find out how the latest advances in the String Catalog can make localization even easier.

### Specify textInputContextIdentifier - 3:18
```swift
override var textInputContextIdentifier: String? { uniqueID }
```

### Place a view directly above the keyboard - 3:41
```swift
textView.inputAccessoryView = viewAboveKeyboard
```

### Use keyboardLayoutGuide to adapt to keyboard - 4:00
```swift
view.keyboardLayoutGuide.topAnchor.constraint(equalToSystemSpacingBelow: textView.bottomAnchor, multiplier: 1.0).isActive = true
```

### Check for marked text before modifying - 4:42
```swift
if textView.markedTextRange.empty { // Perform actions involving editing text }
```

### Use localizedStandardRange when searching - 5:58
```swift
let range = text.localizedStandardRange(of: search)
```

### Use color differences to highlight text - 7:24
```swift
attributedString[range].foregroundColor = highlightColor
```

### Text Styles - 9:39
```swift
// Text Styles
// SwiftUI
Text("Hello, world!") // uses .body Text Style by default
Text("Hello, world!").font(.title)

// UIKit
let label = UILabel()
label.text = "Hello, world!"
label.font = UIFont.preferredFont(forTextStyle: .body)

// AppKit
let textField = NSTextField(labelWithString: "Hello, world!")
textField.font = NSFont.preferredFont(forTextStyle: .body)

// Keep clipsToBounds off
clipsToBounds = false
```

### Typesetting language - 10:03
```swift
// Typesetting language
// SwiftUI
Text(verbatim: "Hello, world!").typesettingLanguage(.init(languageCode: .english))

// UIKit
let label = UILabel()
label.text = "Hello, world!"
label.traitOverrides.typesettingLanguage = Locale.Language(languageCode: .english)
```

### Formatting names - 10:29
```swift
// Formatting names
let nameComponents = PersonNameComponents(givenName: "瑗珺", familyName: "汪", nickname: "珺珺")

// Short Name (respects settings like “Prefer Nicknames”)
let shortName = nameComponents.formatted(.name(style: .short)) // 珺珺

// Abbreviated Name (can be used for monograms)
let monogram = nameComponents.formatted(.name(style: .abbreviated)) // 汪
```

### Examples of personalizing text - 12:20
```swift
"^[Nuestro %@](inflect: true) está ^[hecho](agreeWithArgument: 1) de %@."
"अगर आप पहुँच नहीं ^[पाते हैं](inflect: true)"
"예: ‘^[%@을](inflect: true) 켤 때’"
```

### Format numbers using Text - 13:43
```swift
Text("\(numberOfDays)-day forecast")
```

### Format numbers using AttributedString - 14:21
```swift
AttributedString(localized: "10-day forecast")
AttributedString(localized: "0.5× zoom")
```

### Launch to your app’s settings - 15:23
```swift
// Launch to your app’s settings
if let url = URL(string: UIApplication.openSettingsURLString) {
    await UIApplication.shared.open(url)
}
```
# Resources
* https://developer.apple.com/localization/
* 