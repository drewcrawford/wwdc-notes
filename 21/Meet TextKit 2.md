#textkit

TK1 => Drives across all apple platforms

Manage storage and layout
OpenStep.
TK2 => Already used in Big Sur.


Co-exists than textkit 1.  
More *what*, less *how*.

# Design principles
* Correctness => Abstract away glyph handling
* Safety => Value semantics
* Performance=> Viewport-based layout and rendering

## Correctness
Glyph => in wester languages, one character.  But in other languages, single characters are multiple glyphs
Or the reverse. ("ligature")

In some languages, ligatures are required to be legible.

Seems glyphs can modify earlier glyphs.

Not possible to find a glyph range for the first 4 characters.  No single range to represent these characters.  Since many TK1 APIs require a range, this can break rendering.

### TK2 abstracts away glyphs
* Uses Core Text for all rendering
* Provides higher level objects to control layout

### Selection
`NSTextSelection` => Contains context for a text selection.  Granularity, disjoint ranges, etc.

### Selection navigation
`NSTextSelectionNavigation` => perform actions on text selections
Get new selection object(s) as a result

### Operating on selections

`textSelections(interactingAt:...)`

Can extend forward/backward

## Location and range
`NSTextLocation` `NSTextRange`.  Represents range between start and end location.

Location objects more expressive than integers.  Ranges defined by start and end location.   e.g. consider HTML position (tag position)

## Safety

Value semantics != value types

Value tpes => unique copy, prevent mutation
Remove unattended sharing, side effects.

Immutable classes have value semantics.  These classes behave like value types, so refer to them as having *value semantics*.  Must create a new instance.

### TK1 design
1.  Change text
2.  NSTextStorage
3.  NSLayoutManager
4.  Generate glyphs
5.  position glyphs
6.  draw glyphs
7.  UIView or NSView

Difficult to figure out where to separate the text to create spaces for custom drawing.

Drawing inline with custom image etc.  How to do?
Expectation => Divide text into elements
Put comment in its own element
Position comment after recipe
Provide custom drawing for comment

Reality=> ??
* Find character index in text storage
* Convert to glyph index
* Is it on a grapheme cluster boundary???
* Might need to covert to UTF-16 to check
* Adjust the glyph index if needed
* Change the line spacing after the glyph index
* Customize line fragment geometry

### TK2 flow
1.  Change text
2.  NSTextContentManager
3.  Divide into NSTextElement
4.  NSTextLayoutManager gets elements
5.  Lays out into NSTextLayoutFragment
6.  NSTextViewportLayoutController draws into UIView, CALayer etc.

Control layout and delay of text by obtaining information from the immutable objects.

To make changes, create new instances of the value objects and send them back to the system.  

### `NSTextElement`
(abstract) represnets a portion of text content
Contains range describing where it is in the document

Modeling the document as a series of elements is a lot more powerful.  Easily distinguish what kind of content an  element represents (similar to HTML tags)

### `NSTextContentManager`
(abstract)
* Provides elements from text content
* Wrapper for backing store

### Custom needs
* Subclass `NSTextContentManager` when:
* Using custom document model
* Using custom backing store

Usually you can use the default.

### Paragraph and content storage
`NSTextContentStorage`
* Default content manager
* Uses NSTextStorage backing store

`NSTextParagraph`
* default element type
* Represents single paragraph as `NSAttributedString`

### Changing content storage
Push `NSTextcontentStorage` changes through `nSTextContentManager`

use `performEditingTransaction` to wrap updates!

### Content storage delegate
### Text Layout Manager
`NSTextLayoutManager`
* Controls text layout process
* Generates layout fragments from elements

Does not deal with glyphs!  Uses elements instead.

### Text layout fragment
`NSTextLayoutFragment`
* Layout information for one or more elements

* `textLineFragments`
* `layoutFragmentFrame`
* `renderingSurfaceBounds`

To customize or change the layout, need to understand tehse properties

#### Line fragment
`NSTextLineFragment`
* measurement information for a single line
* geometric info for specific line

#### Line fragment frame
Stacked inside text container during layout
Empty lines have their own frame.
In general, fragment frames are useful for positioning other views or calculating total height.

Does not accurately communicate space to draw the text

#### Rendering surface bounds

Area required to draw the  text

**Text can overshoot the edges of the fragment frame**.

