Discover how to bring Genmoji to life in your app. We'll go over how to render, store, and communicate text that includes Genmoji. If your app features a custom text engine, we'll also cover techniques for adding support for Genmoji.

### Enable support for NSAdaptiveImageGlyph in a UITextView - 3:30
```swift
let textView = UITextView()
textView.supportsAdaptiveImageGlyph = true
```

### Read and write attributed string for serialization - 4:41
```swift
// Extract contents of text view as an attributed string
let textContents = textView.textStorage

// Serialize as data for storage or transport
let rtfData = try textContents.data(from: NSRange(location: 0, length: textContents.length), documentAttributes: [.documentType: NSAttributedString.DocumentType.rtfd])

// Create attributed string from serialized data
let textFromData = try NSAttributedString(data: rtfData, documentAttributes: nil)

// Set on text view
textView.textStorage.setAttributedString(textFromData)
```

### Convert NSAttributedString to HTML - 6:30
```swift
// Converting NSAttributedString to HTML
let htmlData = try textContent.data(from: NSRange(location: 0, length: textContent.length), documentAttributes: [.documentType: NSAttributedString.DocumentType.html])
```

### Support Genmoji in communication notifications - 7:33
```swift
func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
    ...
    let message: NSAttributedString = _myAttributedMessageStringWithGlyph
    
    let context = UNNotificationAttributedMessageContext(sendMessageIntent: sendMessageIntent, attributedContent: _message)
    
    do {
        let messageContent = try request.content.updating(from: context)
        contentHandler(messageContent)
    } catch {
        // Handle error
    }
}
```

### Render NSAdaptiveImageGlyph in custom typesetting solution - 9:45
```swift
// Find typographic bounds for image in NSAdaptiveImageGlyph
let provider = adaptiveImageGlyph
let bounds = CTFontGetTypographicBoundsForAdaptiveImageProvider(font, provider)

// Draw it at the typographic origin point on the baseline
CTFontDrawImageFromAdaptiveImageProviderAtPoint(font, provider, point, context)
```

# Resources
* https://developer.apple.com/documentation/appkit
* https://developer.apple.com/documentation/uikit
* https://developer.apple.com/documentation/webkit/wkwebview
