Discover the latest advances in Mac app development. Get an overview of the new features in macOS Sequoia, and how to adopt them in your app. Explore new ways to integrate your existing code with SwiftUI. Learn about the improvements made to numerous AppKit controls, like toolbars, menus, text input, and more.

# New macOS features

Writing tools.  macOS can help with more sophisticated writing concepts.  Apps get these automatically.  For the best writing experience in your app, consider the interaction behaviors they bring.  If you're doing advanced things with views, see 

[[Get started with Writing Tools]]

## genmoji
small amount of adoption needed to store inline with text.

[[Bring expression to your app with Genmoji]]

## image playground


### 2:09 - Adding the Image Playground experience

```swift
extension DocumentCanvasViewController {
    @IBAction func importFromImagePlayground(_ sender: Any?) {
        // Initialize the playground, get set up to be notified of lifecycle events.
        let playground = ImagePlaygroundViewController()
        playground.delegate = self
        
        // Seed the playground with concepts and source imagery. (Optional)
        playground.concepts = [.text("birthday card")]
        playground.sourceImage = NSImage(named: "balloons")
        
        presentAsSheet(playground)
    }
}

extension DocumentCanvasViewController: ImagePlaygroundViewController.Delegate {
    func imagePlaygroundViewController(
        _ imagePlaygroundViewController: ImagePlaygroundViewController,
        didCreateImageAt resultingImageURL: URL
    ) {
        if let image = NSImage(contentsOf: resultingImageURL) {
            imageView.image = image
        } else {
            logger.error("Could not read image at \(resultingImageURL)")
        }
        
        dismiss(imagePlaygroundViewController)
    }
}
```

Consider adding image playground as another source of images.

## window tiling

Holding option will show a preview of the nearest tile.  Access window tiling options in window->move & resize menu.  See all keyboard shortcuts!

Can conveniently access in window titlebar too.  Resize two windows at the same time!  

New windows appear in untiled size.  Can select 3-window arrangement.  

Best practices
* minimum size
* resize increments
### 5:50 - Using window resize increments

```swift
window.resizeIncrements = NSSize(width: characterWidth, height: characterHeight)
```

ex single character for terminal

* cascading (`cascadingReferenceFrame`).  Cascade newly-opened windows relative to that frame.  NSWindowController does this by default.


# More SwiftUI integrations

Just like you can use swiftui views in your hosting app, you can now share menu as well.

Use from appkit with `NSHostingMenu`.  New NSMenu subclass.  


### 7:05 - Build menus with SwiftUI

```swift
struct ActionMenu: View {
    var body: some View {
        Toggle("Use Groups", isOn: $useGroups)
        
        Picker("Sort By", selection: $sortOrder) {
            ForEach(SortOrder.allCases) {
                Text($0.title)
            }
        }.pickerStyle(.inline)
        
        Button("Customize View…") {
            <#Action#>
        }
    }
}

let menu = NSHostingMenu(rootView: ActionMenu())
let pullDown = NSPopUpButton(image: image, pullDownMenu: menu)
```


Now use swiftui animation types for NSViews.

### 7:43 - Get animated with SwiftUI

```swift
NSAnimationContext.animate(with: .spring(duration: 0.3)) {
    drawer.isExpanded.toggle()
}
```


### 7:55 - Get animated with SwiftUI

```swift
class PaletteView: NSView {
    @Invalidating(.layout)
    var isExpanded: Bool = false
    
    private func onHover(_ isHovered: Bool) {
        NSAnimationContext.animate(with: .spring) {
            isExpanded = isHovered
            layoutSubtreeIfNeeded()
        }
    }
}
```

Interruptable, retargetable.  See [[Enhance your UI animations and transitions]]


# API refinements

## context menus
Use keyboard for context menu.  
Quick, accessible.
Control-return.  
Customizable shortcut.

Where does menu present from?  If your view has a value for menu property, we position over view's bounds.