Rendering surface bounds may be much larger than line fragment frame!

### Customizing layout fragments
Can't directly change since immutable.

Hook into layout process and create new instances.

NSTextLayoutManagerDelegate
* subclass NSTextLayoutFragment for custom drawing
* Create new layout fragments during layout process

## Performance

Non-contiguous text layout

Visible area.  Contiguous layout starts from the beginning of the document, whereas non-contiguous starts from the visible area.

Lay out anywhere in the document without alying out things before it.  

When you scroll through the document, layout happes for the visible area right away.  Happens for portions of text on the screen.

Noncontiguous layout is optional in TK1.  Enabled with `allowsNonContiguousLayout` but because this is simple, it can't express information about the state of the area.

* Control which part of document gets laid out
* Adjust visible area
* Get notified before/after visible area layout

*viewport*

### Focus on viewport for best performance

Avoid layout information outside the viewport when possible.  Might not be accurate unless you explicitly ask.  Call can be expensive for large documents.

### Vieport layout controller
`NSTextViewportLayoutController`
* Source of truth for viewport layout
* Access via property on text layout manager

### Viewport layout process
How to participate in viewport layout process?

* `willLayout` => clear view or layer contents
* `configureRenderingSurfaceFor` => Update geometry for each fragment.
* `didLayout` => Adjust viewport if needed


# Demo
Sample app.

Only paragraphs visible in the viewport are being drawn.  Each paragraph is rendered into its own layer.  Drawing text into separate layers lets us implement a fun feature.

Animating inserting an inline drawing object.  

* Viewport layout
* Comment display

When layout manager is about to layout... due to viewport changes etc., it calls `willLayout` on delegate.

```swift
func textViewportLayoutControllerWillLayout(_ controller: NSTextViewportLayoutController) {
    contentLayer.sublayers = nil
    CATransaction.begin()
}
```

```swift
private func animate(_ layer: CALayer, from source: CGPoint, to destination: CGPoint) {
    let animation = CABasicAnimation(keyPath: "position")
    animation.fromValue = source
    animation.toValue = destination
    animation.duration = slowAnimations ? 2.0 : 0.3
    layer.add(animation, forKey: nil)
}

private func findOrCreateLayer(_ textLayoutFragment: NSTextLayoutFragment) -> (TextLayoutFragmentLayer, Bool) {
    if let layer = fragmentLayerMap.object(forKey: textLayoutFragment) as? TextLayoutFragmentLayer {
        return (layer, false)
    } else {
        let layer = TextLayoutFragmentLayer(layoutFragment: textLayoutFragment, padding: padding)
        fragmentLayerMap.setObject(layer, forKey: textLayoutFragment)
        return (layer, true)
    }
}

func textViewportLayoutController(_ controller: NSTextViewportLayoutController,
                                  configureRenderingSurfaceFor textLayoutFragment: NSTextLayoutFragment) {
    let (layer, layerIsNew) = findOrCreateLayer(textLayoutFragment)
    if !layerIsNew {
        let oldPosition = layer.position
        let oldBounds = layer.bounds
        layer.updateGeometry()
        if oldBounds != layer.bounds {
            layer.setNeedsDisplay()
        }
        if oldPosition != layer.position {
            animate(layer, from: oldPosition, to: layer.position)
        }
    }
    if layer.showLayerFrames != showLayerFrames {
        layer.showLayerFrames = showLayerFrames
        layer.setNeedsDisplay()
    }
    
    contentLayer.addSublayer(layer)
}
```

```swift
func textViewportLayoutControllerDidLayout(_ controller: NSTextViewportLayoutController) {
    CATransaction.commit()
    updateSelectionHighlights()
    updateContentSizeIfNeeded()
    adjustViewportOffsetIfNeeded()
}
```

## Comment display

