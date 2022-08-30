#swiftui engineer.

drag/drop, copy/paste, etc.

I want this application to seamlessly support drag and drop of the scientist portrait to and from the app, paste button.  Sharing on watchOS.

New design for sharesheet this year.  Under the hood, enabling all these features require the models to support being sent over to a receiver inside our app or even in other applications.  Profile structure contains all the information that we have about a single personality.  All profiles packed in an archive can be shared among friends.  We store fun facts about thep ersonality in strings and even can attach videos.

new easy way to make these model types shared.

# Transferable
Swift-first, declarative way to describe how your models can be serialized and deserialized for sharing and data trasnfer.

# Anatomy of Transferable
User passes information from one app to another via copy/paste, etc.  

When you send something between two different apps, all this binary data goes across.  An important part of sending this data is determining what it corresponds to.  It could be text, a video, etc.

How apps generate this binary data?  All types that can be shared with other apps or a single application ahve to provide two pieces of information.
1.  Ways to convert them toa nd from binary data
2. Content type that corresopnds to the structure of the binary data and tells the receiver what they actually got.

Content type => uniform type identifier.  Apple-specific technology that describes identifiers for different binary structures.  Form a tree .  We can define custom identifiers.

To declare a custom identifier, first add it to info plist.  Also a good idea to add a file extension.  If the data is saved on disk, the system will know that your app can open it.

Secondly, declare it in code.
```swift
import UniformTypeIdentifiers

// also declare the content type in the Info.plist
extension UTType {
    static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

[[Uniform Type Identifiers - A (Re)introduction]]

Good news is that many standard types already conform to Transferable.  String, data, URL, AttributedString, image, etc.  You need only a couple of lines of code to paste fun facts to ap rofile with the nwe SwiftUI paste button interface.

not realyl sure what these are about but pasting them here

```swift
import SwiftUI

struct Profile {
    private var funFacts: [String] = []
    mutating func addFunFacts(_ newFunFacts: [String]) {
        funFacts.append(newFunFacts)
    }
}

struct PasteButtonView: View {
    @State var profile = Profile()
    var body: some View {
        PasteButton(payloadType: String.self) { funFacts in
            profile.addFunFacts(funFacts)
        }
    }
}
```

```swift
import SwiftUI

struct PortraitView: View {
    @State var portrait: Image
    var body: some View {
        portrait
            .cornerRadius(8)
            .draggable(portrait)
            .dropDestination(payloadType: Image.self) { (images: [Image], _) in
                if let image = images.first {
                    portrait = image
                    return true
                }
                return false
            }
    }
}
```

```swift
import SwiftUI

struct Profile {
    var name: String
}

struct ProfileView: View {
    @State private var portrait: Image
    var model: Profile

    var body: some View {
        VStack {
            portrait
            Text(model.name)
        }
        .toolbar {
            ShareLink(item: portrait, preview: SharePreview(model.name))
        }
    }
}
```

# Conforming custom types
As I mentioned earlier there are four model types in our app.   STring already conforms to Transferable, we don't need to do anything more.

```swift
extension Profile: Transferable {
	static var transferRepresentation: some TransferRepresentation {}
}
```

three representations
* `CodableRepresentation`
* `DataRepresentation`
* `FileRepresentation`

```swift
import Foundation

struct Profile: Codable {
    var id: UUID
    var name: String
    var bio: String
    var funFacts: [String]
    var video: URL?
    var portrait: URL?
}
```

This conforms to `Codable` so we can use `Codable` representation.  An encoder to convert to binary data, decoder to convert back.  By default, it uses JSON, but you can provide your own encoder/decoder pair.

[[Data you can trust - 18]]


```swift
import CoreTransferable
import UniformTypeIdentifiers

struct Profile: Codable {
    var id: UUID
    var name: String
    var bio: String
    var funFacts: [String]
    var video: URL?
    var portrait: URL?
}

extension Profile: Codable, Transferable {
    static var transferRepresentation: some TransferRepresentation {
        CodableRepresentation(contentType: .profile)
    }
}

