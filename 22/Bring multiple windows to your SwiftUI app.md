#swiftui 

# Scene basics
Apps in swiftui are composed of scenes and views.
scenes => represent their contents with a window onscreen.  On platforms that support multiple windows, such asi PadOS and macOS, a scene can represent itself with several windows.  Behaviors vary based on type used.  A scene may only represnet itself with a single instance regardless of platform capabilitiy.

* WindowGroup => build data-driven apps across all platforms.
* DocumentGroup => document-based apps on iOS and macOS
* Settings => in-app settings values on macOS
```swift
import SwiftUI
import UniformTypeIdentifiers

@main
struct MultiSceneApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }

        #if os(iOS) || os(macOS)
        DocumentGroup(viewing: CustomImageDocument.self) { file in
            ImageViewer(file.document)
        }
        #endif

        #if os(macOS)
        Settings {
            SettingsView()
        }
        #endif
    }
}

struct ContentView: View {
    var body: some View {
        Text("Content")
    }
}

struct ImageViewer: View {
    var document: CustomImageDocument

    init(_ document: CustomImageDocument) {
        self.document = document
    }

    var body: some View {
        Text("Image")
    }
}

struct SettingsView: View {
    var body: some View {
        Text("Settings")
    }
}

struct CustomImageDocument: FileDocument {
    var data: Data

    static var readableContentTypes: [UTType] { [UTType.image] }

    init(configuration: ReadConfiguration) throws {
        guard let data = configuration.file.regularFileContents
        else {
            throw CocoaError(.fileReadCorruptFile)
        }
        self.data = data
    }

    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        FileWrapper(regularFileWithContents: data)
    }
}
```

two new additions
* window => single window on all platforms
* MenuBarExtra => persistent control in the system menubar.

Both as a standalone scene, or compose with other scenes.

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

class ReadingListStore: ObservableObject {
}
```

Single unique window instance.  Useful when the contents of your scene represent global app state.  Window groups' multi-window presentation style.

A game may wish to only allow for a single main window to render its contents.
MenuBarExtra => new macOS-only scene type.

```swift
import SwiftUI

@main
struct UtilityApp: App {
    var body: some Scene {
        MenuBarExtra("Utility App", systemImage: "hammer") {
            AppMenu()
        }
    }
}

struct AppMenu: View {
    var body: some View {
        Text("App Menu Item")
    }
}
```

Will place its label in the menubar and show its contents in eithedr a window that is anchored to the label, or...

* persistent control in amcOS menu bar
* available as long as your pap is running
```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        #if os(macOS)
        MenuBarExtra("Book Club", systemImage: "book") {
            AppMenu()
        }
        #endif
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct AppMenu: View {
    var body: some View {
        Text("App Menu Item")
    }
}

class ReadingListStore: ObservableObject {
}
```



Supports two rendering styles.  Default style, which shows the content in a menu, which pulls down from the menu bar, as well as a style that presents its contents in a chromeless window anchored to the menubar

```swift
import SwiftUI

@main
struct UtilityApp: App {
    var body: some Scene {
        MenuBarExtra("Utility App", systemImage: "hammer") {
            AppMenu()
        }
    }
}

struct AppMenu: View {
    var body: some View {
        Text("App Menu Item")
    }
}
```

```swift
import SwiftUI

@main
struct UtilityApp: App {
    var body: some Scene {
        MenuBarExtra("Time Tracker", systemImage: "rectangle.stack.fill") {
            TimeTrackerChart()
        }
        .menuBarExtraStyle(.window)
    }
}

struct TimeTrackerChart: View {
    var body: some View {
        Text("Time Tracker Chart")
    }
}
```


# Auxiliary scenes
Book club app.

```swift
import SwiftUI

@main
struct BookClubApp: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

class ReadingListStore: ObservableObject {
}
```

On macOS, my book club app can benefit from an additional window to display reading activity over time.  HOw macOS apps can make use of additional screen relal estate and flexible windowing arrangment.s

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

class ReadingListStore: ObservableObject {
}
```

Our activity window's data is derived from overall app state, so `Window` is ideal.  Title as the label of our menu item, which is added to a section of the window menu.  When selecting, the scene's window will be opened.  Otherwise, brough tot he front.
# Scene navigation
How you can integrate them into your app to provide richer experiences.  
```swift
import SwiftUI

struct OpenBookButton: View {
    var book: Book

    var body: some View {
        Button("Open In New Window") {
        }
    }
}

struct Book: Identifiable {
    var id: UUID
}
```

