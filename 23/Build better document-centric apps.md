Discover how you can use the latest features in iPadOS to improve your document-based apps. We'll show you how to take advantage of UIDocument as well as existing desktop-class iPad and document-based APIs to add new features in your app. Find out how to convert data models to UIDocument, present documents with UIDocumentViewController, learn how to migrate your apps to the latest APIs, and explore best practices.

[[Meet desktop-class iPad]]
[[Build a desktop-class iPad app]]

DocumentGroup nwo supports all these features
[[Build Document-Based Apps in SwiftUI]]
[[SwiftUI on iPad Add toolbars, titles, and more]]

UIKit is opt-in.  New base class UIDocumentViewController
Works with UIDocument
navigation bar configuration
Sharing, drag/drop, undo/redo
Fully automatic renaming

# Creating a document
Subclass `UIDocument`.
Per file type
URL based
Asynchronous
Thread-safe

Two main responsibilties
* loading/saving
* accessing content
	* More specific to your app

## Loading/saving

Two convenience methods you can override.


### Loading a document - 3:54
```swift
override func load(fromContents contents: Any, ofType typeName: String?) throws {
    // Load your document from contents
    guard let data = contents as? Data,
          let text = String(data: data, encoding: .utf8) else {
        throw DocumentError.readError
    }
    self.text = text
}
```

### Saving a document - 4:08
```swift
override func contents(forType typeName: String) throws -> Any {
    // Encode your document with an instance of NSData or NSFileWrapper
    guard let data = self.text?.data(using: .utf8) else {
        throw DocumentError.writeError
    }
    return data
}
```


[[Uniform Type Identifiers - A (Re)introduction]]

Total control - override the URL methods.  Full access to URL, etc.  Great if you want to store your documents in a database or have special requirements.  Note that while save op is asynchronous, reading is expected to complete by the time the method returns.


### Manually saving and loading a document - 4:34
```swift
override func save(to url: URL,
                   for saveOperation: UIDocument.SaveOperation,
                   completionHandler: ((Bool) -> Void)? = nil) {
    self.performAsynchronousFileAccess {
        // Set up file coordination and write file to URL
   }
}

override func read(from url: URL) throws {
    // Set up file coordination and read file from URL
}
```

## access content

add properties for content.  ex


### Defining document that require saving - 5:08
```swift
class Document: UIDocument {
    var text: String? {
        didSet {
            if oldValue != nil && oldValue != text {
                self.updateChangeCount(.done)
            }
        }
    }
}
```

Calling `updateChangeCount` to mark document as dirty.  

# Presenting a document
Similar to UIDocument, UIDocumentViewController is abstract base class taht you subclass.

* open, save, and close
* Configure navigation items
* Key commands


### Updating the view hierarchy for a document - 6:30
```swift
override func documentDidOpen() {
    configureViewForCurrentDocument()
}

override func viewDidLoad() {
    super.viewDidLoad()
    configureViewForCurrentDocument()
}

func configureViewForCurrentDocument() {
    guard let document = markdownDocument,
          !document.documentState.contains(.closed)
            && isViewLoaded else { return }
    // Configure views for document
}
```

Note that there's no timing guarantee between `documentDidOpen` is called and when VC is loaded.  Move view configuration into its own method and call from both?  Check if the view is loaded and document is opened before configuring your views.



### Updating navigation items for a document - 7:17
```swift
override func navigationItemDidUpdate() {
    // Customize navigation item
}
```

we make best effort to keep changes to a minimum.  We also offer undoRedoItemGroup.  Put this group in the navbar if you want undo/redo to appear.  Make sure that your document has an undo manager assigned.

We change the hidden property of this group depending on avaiability of undo manager, and we hand enable/disable

We automatically open/close the doc, but if you need to do this outside the VC, like so


### Manually opening a document - 8:01
```swift
documentController.openDocument { success in
    if success {
        self.present(documentController, animated: true)
    }
}
```

Extras
* document optional
* Shows empty state

[[23/What's new in UIKit|What's new in UIKit]]

UIDocumentViewController can be used as your app's VC.  We put a document button in the navigation bar that opens the document picker.

Requires declaring the key UIDocumentClass for relevant filetypes, ex, set it to the subclass matching that filetype

In iOS17  we canform to UINavigationItemrEnableDelegate
and will handle underlying file changes

see 
### Renaming a UIDocument without UIDocumentViewController - 9:20
```swift
navigationItem.renameDelegate = document
```

Now you can rename from title menu.

# Migrating your app

It's easy, 3 steps:
## Update base class

First we change our UIVC base class to UIDocumentViewController instead.  Note this supplies a `document` property.  We implement that by casting our specific ivar.



## Implement new callbacks

Move navigation stuff to navigationDidUpdate.



## Delete unnecessary code
No longer need rename delegate, since that's handled automatically.

Remove more customizations.  Style, backAction are configured automatically as well.  

Focus on unique, key app elements.  All you need to know to take document-centric apps to the next level.

# Next steps
* convert your data model to use UIDocument
* convert your content VC to use UIDocumentViewController baseclass
* delete *all the code*
# Resources
* https://developer.apple.com/documentation/uikit/uidocument
* https://developer.apple.com/documentation/uikit/uidocumentviewcontroller