// also declare the content type in the Info.plist
extension UTType {
    static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

Lets look at ProfilesArchive.

```swift
import CoreTransferable
import UniformTypeIdentifiers

struct ProfilesArchive {
    init(csvData: Data) throws { }
    func convertToCSV() throws -> Data { Data() }
}

extension ProfilesArchive: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        DataRepresentation(contentType: .commaSeparatedText) { archive in
            try archive.convertToCSV()
        } importing: { data in
            try ProfilesArchive(csvData: data)
        }
    }
}
```

Archive can be converted to and from data.  This means that we can use the `DataRepresentation`.  All it takes is calling the two functions we already have.

Videos can be large.  I don't want to load it into memory.  This is where `FileRepresentation` comes in.  We'll see that `FileRepresentation` passes the provided URL to the receiver and uses it to reconstruct the `Transferable` item.
```swift
import CoreTransferable

struct Video: Transferable {
    let file: URL
    static var transferRepresentation: some TransferRepresentation {
        FileRepresentation(contentType: .mpeg4Movie) { 
            SentTransferredFile($0.file)
        } importing: { received in
            let destination = try Self.copyVideoFile(source: received.file)
            return Self.init(file: destination)
        }
    }
  
    static func copyVideoFile(source: URL) throws -> URL {
        let moviesDirectory = try FileManager.default.url(
            for: .moviesDirectory, in: .userDomainMask,
            appropriateFor: nil, create: true
        )
        var destination = moviesDirectory.appendingPathComponent(
            source.lastPathComponent, isDirectory: false)
        if FileManager.default.fileExists(atPath: destination.path) {
            let pathExtension = destination.pathExtension
            var fileName = destination.deletingPathExtension().lastPathComponent
            fileName += "_\(UUID().uuidString)"
            destination = destination
                .deletingLastPathComponent()
                .appendingPathComponent(fileName)
                .appendingPathExtension(pathExtension)
        }
        try FileManager.default.copyItem(at: source, to: destination)
        return destination
    }
}
```


Covers not only simple usecases, but also complex ones.

# Advanced tips and tricks
Whena  profile is copied to any text field, I want to paste the profile's name.  Add a second representation to allow other transferable types to present.

Notice that I added a `ProxyRepresentation`.  

```swift
import CoreTransferable
import UniformTypeIdentifiers

struct Profile: Codable {
    var id: UUID
    var name: String
    var bio: String
    var funFacts: [String]
    var video: URL?
    var portrait: URL?
}

extension Profile: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        CodableRepresentation(contentType: .profile)
        ProxyRepresentation(exporting: \.name)
    }
}

// also declare the content type in the Info.plist
extension UTType {
    static var profile: UTType = UTType(exportedAs: "com.example.profile")
}
```

Now we support both encoder/decoder conversion, and conversion to text.  Proxy only supports exporting to text, but not reconstructing the profile.  Any representation can describe both conversions, or just one.

Do you really need a file representation for the video?  We coudl have a proxy with the URL?

`FileRepresentation` works with URLs written to disk and ensure receivers' access to the file or copy by granting a temporary sandbox representation.

`ProxyRepresentation` treats you the same way as strings, no additional capabilities.  but we can ship both. URL will be treated very differently.  In the latter, payload is the URL structure itself, not the contents of the video.

```swift
import CoreTransferable

struct Video: Transferable {
    let file: URL
    static var transferRepresentation: some TransferRepresentation {
        FileRepresentation(contentType: .mpeg4Movie) { SentTransferredFile($0.file) }
           importing: { received in
               let copy = try Self.copyVideoFile(source: received.file)
               return Self.init(file: copy) }
        ProxyRepresentation(exporting: \.file)
  }

    static func copyVideoFile(source: URL) throws -> URL {
        let moviesDirectory = try FileManager.default.url(
            for: .moviesDirectory, in: .userDomainMask,
            appropriateFor: nil, create: true
        )
        var destination = moviesDirectory.appendingPathComponent(
            source.lastPathComponent, isDirectory: false)
        if FileManager.default.fileExists(atPath: destination.path) {
            let pathExtension = destination.pathExtension
            var fileName = destination.deletingPathExtension().lastPathComponent
            fileName += "_\(UUID().uuidString)"
            destination = destination
                .deletingLastPathComponent()
                .appendingPathComponent(fileName)
                .appendingPathExtension(pathExtension)
        }
        try FileManager.default.copyItem(at: source, to: destination)
        return destination
    }
}
```

There are cases when we don't support converting to CSV.  Add a boolean property telling us if we can convert or not.

`.exportingCondition`.  If our archive doesn't support, it won't be exported in this format.  You can build custom representation, just like custom views in SwiftUI.  
```swift
import CoreTransferable
import UniformTypeIdentifiers

struct ProfilesArchive {
    var supportsCSV: Bool { true }
    init(csvData: Data) throws {  }
    func convertToCSV() throws -> Data { Data() }
}

extension ProfilesArchive: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        DataRepresentation(contentType: .commaSeparatedText) { archive in
            try archive.convertToCSV()
        } importing: { data in
            try Self(csvData: data)
        }
        .exportingCondition { $0.supportsCSV }
    }
}
```

Can create your own!

```swift
struct CombinedTransferRepresentation: TransferRepresentation {
	var body: some TransferRepresentation {
		DataTransferRepresentation(...)
		FileTransferRepresentation(...)
	}
}
```

Helped me quickly build this app with all fucntionality.  I hope it is going to help yuo build a feature-rich app in less time than before.

