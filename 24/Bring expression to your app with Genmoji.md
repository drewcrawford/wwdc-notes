Discover how to bring Genmoji to life in your app. We'll go over how to render, store, and communicate text that includes Genmoji. If your app features a custom text engine, we'll also cover techniques for adding support for Genmoji.

# Express yourself
* expressive
* Versatile
* Playful
* Enhances text

# Emoji enhancements

* Standard emoji
* Personalized content
	* Stickers
	* memoji
	* animoji
	* Genmoji

Traditional emoji are a standardized list of plain text.  Up to viewing device to render things.

| x                       | emoji | personalized images |
| ----------------------- | ----- | ------------------- |
| unicode standard        | yes   | no                  |
| sent as text            | yes   | no                  |
| formats with text       | yes   | yes                 |
| consistent presentation | no    | yes                 |
| personalized            | no    | yes                 |
| image based             | no    | yes                 |

# NSAdaptiveImageGlyph

Brand new API to support using GenMoji and other personalized images.
* standard image format
* Square aspect ratio
* Multiple resolutions
* Augmented with metadata

Use
* individually
* inline with text
* Respects line highlight and text formatting
* copy/paste
* Used as stickers
* Anywhere that supports rich text!

# Adopting in your app

* Works with all rich text views
* As easy as enabling `supportsAdaptiveImageGlyph`.
### Enable support for NSAdaptiveImageGlyph in a UITextView - 3:30
```swift
let textView = UITextView()
textView.supportsAdaptiveImageGlyph = true
```
* On iOS, your text view has a paste configuration that supports image, won't even have to do that as those configurations enable by default.
on macOS, declare `importsGraphics`.

Natively supported by NSAttributedString

`NSAttributedString(adaptiveImageGlyph:,attributes:)`

All system serialization is supported
* rtfd
* NSSecureCoding
* UIActivity/UIPasteboard
* NSPasteboard
* HTML <->NSAttributedString

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


Remembe,r Genmoji are not unicode!!

Not appropriate for some data

* identifiers
* phone numbers
* email addresses

however 
* rich text
* display titles
* user messages

Treat NSAdaptiveImageGlyphs similarly to inline images.
Suggestions
* Use Unicode `NSAttachmentCharacter (0xFFFC)`
* maintain reference to `contentIdentifier`
* Cache by `contentIdentifier` for performance

Example code.
* `decomposeAttributedString`
* `recomposeAttributedString`




### Convert NSAttributedString to HTML - 6:30
```swift
// Converting NSAttributedString to HTML
let htmlData = try textContent.data(from: NSRange(location: 0, length: textContent.length), documentAttributes: [.documentType: NSAttributedString.DocumentType.html])
```


This emits cross-compatible HTML so advanced engines that support apple stuff, e.g. webkit inline text.

For other engines, fallback images are displayed instead.  Notice that the `alt` text is sourced from `content-description`.  

Notifications:
`UNNotificationAttributedMessageContext` available for `INSendMessageIntent`.
Payload just needs to contain rich text representation.



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

we recommend a notification extension to parse rich text, etc.
# Compatibility
* Meaning can be lost or altered if not properly constructed
* Not supported as widely as emoji
* Applications that do not support inline images, consider using a text representation such as a content description
* Block opening on unsupported systems

Unique entity inside RTFD
Falls back to text attachment

| runtime            | sdk version       | x                         |
| ------------------ | ----------------- | ------------------------- |
| iOS18,macosSequioa | iOS18,macosSequia | yes, NSAdaptiveImageGlyph |
| iOS18, macosSequia | Earlier           | text attachment           |
| earlier            | -                 | text attachment           |

# Advanced usage
Image glyphs can be fully supported by third-party data formats and custom text engines. 

* NSTextView
* UITextView
* WKWebView
* SwiftUI.Text

Custom text engines supported by textkit  (NSAttributedString(drawin: CGRect))

Coretext:
* Integrated into CTLine

### Render NSAdaptiveImageGlyph in custom typesetting solution - 9:45
```swift
// Find typographic bounds for image in NSAdaptiveImageGlyph
let provider = adaptiveImageGlyph
let bounds = CTFontGetTypographicBoundsForAdaptiveImageProvider(font, provider)

// Draw it at the typographic origin point on the baseline
CTFontDrawImageFromAdaptiveImageProviderAtPoint(font, provider, point, context)
```


`CTFontGetTypographicBoundsForAdaptiveImageProvider` returns metrics for line layout.  When it comes time to draw,
`CTFontDrawImageFromAdaptiveImageProviderAtPoint` will position the image within those bounds.  


# Wrap up
* new emoji keyboard
* Genmoji
* NSAdaptiveImageGlyph


# Resources
* https://developer.apple.com/documentation/appkit
* https://developer.apple.com/documentation/uikit
* https://developer.apple.com/documentation/webkit/wkwebview
