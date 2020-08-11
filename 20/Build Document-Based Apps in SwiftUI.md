#swiftui 

Files and stuff
As distinct from importing documents in the app's database, working on files "in-place".

Opening documents is a feature your app can support without being document-based.  Xcode, mail.

`DocumentGroup`


```swift
@main
struct TextEdit: App {
    var body: some Scene {
        DocumentGroup(newDocument: TextDocument()) { file in
            TextEditor(text: file.$document.text)
        }
    }
}
```

You're indicating that your app supports opening/managing this type of document in-place.  

Structure of our code matches the hierarchy of ownership.  App contains a single DocumentGroup scene, which is capable of opening multiple windows of that document content.

It could support multiple DocumentGroups for different types, or a WindowGroup and a DocumentGroup.  Composeable.

SwiftUI gives your app the expected platform beahviors.  Open/save menu items, state tracking, document browser (iOS), etc.

There is a 'document app' template.  Both multiplatform and specific platforms, I guess.

Document types.  Imported vs invented ("Exported Type Identifiers") types.

We pass thet ype of document to DocumentGroup.  Binding lets SwiftUI know when the text is updated.

`UTType(importedAs:)` vs `UTType(exportedAs:)`.  The imported type is a computed variable, because it may change over the app lifecycle as other apps are installed or uninstalled.  Our type is immutable, because it comes from us

```swift
extension UTType {
	static var exampleText: UTType {
		UTType(importedAs: "com.example.plain-text")
	}
	static let shapeEditDocument = UTType(exportedAs: "com.example.ShapeEdit.shapes")
}
```

Conform to `FileDocument` protocol.