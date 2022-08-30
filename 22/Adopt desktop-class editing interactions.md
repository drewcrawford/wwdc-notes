#ipados 

iPad is continuously evolving.  In this video, you'll learn about the exciting new editing interactions. 

# Edit menu
Features an all-new design that remains familiar but is more interactive and easier to discover actions.  Now has alternate presentations based on the input method used.

For touch interaction, edit menu still has familiar compact appearance, but with an improved paging behavior allowing actions to be more discoverable than before.  With magic keyboard or a trackpad, a context menu is presenetd on secondary or rightclick for a more desktop-class experience.

iPhone will display new edit menu.  Mac Catalyst, context menus that mac users are familira with is presented.

In iOS 16, text edit menu gets a major powerful with new data detectors integration
* unit and currency conversions
* smart lookup
* no adoption needed!

If you select an address in safari, you will get maps-based actions.  On top of existing edit menu actions.  All edit menus.

## Adding items into text edit menus
```swift
func textView(
    _ textView: UITextView, 
    editMenuForTextIn range: NSRange, 
    suggestedActions: [UIMenuElement]) -> UIMenu?
```



Similar methods on UITextFieldDelegate, UITextInput, etc.  Please note that inserting menu items using UIMenuController is now deprecated in iOS 16.  Use the new methods to add menu items in your text edit menus.  

> Where we're going, we don't need menu controllers!
```swift
func textView(
    _ textView: UITextView,
    editMenuForTextIn range: NSRange,
    suggestedActions: [UIMenuElement]
) -> UIMenu? {
    var additionalActions: [UIMenuElement] = []
    if range.length > 0 {
        let highlightAction = UIAction(title: "Highlight", ...)
        additionalActions.append(highlightAction)
    }
    let insertPhotoAction = UIAction(title: "Insert Photo", ...)
    additionalActions.append(insertPhotoAction)
    return UIMenu(children: suggestedActions + additionalActions)
}
```

Append my actiosn to the system suggested actions.  Includes items like cut, copy, paste, and return the menu.  That's it.

## UIEditMenuInteraction
* programmatic edit menu presentation
* context menu presentation on secondary click
* Replaces `UIMenuController` **which is deprecated**.

```swift
let editMenuInteraction = UIEditMenuInteraction(delegate: self)
view.addInteraction(editMenuInteraction)

let tapRecognizer = UITapGestureRecognizer(target: self, action: #selector(didTap(_:)))
tapRecognizer.allowedTouchTypes = [UITouch.TouchType.direct.rawValue as NSNumber]
view.addGestureRecognizer(tapRecognizer)

@objc func didTap(_ recognizer: UITapGestureRecognizer) {
    let location = recognizer.location(in: self.view)
    if self.hasSelectedObjectView(at: location) {
        let configuration = UIEditMenuConfiguration(identifier: nil, sourcePoint: location)
        editMenuInteraction.presentEditMenu(with: configuration)
    }
}
```

Here we configure `allowedTouchTypes` to forbid indirect pointer events.

Determine performable actions in the interactions' view to display in the menu.  Once ocnfigured, call `presentEditMenu(with:)` to show the menu.

We want to offset the menu to not block the content.  And add a new item.

```swift
func editMenuInteraction(
    _ interaction: UIEditMenuInteraction,
    targetRectFor configuration: UIEditMenuConfiguration
) -> CGRect {
    guard let selectedView = objectView(at: configuration.sourcePoint) else { return .null }
    return selectedView.frame
}

func editMenuInteraction(
    _ interaction: UIEditMenuInteraction,
    menuFor configuration: UIEditMenuConfiguration,
    suggestedActions: [UIMenuElement]
) -> UIMenu? {
    let duplicateAction = UIAction(title: "Duplicate") { ... }
    return UIMenu(children: suggestedActions + [duplicateAction])
}
```

Return the frame that we don't want to occlude.  I guess.

When I tap again on the selected view, menu no longer occludes, presenting around it.  Duplicate action is also included.

## mac catalyst support
* presents context menus on right click
* programmatic presentation
	* bridges on iPad idiom
	* Not supported on Mac idiom

## Using UIMenu with edit menu
* built on the UIMenuElement family of API
* Uses UIMenuSystem.context
[[Modernizing your UI for iOS 13]]
[[Take your iPad apps to the next level]]

## UIMenu enhancements

UIMenu now has a preferred element size letting you choose different layouts int he context menu.

