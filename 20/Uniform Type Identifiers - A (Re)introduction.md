2004 was a long time ago.
# Identifying a file's type
"regular file".  How does the system know a file's type?
Reading the bytes is extremely expensive, and requires read permissions most processes do not have.

In fact, the OS bases most of its decisions on the path extension.  We typically don't show them but they're still there.

Note that `.jpg` and `.jpe` are both alternatives.
On the web, we use MIME types instead.  For jpeg, that's `image/jpeg`.  But some people use `image/jpg` which is nonstandard  but commonly used.

On apple platforms, we use UTI to canonically identify the file format.  For jpeg, it's `public.jpeg`.  This refers to all jpeg images whether local or web.

Filetypes exist in a hierarchy.  `jpeg` is also an "image".  jpeg conforms.
Types "conform to " other types
Types support multiple inheritance
Types are like protocols

public.jepg -> public.image -> public.data ("any regular file")

... -> public.item (any FS object)

`public.content` -> tells us that we should treat subtypes as things that are important to the user.

In addition to files, used for pasteboard content.
Hardware models.  All mac models perform to `com.apple.mac`.



# Telling the system about your types
## System declared types
CoreTypes declare them already.  Just reference them.

## Generating a uniform type identifier
Reverse-DNS, case-insensitive ASCII strings.
Meaningful identifiers help.
Some namespaces are reserved

* `public.*` is reserved by apple for standardized types.
* `dyn.*` is reserved on-the-fly and system use.  Used for compatibility shims.  Fairly rare.
* `com.example.*` Examples and sample code
* `com.apple.*` apple types

## How to add a type
1.  Declare the type.  Type exists, but you can't open it.
2.  Support the type.  This lets you open the type.

## Importing and exporting types
Importing -> When you don't own the type and the system doesn't declare it.  The other app provides more authoritative information.
Export -> You invented the type and control its declaration

Don't import or export something in CoreTypes.

## Demo

We conform to `public.data`, `public.content` and `public.json`

The last one, we might omit if we want the json part to be an implementation detail of our type that we dont' want developers to rely on.  However it might be convenient if an important part of our file is to be edited with a json editor.

Choose long extensions to avoid conflicts.

[[Build Document-Based Apps in SwiftUI]]

For imported types, system prefers its declaration as more authoritative.

Set our handle rank to "Alternate".  On macOS, we specify role of "Viewer".

# Working with types in your code
```swift
for fileURL in /* */ {
    let resourceValues = try fileURL.resourceValues(forKeys: [.typeIdentifierKey])
	if let type = resourceValues.typeIdentifier {
		let description = UTTypeCopyDescription(type as CFString)?.takeUnretainedValue()
		print("\(fileURL)'s type: \(description)")
		
		if UTTypeConformsTo(type as CFString, kUTTypeImage) {
		    drawPicture(at: fileURL)
		}
		else if UTTypeConformsTo(type as CFString, kUTTypeAudio) {
		    playSong(at: fileURL)
		}
	}
}
```
Problems
* CFString is ugly

in Big Sur, we have `UniformTypeIdentifiers` framework that brings swiftyness to type identifiers.  Now we can use `UTType`.

```swift
import UniformTypeIdentifier
for fileURL in /* */ {
    let resourceValues = try fileURL.resourceValues(forKeys: [.contentTypeKey])
	if let type = resourceValues.contentType {
		let description = type.localizedDescription
		print("\(fileURL)'s type: \(description)")
		
		if type.conforms(to: .image) {
		    drawPicture(at: fileURL)
		}
		else if type.conforms(to: .audio) {
		    playSong(at: fileURL)
		}
	}
}
```


```swift
// Adding your own constants
extension UTType {
    ///The type of my file mformat
	///This type is exported.  We assert that you own this type
	public static let myFileFormat = UTType(exportedAs: "com.example.myfileformat")
	
	///The type of my competitor's format.  This type is imported
	public static var competitorFileFormat: UTType {
	    UTType(importedAs: "com.competitor.fileformat")
	}
}
```
Rather than declaring your imported type as constants, declare them as computed variables, so you pick up updated types automatically if an app is installed while you're open.

We have bugchecker for type issues if you don't declare them in your plist.

Foundation now recommends filenames to use based on UTType.
SwiftUI has pasteboard support for UTType as well.

```swift
struct MyGreatView: View {
   @State var text: String?  nil
   
   var body: some View {
      MyGreatView(content: $text)
	  .onDrop(of: [.text], isTargeted: nil) {providers in
	  	_ = providers.first?.loadObject(ofClass: String.self) { string, error in
		   text = string
		}
		return true
	  }
   }
}
```

app lifecycle

```swift
extension UTType {
   static let myGreatType = UTType(exportedAs: /* ... */)
   
}
struct MyGreatDocument: FileDocument {
    static var readableContentTypes: [UTType] = [.myGreatType]
	
	init(configuration: ReadConfiguration) throws { /* ... */ }
	func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper { /* ... */ }
}
```

[[Build Document-Based Apps in SwiftUI]]


AppKit will let you get an icon for types.
Create attribute set
specify open/saved panel types

There are a number of apps which don't support new API yet.  Get `type.identifier as CFString`.
To go back, `UTType(identifier as String)`.

# Wrap up
Uniform type identifiers are Apple's canonical 'file type' mecanism
Extend the type hierarchy with your own types
Improve your app's performance using the new UniformTypeIdentifiers framework
