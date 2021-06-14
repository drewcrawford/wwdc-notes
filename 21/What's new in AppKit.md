#appkit #macOS 

# Design updates
Continued to iterate and refine the Big Sur design.

New touches:
* Popovers that appear and recede
* Sliders that smoothly glide into position
* Refined metrics and toolbar controls
* Springloaded in search button
* Spacing between table sections


# Control enhancements
## Control tinting
Custom tinting of individual buttons, segmented controls, and sliders.  
`bezelColor`
`selectedSegmentColor`
`trackFillColor`

Starting in macOS monterey, theyr'e functional for in-window controls as well.

Usually do with accent color.
Customizable by the user
Define a custom accent color to theme your app.

New tinting API provides a way to override the color per-control

* Best for semantically meaningful colors
* Tinted buttons are always colorful
* Avoid confusion with the defualt button
* Indicate purpose with more than just color.

Puts your tint color front and center.  However, take care in your design not to create confusion with the default button.

Buttons no longer highlight usng the accent color.

Use interior background style if you're doing custom drawing.  

New push button design.  Supports multiple lines.  Newly expanded style offers flexibility for special cases.

## Localized keyboard shortcuts
* Many keyboard layouts
* Not all shortcuts are easy to type on all keyboards
* Some shortcuts should change for RTL languages

Now AppKit can do it for you.

in us english, Cmd+\.  fine on US, but impossible to type on japanese, which doesn't have a `\` key.  Remapped automatically.

Keyboard shortcut with directional meaning.  e.g. cmd-\[ or cmd-\] for forward/back.  Applies to brackets, braces, parentheses, and arrow keys.

Maybe you want to disable?  e.g. if your menu has absolute directionality.  Control with properties on NSMenuItem

* `allowsAutomaticKeyEquivalentMirroring`
* `allowsAutomaticKeyEquivalentLocalization`

Global opt-out via application delegate
For apps with heavily customized keyboard bindings
Most apps should use the menu item APIs

`applicationShouldAutomaticallyLocalizeKeyEquivalents`.  Menu item APIs are preferred.  Most apps shouldn't opt-out.


# Symbol images
SFSymbols.  In macOS Monterey, have SFSymbols3.  

* Expanded capabilities in SFSymbols app
* Updated authoring format for custom symbols
* Layered symbols
* New APIs for assigning color to symbol layers

Previously,
* Template.  Single tint color
* Multicolor.  Full-color image per path element, defined in symbol image

Now
* Hierarchical.  Emphasis on specific parts.
* Palette.  Assign any color to each layer of the symbol.

`NSImage.SymbolConfiguration`.

## Symbol variants
New API for transforming symbols between variants

e.g. heart => filled.  => circle, => slash, etc.

When you prefer a particular style of symbol per context.   Improves use of controls because you can talk about the variant independently of the base symbol

[[Design and build SF Symbols]]

# TextKit2
The text layout and redering engine.
* A great text engine with a long track record
* Need for non-linear text layout
* So, we built a new TextKit

* Consistent international text experience
* Easier to mix text with visual content
* Super fast layout and rendering
* Coexists with TextKit 1.

Starting in Big Sur, textedit already used it.  You've been getting a sneak peek this whole time.

## Non-linear
Can perform text layout at a granular level, avoidingunnecessary work.

e.g., if only part of the text is visible, don't layout offscreen text.  

[[Meet TextKit2]]
* Customization is simple and powerful
* Easy to incorporate non-text elements
* Improved performance for large documents


# Swift #swift 
* async/await #async 
* Actor types

Methods transformed for async/await.  

NSColorSampler is now async, because it waits for user to pick color.

With async/await, you can express this as an async function call.

Nearly all AppKit accesses must be isolated
Introducing `@MainActor`
In AppKit, we designated `NSResponder`, `NSView`, etc., `NSCell`, `NSAlert`, and more as main actors.

* SAfe to call methods from within the main actor
* Async code can't invoke UI code directly
* Use async/await to run code on the `@MainActor`.

[[Meet asyncawait in Swift]]
[[Protect mutable state with Swift actors]]

## struct `AttributedString`
 New value type for attriuted string
 AppKit provides attributes relevant to the text system
 Converts to and from NSAttributedString
 
 [[What's new in Foundation]]
 
 ## View invalidation
 New property wrapper for driving view updates
 Reduces a common type of boilerplate
 
 View properties contain a lot of `didSet`, side effects, etc.  

We made this better by creating a new propertywrapper.  `@Invalidating`.  Specify one or more aspects of the view to invalidate when the wrapped property changes.  e.g. `display`, `layout`, etc.  

* Display
* Layout
* constraints
* Intrinsic content size
* restorable state

Works with `NSView` subclasses
Value must conform to Equatable
Extensible via NSViewInvalidating protocol

# Shortcuts

Full power of shortcuts to the mac.  Integrating shortcuts with appkit.

* Works just like services
* Queried from the responder chain
* Based on the types of data your app can provide or receive

`validRequestor`.  REturn an object returning to `NSServicesMenuRequestor`.  

Once shortcut is invoked, get calls to write or read data.

# Siri intents
Siri intents have come to macOS
Create an intents extension in xcode

Handle intents via `NSApplicationDelegate`.

Check intents framework for more details on implementing return values

# Next steps
* Explore control tinting and SFSymbols 3
* Update a custom text experience to TextKit 2
* Plan for swift concurrency
* Add support for Shortcuts automation