* small
* medium => a bit more detail.  Used by text edit menu
* large => default, full-width apeparance

`keepsMenuPresented`:

```swift
UIAction(title: "Increase",
         image: UIImage(systemName: "increase.indent"),
         attributes: .keepsMenuPresented) { ... }

UIAction(title: "Decrease",
         image: UIImage(systemName: "decrease.indent"),
         attributes: .keepsMenuPresented) { ... }
```

this is for thigns like "increase size" i guess.

## Edit menu
* customize text edit menu elements
* add menu element titles and images
* move to UIEditMenuInteraction

# Find and replace
new in iOS 16, we itnroduced a new UI component for finding and replacing text.  Included with many of the built-in apps and flex muscle memory with commonly-used editing shortcuts.

Software keyboard, hardware keyboard.

On iPhone, we adapt to small screen size by using compact layout.  Dismissabl, minimization, keyboard avoidance.

On mac, we present the find panel inline with your content, behaving just like appkit.  

## Find with system views
* uitextview
* wkwebview
* pdfview
* quicklook

```swift
open var findInteraction: UIFindInteraction? { get }
textView.isFindInteractionEnabled = true
```

that's it

If you're using quicklook to display text, already available without any work from you.

## Find invocation
Keyboard shortcuts (cmd-f,cmd-g) provided as system shortcuts
find menu in macOS menu bar
avialble when text view becomes first repsonder
findInteraction.presentFindNavigator(showingReplace:) to invoke programmatically.

## Find panel + mac catalyst
* floating on iOS
* embedded inlineo n macOS
* scroll view insets adjusted automatically
* adapts to trait collection changes
* make sure there's enough room

## Find options
* accessible via the magnifying glass menu
customizable via `optionsMenuProvider`

Moer important with custom implementations.

You can still add the find interaction to your app with custom views

## Custom views
* `UIFindInteraction` installable on any view
* Works with existing find implementations
* Easy to adopt if you already use `UITextInput`.

After installing UIFindInteraction, set up a find interaction delegate.  The find interaction delegate, besides being notified about when a find session begins/ends, is responsible for dealing out UIFindSessions.  These sessions are an abstract base class taht enapsulates all of the state for a given session, such as the highlighted result.  Services all actions requested from the UI.

You can choose to vend a subclass of `UIFindSession`.  This is a good option if you alreayd have an existing find/replace implementation in your app and want to bridge to system UI.  Otherwise, let the system take care of the state for you, and adopt the `UITextSearching` protocol on whatever class encapsulates the content.

Return a `UITextSearchingFindSession`.  This is the best option if you don't yet have a find implementation for your custom view.  example

```swift
let customDocument = MyDocument(string: "")
lazy var customView = MyTextView(document: customDocument)

lazy var findInteraction = UIFindInteraction(sessionDelegate: self)

override var canBecomeFirstResponder: Bool { true }

override func viewDidLoad() {
    customView.addInteraction(findInteraction)
}

func findInteraction(_ interaction: UIFindInteraction, sessionFor view: UIView) -> UIFindSession? {
    return UITextSearchingFindSession(searchableObject: customDocument)
}
```

Either your VC or custom view can become first responder to keyboard shortcuts work as expected.  Create find interaction, provide a session delegate to deal out sesisons.  Here, VC is the session delegate.

When asked for a find session, just return a enw UITextSearchingFindSession, providing your document as the searchable object.  Ensure your document class conforms to UITextSearching protocol.

Aggregator is another abstract class.  Could be, DOM range if you are webkit, for example.  

Must decorate results for a given style, when "decorate" is called.

Support multiplexing across multiple visible documents.  e.g. mail's conversation view.  Can install a single find interaction on the root-level collection view, and perform a find across all docs at the same time.

# Find interaction
* enable `isFindInteractionEnabled`
* Move to `UIFindInteraction`
* `UITextSearching` for new implementations
* Check for keyboard shortcut conflicts (cmd-f,cmd-g, etc.)

That's waht it takes to refresh your interactions for iOS 16.

# next steps
* try the new text edit menu
* Adopt edit menu for custom UI
* Make text ocntent searchable



* https://developer.apple.com/documentation/uikit/uifindinteraction
* https://developer.apple.com/documentation/uikit/uieditmenuinteraction
* https://developer.apple.com/documentation/uikit/app_and_environment/building_a_desktop-class_ipad_app
* https://developer.apple.com/documentation/uikit/app_and_environment/supporting_desktop-class_features_in_your_ipad_app