If you draw a custom selection, implelment `NSViewContentSelectionInfo` protocol.  View's menu will be positioned near the selection.  

## Text and SF Symbols

Text highlighting.  Highlights can be used to emphasize text wiht a background color and contrasting foreground color.  Any NSTextview that supports rich text.  First, select a range of text.  Right click, use font menu and navigate to highlight submenu.  Choose from a number of highlight color schemes, or use accent color.  for textedit, that's blue.

You may wish to implement for custom text attribute controls.


### 10:31 - Text highlighting

```swift
let attributes: [NSAttributedString.Key: Any] = [
    .textHighlight: NSAttributedString.TextHighlightStyle.systemDefault,
    .textHighlightColorScheme: NSAttributedString.TextHighlightColorScheme.pink,
]
```


systemDefault -> range of text should be highlighted.  Color based on app's accent color.

Use `.textHighlightColorScheme` to pick a custom color.

SF Symbols 6.  800 new symbols, wide variety of subjects.  Even more effects, like wiggle, rotate, and breathe.



### 11:11 - SF Symbols effects

```swift
imageView.addSymbolEffect(.wiggle)
imageView.addSymbolEffect(.rotate)
imageView.addSymbolEffect(.breathe)
```

### 11:24 - SF Symbols playback (periodic)

```swift
imageView.addSymbolEffect(.wiggle, options: .repeat(.periodic(3, delay: 0.5)))
```

### 11:30 - SF Symbols playback (continuous)

```swift
imageView.addSymbolEffect(.wiggle, options: .repeat(.continuous))
```


### 11:37 - SF Symbols magic replace

```swift
imageView.setSymbolImage(badgedSymbolImage, contentTransition: .replace)
```

[[What’s new in SF Symbols 6]]
[[Animate symbols in your app]]



## Saving documents

Often convenient to select format of file in picker.  Now you don't need a custom accessory view just for this.  Standard file format picker is as easy as setting `showscontentTypes = true`.


### 12:19 - Save panel content types

```swift
extension ImageViewController: NSOpenSavePanelDelegate {
    @MainActor
    @IBAction internal func saveDocument(_ sender: Any?) {
        Task {
            let savePanel = NSSavePanel()
            savePanel.delegate = self
            savePanel.identifier = NSUserInterfaceItemIdentifier("ImageExport")
            savePanel.showsContentTypes = true
            savePanel.allowedContentTypes = [.png, .jpeg]
            
            let result = await savePanel.beginSheetModal(for: window)
            
            switch result {
            case .OK:
                let url = savePanel.url
                // Save the document to 'url'. It already has the appropriate extension.
            case .cancel:
                break
            default:
                break
            }
        }
    }

    func panel(_ panel: Any, displayNameFor type: UTType) -> String? {
        switch type {
        case .png:
            NSLocalizedString("PNG (Greater Quality)", comment: <#Comment#>)
        case .jpeg:
            NSLocalizedString("JPG (Smaller File Size)", comment: <#Comment#>)
        default:
            nil
        }
    }
}
```

types specified in `allowedContentTypes` property.  To provide custom display names, implement `panel(displayNameFor:)`.

## new cursors

System cursors are now available in the SDK.  

Frame-resize cursors.  used to resize an element from its edges and corners.

Position - edge or corner that we're interacting with.  directions -> can be resized.  Handle minimum or maximum size!

### 13:34 - Frame-resize cursors

```swift
let cursor = NSCursor.frameResize(position: .bottomRight, directions: .all)
```

If you're moving a separator, use column and row resize cursors.  Bar is perpendicular to arrow.  Cursors are useful when resizing the widths of table columns or heights of rows in a spreadsheet.  


### 14:20 - Column and row resize cursors

```swift
let cursor = NSCursor.columnResize(directions: .left)
let cursor = NSCursor.rowResize(directions: .up)
```