* openWindow => supports both WindowGroup and Window scenes.  Identifier passed must match an identifier for a scene.  Can take a presentation value which the presented scene wil use.  Only suported by WindowGroup.
	* Type must match with an existing scene.

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
    }
}

struct OpenWindowButton: View {
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open Activity Window") {
            openWindow(id: "activity")
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

class ReadingListStore: ObservableObject {
}
```

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        WindowGroup("Book Details", for: Book.ID.self) { $bookId in
            BookDetail(id: $bookId, store: store)
        }
    }
}

struct OpenWindowButton: View {
    var book: Book
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open In New Window") {
            openWindow(value: book.id)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

struct BookDetail: View {
    @Binding var id: Book.ID?
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Book Details")
    }
}

struct Book: Identifiable {
    var id: UUID
}

class ReadingListStore: ObservableObject {
}
```


* newDocument action 
	* supports FileDocument and RferenceFileDocument
	* DocumentGroup must have an editor role
	* Document is created when the window is presented

```swift
import SwiftUI
import UniformTypeIdentifiers

@main
struct TextFileApp: App {
    var body: some Scene {
        DocumentGroup(viewing: TextFile.self) { file in
            TextEditor(text: file.$document.text)
        }
    }
}

struct NewDocumentButton: View {
    @Environment(\.newDocument) private var newDocument

    var body: some View {
        Button("Open New Document") {
            newDocument(TextFile())
        }
    }
}

struct TextFile: FileDocument {
    var text: String

    static var readableContentTypes: [UTType] { [UTType.plainText] }

    init() {
        text = ""
    }

    init(configuration: ReadConfiguration) throws {
        guard let data = configuration.file.regularFileContents,
              let string = String(data: data, encoding: .utf8)
        else {
            throw CocoaError(.fileReadCorruptFile)
        }
        text = string
    }

    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        let data = text.data(using: .utf8)!
        return FileWrapper(regularFileWithContents: data)
    }
}
```

* openDocument
	* takes a URL to an existing file
	* Requires a defined document type for reading the file's type

```swift
import SwiftUI
import UniformTypeIdentifiers

@main
struct TextFileApp: App {
    var body: some Scene {
        DocumentGroup(viewing: TextFile.self) { file in
            TextEditor(text: file.$document.text)
        }
    }
}

struct OpenDocumentButton: View {
    var documentURL: URL
    @Environment(\.openDocument) private var openDocument

    var body: some View {
        Button("Open Document") {
            Task {
                do {
                    try await openDocument(at: documentURL)
                } catch {
                    // Handle error
                }
            }
        }
    }
}

struct TextFile: FileDocument {
    var text: String

    static var readableContentTypes: [UTType] { [UTType.plainText] }

    init() {
        text = ""
    }

    init(configuration: ReadConfiguration) throws {
        guard let data = configuration.file.regularFileContents,
              let string = String(data: data, encoding: .utf8)
        else {
            throw CocoaError(.fileReadCorruptFile)
        }
        text = string
    }

    func fileWrapper(configuration: WriteConfiguration) throws -> FileWrapper {
        let data = text.data(using: .utf8)!
        return FileWrapper(regularFileWithContents: data)
    }
}
```

openWindow example

```swift
struct OpenWindowButton: View {
    var book: Book
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open In New Window") {
            openWindow(value: book.id)
        }
    }
}

struct Book: Identifiable {
    var id: UUID
}
```

## Presentation values
* prefer your model's identifier, rather than the model itself
* If we were to use it as the presented value, our new window gets a copy.  Any edits to either one will not affect the other one
* Let our model store be the source of truth.
* See docs on value types
* Value must conform to both `Hashable` and `Codable`
	* hashable => associate with an open window
	* codable => persist for state restoration
* prefer lightweight values

```swift
struct OpenWindowButton: View {
    var book: Book
    @Environment(\.openWindow) private var openWindow

    var body: some View {
        Button("Open In New Window") {
            openWindow(value: book.id)
        }
    }
}

struct Book: Identifiable {
    var id: UUID
}
```

now we need to define the window group.  Takes the data for the book ID type

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        WindowGroup("Book Details", for: Book.ID.self) { $bookId in
            BookDetail(id: $bookId, store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

struct BookDetail: View {
    @Binding var id: Book.ID?
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Book Details")
    }
}

struct Book: Identifiable {
    var id: UUID
}

class ReadingListStore: ObservableObject {
}
```

when a value is provided to the wwindowgroup, swiftUI creates a new child scene for that value, and the group content will be defined by that value, using the group's viewbuilder.
Each unique value will be a new scene.  Equality will be used inf a new window will be created or reuse existing.

Group will use a window that already exists if possible.
Selecting a context menu action for an existing open book will just bring it to the front

## State restoration
```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        WindowGroup("Book Details", for: Book.ID.self) { $bookId in
            BookDetail(id: $bookId, store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

struct BookDetail: View {
    @Binding var id: Book.ID?
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Book Details")
    }
}

struct Book: Identifiable {
    var id: UUID
}

class ReadingListStore: ObservableObject {
}
```

* viewBuilder is provided a binding to the presented value
* Binding can be modified while window is open
* Set to most recent value when scene is restored.


# Scene customizations

Becauwe we defined two scenes, SwiftUI by defualt adds a menu for each group in the file menu.  

I'd prefer that the windows can only be opened via the context window.  `.commandsRemoved` allows you to modify a scene so it no longer provides its default commands.

```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        WindowGroup("Book Details", for: Book.ID.self) { $bookId in
            BookDetail(id: $bookId, store: store)
        }
        .commandsRemoved()
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

struct BookDetail: View {
    @Binding var id: Book.ID?
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Book Details")
    }
}

struct Book: Identifiable {
    var id: UUID
}

class ReadingListStore: ObservableObject {
}
```

Since I'm going to apply a few modifiers, I'll extract ReadingActivityScene into a custom scene.



```swift
import SwiftUI

@main
struct BookClub: App {
    @StateObject private var store = ReadingListStore()

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }

      	ReadingActivityScene(store: store)
      
        WindowGroup("Book Details", for: Book.ID.self) { $bookId in
            BookDetail(id: $bookId, store: store)
        }
        .commandsRemoved()
    }
}