```swift
func textContentStorage(_ textContentStorage: NSTextContentStorage, textParagraphWith range: NSRange) -> NSTextParagraph? {
    // In this method, we'll inject some attributes for display, without modifying the text storage directly.
    var paragraphWithDisplayAttributes: NSTextParagraph? = nil
    
    // First, get a copy of the paragraph from the original text storage.
    let originalText = textContentStorage.textStorage!.attributedSubstring(from: range)
    if originalText.attribute(.commentDepth, at: 0, effectiveRange: nil) != nil {
        // Use white colored text to make our comments visible against the bright background.
        let displayAttributes: [NSAttributedString.Key: AnyObject] = [.font: commentFont, .foregroundColor: commentColor]
        let textWithDisplayAttributes = NSMutableAttributedString(attributedString: originalText)
        // Use the display attributes for the text of the comment itself, without the reaction.
        // The last character is the newline, second to last is the attachment character for the reaction.
        let rangeForDisplayAttributes = NSRange(location: 0, length: textWithDisplayAttributes.length - 2)
        textWithDisplayAttributes.addAttributes(displayAttributes, range: rangeForDisplayAttributes)
        
        // Create our new paragraph with our display attributes.
        paragraphWithDisplayAttributes = NSTextParagraph(attributedString: textWithDisplayAttributes)
    } else {
        return nil
    }
    // If the original paragraph wasn't a comment, this return value will be nil.
    // The text content storage will use the original paragraph in this case.
    return paragraphWithDisplayAttributes
}
```

hiding comments, without deleting them from the underlying text storage

```swift
func textContentManager(_ textContentManager: NSTextContentManager,
                        shouldEnumerate textElement: NSTextElement,
                        with options: NSTextElementProviderEnumerationOptions) -> Bool {
    // The text content manager calls this method to determine whether each text element should be enumerated for layout.
    // To hide comments, tell the text content manager not to enumerate this element if it's a comment.
    if !showComments {
        if let paragraph = textElement as? NSTextParagraph {
            let commentDepthValue = paragraph.attributedString.attribute(.commentDepth, at: 0, effectiveRange: nil)
            if commentDepthValue != nil {
                return false 
            }
        }
    }
    return true
}
```

Generating special layout fragments for comments

```swift
func textLayoutManager(_ textLayoutManager: NSTextLayoutManager,
                       textLayoutFragmentFor location: NSTextLocation,
                       in textElement: NSTextElement) -> NSTextLayoutFragment {
    let index = textLayoutManager.offset(from: textLayoutManager.documentRange.location, to: location)
    // swiftlint:disable force_cast
    let commentDepthValue = textContentStorage!.textStorage!.attribute(.commentDepth, at: index, effectiveRange: nil) as! NSNumber?
    if commentDepthValue != nil {
        let layoutFragment = BubbleLayoutFragment(textElement: textElement, range: textElement.elementRange)
        layoutFragment.commentDepth = commentDepthValue!.uintValue
        return layoutFragment
    } else {
        return NSTextLayoutFragment(textElement: textElement, range: textElement.elementRange)
    }
}
```

Drawing the comment bubble

```swift
var commentDepth: UInt = 0

private var tightTextBounds: CGRect {
    var fragmentTextBounds = CGRect.null
    for lineFragment in textLineFragments {
        let lineFragmentBounds = lineFragment.typographicBounds
        if fragmentTextBounds.isNull {
            fragmentTextBounds = lineFragmentBounds
        } else {
            fragmentTextBounds = fragmentTextBounds.union(lineFragmentBounds)
        }
    }
    return fragmentTextBounds
}

// Return the bounding rect of the chat bubble, in the space of the first line fragment.
private var bubbleRect: CGRect { return tightTextBounds.insetBy(dx: -3, dy: -3) }

private var bubbleCornerRadius: CGFloat { return 20 }

private var bubbleColor: Color { return .systemIndigo }

private func createBubblePath(with ctx: CGContext) -> CGPath {
    let bubbleRect = self.bubbleRect
    let rect = min(bubbleCornerRadius, bubbleRect.size.height / 2, bubbleRect.size.width / 2)
    return CGPath(roundedRect: bubbleRect, cornerWidth: rect, cornerHeight: rect, transform: nil)
}

override var renderingSurfaceBounds: CGRect {
    return bubbleRect.union(super.renderingSurfaceBounds)
}

override func draw(at renderingOrigin: CGPoint, in ctx: CGContext) {
    // Draw the bubble and debug outline.
    ctx.saveGState()
    let bubblePath = createBubblePath(with: ctx)
    ctx.addPath(bubblePath)
    ctx.setFillColor(bubbleColor.cgColor)
    ctx.fillPath()
    ctx.restoreGState()
    
    // Draw the text on top.
    super.draw(at: renderingOrigin, in: ctx)
}
```

## More topics in sample code
* Converting clicks, drags, touches to text selection
* Text selection rendering
* Getting layout fragment geometry
* Document height estimation


