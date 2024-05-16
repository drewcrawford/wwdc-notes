#coretext
Text is an absolutely critical component of every app. Discover the latest features and enhancements for creating rich text experiences on Apple platforms. We'll show you how to take advantage of common text elements and create entirely custom interactions for your app. Learn about updates to dictation, text loupe, and text selection, and explore improvements to text clipping, line wrapping, and hyphenation.

Consume information adn communicate.

Even more tools to create a powerful text experience.  Starting from scratch or high-level abstractiosn.

# Changes in selection UI
Redesigned text cursor.  Inline, interactive switcher.
More ergonomic selection handles
Completely new loupe, to make it easy to place the cursor in large bodies of text.

* Works with `UITextView` and `UITextField`.  Also `UITextInteraction`.
If you have a customized UI, it can be challenging to keep up with changes.

We have a new UITextSelectionDisplayInteraction, which just provides the display without the gestures of the other things.

## UITextSelectionDisplayInteraction
* `UIInteraction` to be installed on any UIView
* must adopt `UITextInput` protocol
Provides the cursor view, range highlight, selection handles, etc.

You can customize the behavior if you need to.

code example not provided.

call `setNeedsSelectionUpdate`.  In addition to UITextSelectionDisplayInteraction, you can also use `UITextLoupeSession` object for displaying loupe
c an be used on any view
use a gesture recognizer to drive loupe updates.




# Text item actions and menus
Text item interactions in UITextview are now more customizable with new APIs on UITextView delegate.  make it possible to modify the primary action for text items.  Candidate menu shown here.

previously, UITextView allows developers to disable item interactions through `shouldInteractWith` APIs.  

* customize the primary action or menu content
* new methods on `UITextViewDelegate`.

## text items
represent the content that supports item interaction.
* NSTextAttachment
* Links -> NSLinkAttributeName
* also supports tagging custom ranges of text, with `UITextItemTagAttributeName`.

ex, add custom menus to parts of the text.  To continue to suppress or disable the primary action, return `nil` for primary action delegate method.


# Lists and bullets
* supports several different kinds of bullets
* automatic number of items
* Localized for every language
* 

# Dictation
New UI.

After scrolling, we draw a cursor arrow pointing to offscreen position.



## NSTextInsertionIndicator
For macOS.  Remain consistent with the system selection UI.
* system-provided insertion point
* customizable
* follows current accent color
* Supports dictation effects

can disable the glow effect.  

basically this is an NSView we insert into the hierarchy.


Adopt `NSTextInputClient` `selectionRect` and `documentVisibleRect` properties.
notify scrolling beginning and ending.  


# Internationalization

Cruicial to providing an outstanding text experience.  Important cahnges to standard text controls, enhancing their readability and ergonomics. Supporting dynamic type goes a long way toward improving the layout of your UI in every language.  Consider, many languages can ahve variable line heights in addition to layout direction and font styles.

Commonly causes clipped text.  

Line heights are now dependent on preferred languages.
Independent of actual language content

Been in effect for several releases.  But in iOS 17, it's much more dynamic.  Depends on text style and language views.  To take advantage of this behavior, use `UIFont.preferredFont(forTextStyle:)`.  

Use `clipsToBounds=false`.  Usually we can expand outside the textview.  UIKit has been updated to prevent unnecessary use of this setting where it was previously enabled by default.

Respond to changes in vertical height

## Wrapping and hyphenation
Improved line breaking for chinese, german, japanese, and korean
Optimized for text style and language
adopt text styles

# What's next
* custom text views can now use system selection UI
* Use text item interactions for better links
* Leverage TextKit 2
* Adopt text styles




# Resources
* https://developer.apple.com/documentation/uikit/keyboards_and_input/adopting_system_selection_ui_in_custom_text_views
* https://developer.apple.com/documentation/appkit/text_display/adopting_the_system_text_cursor_in_custom_text_views