struct ReadingActivityScene: Scene {
    @ObservedObject var store: ReadingListStore

    var body: some Scene {
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading List")
    }
}

struct ReadingActivity: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Reading Activity")
    }
}

struct BookDetail: View {
    @Binding var id: Book.ID?
    @ObservedObject var store: ReadingListStore

    var body: some View {
        Text("Book Details")
    }
}

struct Book: Identifiable {
    var id: UUID
}

class ReadingListStore: ObservableObject {
}
```

SwiftUI by default places a window in the center of the screen.  I'd prefer if the reading activity was placed in a different location by default.  Add the `defaultPosition` modifier to specify a position to be used when no previous state is available.

Relative to the screen size, locale, etc.



```swift
struct ReadingActivityScene: Scene {
    @ObservedObject var store: ReadingListStore

    var body: some Scene {
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        .defaultPosition(.topTrailing)
    }
}

class ReadingListStore: ObservableObject {
}
```

Differentiates my activity window from other windows onscreen.

Can also specify a size


```swift
struct ReadingActivityScene: Scene {
    @ObservedObject var store: ReadingListStore

    var body: some Scene {
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        #if os(macOS)
        .defaultPosition(.topTrailing)
      	.defaultSize(width: 400, height: 800)
        #endif
    }
}

class ReadingListStore: ObservableObject {
}
```

Value given to the layuot system to derive an initial size for the window.  One more modifier.  `keyboardShortcut` has been expanded to work on scene types as well.  Affects the command which creates a new window.  Great way to customize your app by providing shortcuts to commonly-used scenes.

```swift
struct ReadingActivityScene: Scene {
    @ObservedObject var store: ReadingListStore

    var body: some Scene {
        Window("Activity", id: "activity") {
            ReadingActivity(store: store)
        }
        #if os(macOS)
        .defaultPosition(.topTrailing)
      	.defaultSize(width: 400, height: 800)
        #endif
        #if os(macOS) || os(iOS)
        .keyboardShortcut("0", modifiers: [.option, .command])
        #endif
    }
}

class ReadingListStore: ObservableObject {
}
```

This completes our tour.

# next steps

[[SwiftUI on iPad Organize your interface]]
[[SwiftUI on iPad Add toolbars, titles, and more]]
* https://developer.apple.com/forums/tags/wwdc2022-10061
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=141&tag3=457030
* https://developer.apple.com/documentation/swiftui/bringing_multiple_windows_to_your_swiftui_app
* https://developer.apple.com/documentation/SwiftUI/OpenDocumentAction
* https://developer.apple.com/documentation/SwiftUI/NewDocumentAction
* https://developer.apple.com/documentation/SwiftUI/OpenWindowAction
* https://developer.apple.com/documentation/SwiftUI/Window
* https://developer.apple.com/documentation/SwiftUI/MenuBarExtra
* https://developer.apple.com/swift/blog/?id=10
* https://developer.apple.com/documentation/SwiftUI/DocumentGroup
* https://developer.apple.com/documentation/SwiftUI/WindowGroup

