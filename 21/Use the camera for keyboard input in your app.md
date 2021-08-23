Capture text from the world around you.

# Filtering content
* Filtering content is based on UITextContentType and UIKeyboardType
* Both are exposed properties in UITextField and UITextView

[[AutoFill Everywhere]]
[[Automatic STrong Passwords and Security Code AutoFill - 18]]
[[The keys to a better input experience - 17]]

Various values.  Camera won't filter for all of them.  Just

* Full street address
* telephone number
* email address
* URL
* shipment tracking number
* flight number
* date time and duration

Simple to use.  In IB, look for "Content Type" and "Keyboard Type".  Or in code.

```swift
phone.keyboardType = .phonePad
phone.autocorrectionType = .no

address.textContentType = .fullStreetAddress
```

# How do we make this more discoverable?

ex, notes field.  Not obvious that I can use the camera for input.  

If you want an onscreen button, add your own dedicated button.

```swift
let textFromCamera = UIAction.captureTextFromCamera(responder: self.notes, identifier: nil)
```

This knows how to launch the camera, image and label.  

Just requires a responder to accept the text.  Multi-menu example with various actions we're not going to talk about

```swift
let textFromCamera = UIAction.captureTextFromCamera(responder: self.notes, identifier: nil)

let choosePhotoOrVideo = UIAction(…)
let takePhotoOrVideo = UIAction(…)
let scanDocuments = UIAction(…)

let cameraMenu = UIMenu(children: [choosePhotoOrVideo, takePhotoOrVideo, scanDocuments, textFromCamera])

let menuToolbarItem = UIBarButtonItem(title: nil, image: UIImage(systemName: "camera.badge.ellipsis"), primaryAction: nil, menu: cameraMenu)
```

New action called `captureTextFromCamera`, similar to standard actions like cut/copy/paste.  

# Requirements
* 2018 or newer iPhone with Neural Engine
* Responder must handle text insertion
* UITextView and UITextfield responders must be editable
* Specific languages

# Protocols at play
Adopt *UIKeyInput*.  Basic methods to accept keyboard input.

We call `insertText`.  In reality, they adopt a protocol called `UITextInput`, which is an extension of `UIKeyInput`.  This lets you see a preview.  If you don't want it, you only need `UIKeyInput`.

`UITextInputTraits` is a base prootcol with only optional properties.  

For camera input, adopt `UIKeyInput` and `UITextInputTraits` or `UITextInput`.

# Image

```swift
class HeadlineImageView: UIImageView, UIKeyInput {
    var headlineLabel: UILabel = UILabel()
    var hasText: Bool = false

    override init(image: UIImage?) {
        super.init(image: image)
        initializeLabel()
    }
    
    func insertText(_ text: String) {
        headlineLabel.text = text
    }

    func deleteBackward() { }
}
```

To adopt UIKeyInput, we need 3 methods.  `hasText`, `deleteBackward`, and `insertText`.

`hasText` can return false, `deleteBackward()` doesnt need to do anything. 

# Wrap up
* Take advantage of text content types
* Create your own camera launchers
* Check for support
* Not just for text fields or views




