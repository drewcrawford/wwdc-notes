Last year we introduced TextKit2.

The viewport-based layout delivers high-performance layout, esp for long docs.  Better internationalization support.  Full uspport for modern font like opentype, variable fonts.  Makes it easier to customize the layuot of your text.

Moving forward, TK2 engine forms the foundation of text layout and rendering on al platforms.  Each performance enhancements, updates, and improvements will all be focused on v2.  

* learn the fundamentals
* explore sampel code
[[Meet TextKit 2]]

Fundamentals of your layout components.  In ocntrast, this video covers the latest advancements and how to get the most out of text views.

All text controls in UIKit and appkit are using tk2, including `UITextView`.  we're using tk2 for layout/rendering everywhere.

Improtant to transition ASAP.  zero-code transition and we expec thtis to be true for apps that don't make any special modifications to their text views.

# What's new in TextKit 2
* all text controls use tk2
* automatic opt in for uitextview

in iOS 16, all text controls use by default.  A few situations where textview's might not get opted in, will cover later.

appkit => similar, longer story.

textedit uses textkit2 everywhere in ventura.  Previuosly only in plain text mode.  

Added convenience constructors for UITextView and NSTextView.  Can choose at init time which engine to use. `true` for TK2.  `false` for TK1.

for IB, use "Text Layout".  Default => system default, tk2.  You can choose tk1 or tk2 explicitly.

"Non-simple text containers" to wrap around images or inline content.  Use `exclusionPaths` to create these.

Traditional line breaking.  Even line breaking.  Notice stretched-out lines and large inter-word spacing.  Less of that in the new mode.  Text easier to read, and we did it for free with TK2.

* NSTextList available in both AppKit and UIKit.
* Programmatically create numbered or bulleted list in a textview
* Used to be in AppKit only.

`NSTextList`.  put it in a paragraph style.  

There are a few new textkit2 additions.
* Text elements can be structured as trees

NSTextListElement.  Content manager generates these items to represent items in list.

To get a more in-depth view of lists and items,r efer to sample code.  

Attachments.  Use a NSView inside text.  Makes event handling with text attachment easier.


# Compatability mode
We understand full adoption may take some time.  

When you explicitly call an NSLayoutManager, TK replaces itself with TK1.  Also happens if the text view encounters attributes not yet supported, such as tables, or when printing

Log message generated at fallback time
Break on `_UITextViewEnablingCompatabilityMode`.  

There are notifications posted at fallback time.  `willSwitchToNSLayoutManagerNotification` and similar.

Can opt out programmatically at initialization time.  Use you rown text container and a tk1 layout manager.

Another option is to use the conenience constructor passing `false` as discussed.  This forces TK1.  Can also opt out with IB.

Be careful.  If you're swapping out text container's layout manager during or after initialization, we fall back.  It's inefficient to create TK2 objects during init only to throw them away.  Potential side effects depending on timing.  e.g. during typing, could lose focus, etc.  Avoid this by opting out at init time.

# modernization strategies
Only **one layout manager per text view**.  NSTextLayoutManager and an NSLayoutManager at the same time.  

No automatic way to going back to tk2.  You will lose any UI state, etc.  So for optimum perf, system will never switch back to TK2.  It's a one way operation.

**Avoid compatability mode.**

reasons for fallback:
* accessing layoutManager
* less common:
	* unsupoprted attributes
	* printing

**Avoid accessing the layoutManager property.** or the textContainer's layout manager.

use this pattern when deploying to old OS
```swift
if let textLayoutManager = textView.textLayoutManager {
    // TextKit 2 code goes here
}
else {
    let layoutManager = textView.layoutManager    
    // TextKit 1 code goes here
}
```

tk1 code only runs when tk2 is not available, and your layout manger query won't cause fallback.

please report feedback.  Capture stack traces from these breakpoints
* `_UITextViewEnablingCompatabilityMode` (UIKit)
* `_willSwitchToNSLayoutManagerNotification` (AppKit)

