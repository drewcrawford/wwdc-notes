#swiftui 

* Views (last year)
* Scenes (new) 
* apps (new)

# Views, scenes, and apps
Everything is a view.
Not all views belong to the same app.  Since apps do not have complete control over the screen.
Instead, the platform controls how apps are presented.

We refer to these regions as *scenes*.  

Some platforms can show multiple windows sxs (ipados).  Other platforms prefer to show only a full-screen window.

In macOS you can gather windows (scenes) into tabs.  Also the window is associated with its own scene.

apps->scenes->views hierarchy.  
Views form the content of scenes, allowing them to be independently displayed by the platform.
All these scenes form app.

"fits into a handful of lines of code" 🤣


```swift
//new attribute in Swift 5.3 that allows a type
//to serve as the entrypoint for our program's execution.
@main
struct BookClubApp: App {
	//note that both views and apps are able to declare data dependencies.
	//StateObject is new this year.
    @StateObject private var store = ReadingListStore()
	
	//app returns Scene instead of body (View situation)
    var body: some Scene {
		//contained within a scene, called WindowGroup.
		//manages the window we will render into.  It can also
		//create additional windows, or new tabs within a window, if
		//supported on the platform.
        WindowGroup {
			//interface of our app
            ReadingListViewer(store: store)
        }
    }
}

//allows me to browse my reading list
struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        NavigationView {
            List(store.books) { book in
                Text(book.title)
            }
            .navigationTitle("Currently Reading")
        }
    }
}

class ReadingListStore: ObservableObject {
    init() {}

    var books = [
        Book(title: "Book #1", author: "Author #1"),
        Book(title: "Book #2", author: "Author #2"),
        Book(title: "Book #3", author: "Author #3")
    ]
}

struct Book: Identifiable {
    let id = UUID()
    let title: String
    let author: String
}
```
# Understanding scenes
"App expose" allows me to open multiple windows on #ipados 
State of the views in those scenes will be independent.

Navigation title -> some kind of modifier that applies to parent scene / something?

On the mac, it's very common for apps to support multiple windows.  Using `WindowGroup`, swiftUI will provide a menu item for new windows.

MacOS also supports grouping windows together.  Via "window" menu, can merge my open windows.

On macos, when theplatform needs to create a window for your app, WindowGroup will make a new child scene.  By default, this will be inside a window.

On macOS/iPadOS, WindowGroup can instantiate multiple children.  This can happen in response to user actions.  While each scene shares the definition of its user interface, the views all have their own independent state.  This means that changing the view state in one window, does not affect the state of the others.



```swift
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
        NavigationView {
            List(store.books) { book in
                Text(book.title)
            }
            .navigationTitle("Currently Reading")
        }
    }
}

class ReadingListStore: ObservableObject {
    init() {}

    var books = [
        Book(title: "Book #1", author: "Author #1"),
        Book(title: "Book #2", author: "Author #2"),
        Book(title: "Book #3", author: "Author #3")
    ]
}

struct Book: Identifiable {
    let id = UUID()
    let title: String
    let author: String
}
```

Since the platform is in charge of the lifecycle of scenes, we introduced a new proeprty wrapper, `SceneStorage` which can persist your view state.

```swift
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
	//this view state will be automatically restored at the right time
    @SceneStorage("selectedItem") private var selectedItem: String?
    
    var selectedID: Binding<UUID?> {
        Binding<UUID?>(
            get: { selectedItem.flatMap { UUID(uuidString: $0) } },
            set: { selectedItem = $0?.uuidString }
        )
    }

    var body: some View {
        NavigationView {
            List(store.books) { book in
                NavigationLink(
                    destination: Text(book.title),
                    tag: book.id,
                    selection: selectedID
                ) {
                    Text(book.title)
                }
            }
            .navigationTitle("Currently Reading")
        }
    }
}

class ReadingListStore: ObservableObject {
    init() {}

    var books = [
        Book(title: "Book #1", author: "Author #1"),
        Book(title: "Book #2", author: "Author #2"),
        Book(title: "Book #3", author: "Author #3")
    ]
}

struct Book: Identifiable {
    let id = UUID()
    let title: String
    let author: String
}
```
# Customizing apps
## `DocumentGroup`
New this year is the `DocumentGroup` scene type, which automatically manages opening, editing, saving document-based schemes

[[build document-based apps in swiftui]]

```swift
import SwiftUI
import UniformTypeIdentifiers

@main
struct ShapeEditApp: App {
    var body: some Scene {
        DocumentGroup(newDocument: ShapeDocument()) { file in
            DocumentView(document: file.$document)
        }
    }
}

struct DocumentView: View {
    @Binding var document: ShapeDocument
    
    var body: some View {
        Text(document.title)
            .frame(width: 300, height: 200)
    }
}

struct ShapeDocument: Codable {
    var title: String = "Untitled"
}

extension UTType {
    static let shapeEditDocument =
        UTType(exportedAs: "com.example.ShapeEdit.shapes")
}

extension ShapeDocument: FileDocument {
    static var readableContentTypes: [UTType] { [.shapeEditDocument] }
    
    init(fileWrapper: FileWrapper, contentType: UTType) throws {
        let data = fileWrapper.regularFileContents!
        self = try JSONDecoder().decode(Self.self, from: data)
    }

    func write(to fileWrapper: inout FileWrapper, contentType: UTType) throws {
        let data = try JSONEncoder().encode(self)
        fileWrapper = FileWrapper(regularFileWithContents: data)
    }
}
```

## Preferences
```swift
@main
struct BookClubApp: App {
    @StateObject private var store = ReadingListStore()

    @SceneBuilder var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
        
    #if os(macOS)
		//this year we're exposing this type, on macOS
		//which exposes the preferences window.
		//sets up Preferences in app window.
        Settings {
            BookClubSettingsView()
        }
    #endif
    }
}

struct BookClubSettingsView: View {
    var body: some View {
        Text("Add your settings UI here.")
            .padding()
    }
}

struct ReadingListViewer: View {
    @ObservedObject var store: ReadingListStore

    var body: some View {
        NavigationView {
            List(store.books) { book in
                Text(book.title)
            }
            .navigationTitle("Currently Reading")
        }
    }
}

class ReadingListStore: ObservableObject {
    init() {}

    var books = [
        Book2(title: "Book #1", author: "Author #1"),
        Book2(title: "Book #2", author: "Author #2"),
        Book2(title: "Book #3", author: "Author #3")
    ]
}

struct Book: Identifiable {
    let id = UUID()
    let title: String
    let author: String
}
```

## commands
```swift
//encapsulate a command in a custom type
struct BookCommands: Commands {
	//target actions based on a focus.  Like responder chain
    @FocusedBinding(\.selectedBook) private var selectedBook: Book?
    
    var body: some Commands {
        CommandMenu("Book") {
            Section {
                Button("Update Progress...", action: updateProgress)
                    .keyboardShortcut("u")
                Button("Mark Completed", action: markCompleted)
                    .keyboardShortcut("C")
            }
            .disabled(selectedBook == nil)
        }
    }
    
    private func updateProgress() {
        selectedBook?.updateProgress()
    }
    private func markCompleted() {
        selectedBook?.markCompleted()
    }
}
```
See our reference documentation?

# Next steps
* Learn about `@StateObject` and other data flow techniques
* Take advantage of new Swift syntax features

[[Data Essentials in SwiftUI]]
[[20/what's new in swift]]