zoomIn/zoomOut cursors.  

standard apperaance
app's custom cursors aren't communicating something that the system ones don't, question whether custom ones are really needed.

### 14:29 - Zoom in and out cursors

```swift
let cursor = NSCusor.zoomIn
let cursor = NSCusor.zoomOut
```

Not only is it easier to make use of standard cursors, but they also support larger accessibility sizes.  
As well as settings for adjusting pointer colors.  Accessibility colors.
## customizable toolbars

* display modes
* `allowsDisplayModeCustomization` property.  Ensure your toolbar has an identifier so that appkit can save the style preference
* set a toolbar identifier
* double check labels

Use new `itemIdentifiers` property.  Setting a toolbar's item identifiers automatically makes minimal removals/additions for you.  Only use for dynamic non-customizable toolbar.

Just disable items that aren't applicable to current selection.  Only programmatically add/remove when there's a change to a window's mode like edit mode etc.

Conditionally show with `NSToolbarItem.isHidden`.  Still appear in customization.  Use when item isn't applicable *regardless of current selection in the window*.  ex hide downloads until first download starts.



### 15:57 - Display mode customizable toolbar

```swift
let toolbar = NSToolbar(identifier: NSToolbar.Identifier("ViewerWindow"))
toolbar.allowsDisplayModeCustomization // Defaults to `true`.
```

### 16:57 - Hidden toolbar items

```swift
let downloadsToolbarItem: NSToolbarItem
downloadsToolbarItem.isHidden = downloadsManager.downloads.isEmpty
```

text entry suggestions - custom suggestions as you type.  System standard suggestions menu.  Now standardized in appkit.

Any NSTextField, including subclasses like `NSSearchField`.  set `suggestionsDelegate`.  Provides items while typing.  Responds both synchronously and asynchronously
Highlight and selection

Design guidance:
* responsive - keep up with how fast they type
* predictable - async suggestions should come after sync ones, build muscle memory etc
* Simple.  



### 17:49 - Text entry suggestions

```swift
class MYViewController: NSViewController {
    let museumTextField = NSTextField(string: "")
    let museumTextSuggestionsController = MuseumTextSuggestionsController()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.museumTextField.suggestionsDelegate = self.museumTextSuggestionsController
    }
}

class MuseumTextSuggestionsController: NSTextSuggestionsDelegate {
    typealias SuggestionItemType = Museum
    
    func textField(
        _ textField: NSTextField,
        provideUpdatedSuggestions responseHandler: @escaping ((ItemResponse) -> Void)
    ) {
        let searchString = textField.stringValue
        
        func museumItem(_ museum: Museum) -> Item {
            var item = NSSuggestionItem(representedValue: museum, title: museum.name)
            item.secondaryTitle = museum.address
            return item
        }
        
        let favoriteMuseums = Museum.favorites.filter({ $0.matches(searchString) })
        let favorites = NSSuggestionItemSection(
            title: NSLocalizedString(
                "Favorites",
                comment: "The title of suggestion results section containing favorite museums."
            ),
            items: favoriteMuseums.map(museumItem(_:))
        )
        
        var response = NSSuggestionItemResponse(itemSections: [favorites])
        response.phase = .intermediate
        responseHandler(response)
        
        Task {
            let otherMuseums = await Museum.allMatching(searchString)
            let nonFavorites = NSSuggestionItemSection(
                items: otherMuseums.map(museumItem(_:))
            )
            
            var response = NSSuggestionItemResponse(itemSections: [
                favorites,
                nonFavorites,
            ])
            
            response.phase = .final
            responseHandler(response)
        }
    }
}
```

## text input


# Next steps
* new macOS features
* intelligence
* window tiling
* More swiftUI integrations
	* Menu, animation
* api refinemenets
	* standard components
	* context menus
	* toolbars







# Resources
* https://developer.apple.com/documentation/Updates/AppKit