## Updating NSLayoutManager code
Some API have simlar names between tk1/tk2.

| NSLayoutManager (TK1)                               | NSTextLayoutManager (TK2)             |
| --------------------------------------------------- | ------------------------------------- |
| usedRect(for:)                                      | usageBoundsForTextContainer           |
| `addTemporaryAttribute(_:value:forCharacterRange:)` | `addRenderingAttribute(_:value:for:)` |
| `removeTemporaryAttribute(...)`                     | removeRenderingAttribute(...)         |
| setTemporaryAttributes(...)                         | setRenderingAttributes(...)           |
| temporaryAttribute(...)                             | enumerateRenderingAttributes(...)     |

some tk1 apis have no direct equivalents.  To understand why, there is no correct character-to-glyph mapping for many scripts.

TK1 assumes you can associate contiguous characters with contiguous glyphs.  That isn't true.

**0 glyph apis in tk2**.

Replacing these APIs requires a different approach:
1.  Identify glyph APIs to be replaced
2. Define the high-level task
3. Identify TK2 structures to use


```swift
// Example: Updating glyph-based code 

var numberOfLines = 0
let textLayoutManager = textView.textLayoutManager

textLayoutManager.enumerateTextLayoutFragments(from:
                                               textLayoutManager.documentRange.location,
                                               options: [.ensuresLayout]) { layoutFragment in
        numberOfLines += layoutFragment.textLineFragments.count
}
```

high level task: counting number of lines.
so use NSTextLineFragment and NSTextLayoutFragment.


## Updating NSRange code
TK1 uses NSRange to index into text content.  NSRange is a linear index into a string.

The linear model doesn't work for indexing into any content that has more structure than a string.e.g. html documents are a tree where each tag is a node in the tree.  If it's part of an HTML document, no way our NSRange to tell us which tag it's in.

TK2 uses NSTextRange.  
NSTextLocation => object representing a location within text content
NSTextRange => range between start and ned NSTextLocatin objects
end excluded from range

ex, "dom node plus some offset".  Any custom object can be a location.
Crucial infrastructure for various backing stores etc.

**Text views still use NSRange**.  

Convert between NSRange and NSTextRange.

Define the location as the integer index into the string.
location => start location: nsRange.location
length => end location: nsRange.location + nsRagne.length

In practice, use the content manager to compute the locations.

```swift
let textContentManager = textLayoutManager.textContentManager

let startLocation = textContentManager.location(textContentManager.documentRange.location, 
                                                offsetBy: nsRange.location)!

let endLocation = textContentManager.location(startLocation, 
                                              offsetBy: nsRange.length)

let nsTextRange = NSTextRange(location: startLocation, end: endLocation)
```
To go from NSTextRange=>NSRange,

```swift
let textContentManager = textLayoutManager.textContentManager

let location = textContentManager.offset(from: textContentManager.documentRange.location,
                                         to: nsTextRange!.location)

let length = textContentManager.offset(from: nsTextRange!.location,
                                       to: nsTextRange!.endLocation)

let nsRange = NSRange(location: location, length: length)
```

For UITextRange=>NSTextRange,
 use integer offsets as intermediary.  between UITextRange andd NSTextRange.
 
```swift
let offset = textView.offset(from: textview.beginningOfDocument, to: uiTextRange.start)

let startLocation = textContentManager.location(textContentManager.documentRange.location, 
                                                offsetBy: offset)!

let nsTextRange = NSTextRange(location: startLocation)
```

For custom views as UITextInput, you have control.  Make your uITextPOsition subclass conform to NSTextLocation.  Implement the required method, and use your subclass to create NSTextRange directly.

avoid ureusing UITextPosition across different views
UITextPosition is valid **only** for the view that created it.

# next steps
* use textkit 2 in your app
* check for unintended fallback
* modernize your app


* https://developer.apple.com/forums/tags/wwdc2022-10090
* https://developer.apple.com/forums/create/question?&tag1=323&tag2=358030