# Modernization
Now that we've gone over what TK2 can do, let's discuss approaches for modernization?
* All classes available in UIKit and AppKit
* *Some* UIKit and AppKit controls updated to TK2
	* Committed to compatibility
	* Opting in NSTextView
	
```swift
	var scrollView: NSScrollView!
var containerSize = CGSize.zero
var textContainer = NSTextContainer()

// Important: Keep a reference to text storage since NSTextView weakly references it.
var textContentStorage = NSTextContentStorage()

override func viewDidLoad() {
    super.viewDidLoad()
    
    scrollView =
        NSScrollView(frame: NSRect(origin: CGPoint(),
                                   size: CGSize(width: view.bounds.width, height: view.bounds.height)))
    scrollView.translatesAutoresizingMaskIntoConstraints = false
    view.addSubview(scrollView)
    
    NSLayoutConstraint.activate([
        scrollView.leadingAnchor.constraint(equalTo: (view.leadingAnchor)),
        scrollView.trailingAnchor.constraint(equalTo: (view.trailingAnchor)),
        scrollView.topAnchor.constraint(equalTo: (view.topAnchor)),
        scrollView.bottomAnchor.constraint(equalTo: (view.bottomAnchor))
        ])
    
    setUpScrollView(scrollsHorizontally: false)
}

func setUpScrollView(scrollsHorizontally: Bool) {
    scrollView.borderType = .noBorder
    scrollView.hasVerticalScroller = true
    scrollView.hasHorizontalScroller = scrollsHorizontally
    
    setUpTextContainer(scrollsHorizontally: scrollsHorizontally)
    setUpTextView(scrollsHorizontally: scrollsHorizontally)
}

func setUpTextContainer(scrollsHorizontally: Bool) {
    let contentSize = scrollView.contentSize
    if scrollsHorizontally {
        containerSize = NSSize(width: CGFloat.greatestFiniteMagnitude, height: CGFloat.greatestFiniteMagnitude)
        textContainer.containerSize = containerSize
        textContainer.widthTracksTextView = false
    }
    else {
        containerSize = NSSize(width: contentSize.width, height: CGFloat.greatestFiniteMagnitude)
        textContainer.containerSize = containerSize
        textContainer.widthTracksTextView = true
    }
}

func setUpTextView(scrollsHorizontally: Bool) {
    let textLayoutManager = NSTextLayoutManager()
    textLayoutManager.textContainer = textContainer

    textContentStorage.addTextLayoutManager(textLayoutManager)

    // Workaround: Pass textLayoutManager.textContainer to the NSTextView initializer
    let textView = NSTextView(frame: scrollView.contentView.bounds, textContainer: textLayoutManager.textContainer)

    textView.isEditable = true
    textView.isSelectable = true
    textView.minSize = CGSize()
    textView.maxSize = containerSize
    textView.isVerticallyResizable = true
    textView.isHorizontallyResizable = scrollsHorizontally
    textContentStorage.performEditingTransaction {
        textView.textStorage?.append(NSAttributedString(string: "Text content..."))
    }
    scrollView.documentView = textView
}
```

Basically you want to use `NSTextView(frame:, textcontainer:)` where your text container is the one of some `NSTextLayoutManager`

Access layout manager and content storage with new properties.  

Recall that `NSTextView` has a `layoutManager`.  This is a TK1 object and is not compatible with TK2.  TextView can't have both a `layoutManager` and a `textLayoutManager`.

Special compatibility mode
* Automatically switches to use TK1
* Replaces NSTextLayoutManager with NSLayoutManager

TK will remain in compatibility mode from that point forward.  Even if you opted into TK2, it will switch back to TK1 if you access `layoutManager`.  Or if it encounters an unsupported format.

## Field editors


Field editors.  Uses TK2 by default, but can switch to compatability mode

```swift
fieldEditor.layoutManager //forces compatibility mode
fieldEditor.textContainer.layoutManager //similarly
```

Issues notifications to switch to TK1
`willSwitchToNSLayoutManagerNotification`.

* UITextField uses TK2 automatically
* Evaluate existing code for uses of `.layoutManager`

https://developer.apple.com/documentation/uikit/textkit/using_textkit_2_to_interact_with_text
https://developer.apple.com/documentation/appkit/textkit
