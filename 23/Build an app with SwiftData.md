#swiftdata
Discover how SwiftData can help you persist data in your app. Code along with us as we bring SwiftData to a multi-platform SwiftUI app. Learn how to convert existing model classes into SwiftData models, set up the environment, reflect model layer changes in UI, and build document-based applications backed by SwiftData storage. To get the most out of this session, you should be familiar SwiftData. For an introduction, check out "Meet SwiftData" from WWDC23.

Recently we've introduced SwiftData, a new persistent model layer in Swift.

[[Meet SwiftData]]
code-along:
https://developer.apple.com/wwcdc23/10154

# meet the app
# SwiftData models
## Defining a SwiftData model - 3:33
```swift
@Model
final class Card {
    var front: String
    var back: String
    var creationDate: Date

    init(front: String, back: String, creationDate: Date = .now) {
        self.front = front
        self.back = back
        self.creationDate = creationDate
    }
}
```
## Binding to a SwiftData model - 4:25
```swift
@Bindable var card: Card
```


`@Observable` set up data flow with less code, pair with `@Bindable`.
Automatic dependencies
Seamlessly bind models' mutable state to UI.
[[Discover observation in SwiftUI]]

# Querying models to display in UI
## Query models from SwiftData storage - 5:52
```swift
@Query private var cards: [Card]
```

Use `@Query` to display models managed by swiftdata in the UI.  Property wrapper
* provides the view with data
* triggers view update on every change of the models
* A view can have multiple `@Query` properties
* Lightweight syntax to support sorting, ordering, filtering, and even... changes
* Uses `ModelContext` as the source of data
* We will get that from `.modelContainer`.  This sets up the model container.
* Creates the storage stack including the context that Query will use
* A view has a single model container
* an application can create and use as many containers as it wants for different view hierarchies.

## Setting up a model container for the window group - 8:27
```swift
WindowGroup {
    ContentView()
}
.modelContainer(for: Card.self)
```

Sample data.

## Providing a preview with sample data - 9:24
```swift
#Preview {
    ContentView()
        .frame(minWidth: 500, minHeight: 500)
        .modelContainer(previewContainer)
}
```


# Creating and updating
## Accessing the model context of the ContentView - 10:30
```swift
@Environment(\.modelContext) private var modelContext
```

* provides access to the model context
* a view has a single model context

## Insert a new model in the context - 10:51
```swift
let newCard = Card(front: "Sample Front", back: "Sample Back")
modelContext.insert(object: newCard)
```

You don't need to save the context after inserting the model.  Autosaves the model context.  The autosaves are triggered by UI-related events and user input.  We dont' need to worry about saving, because swiftdataa does it for us.  Only a few cases where we need to ensure all changes are persisted immediately.

before sharing data storage or 'senidng it over'.  In these cases, save explicitly.

demo

# \[bonus] document-based apps 

Describes certain types of applications that allow users to create, open, view, edit different types of documents.  Every document is a file, and users can store, open, share them.

## Start document-based application setup - 13:34
```swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        #if os(iOS) || os(macOS)
        DocumentGroup(editing: Card.self, contentType: <#UTType#>) {
            <#code#>
        }
        #else
        WindowGroup {
            ContentView()
                .modelContainer(for: Card.self)
        }
        #endif
    }
}
```

## contentType
Swift data document-based apps need to declare custom content types.  Each swiftdata document is built from a unique models.

swift data documents are packages.  If you mark some properties with exteernal storage attribute, all external items will be part of the package.

declare content type in plist.  "Exported type identifiers"
* extension
* description
* identifier: same as `UTType(exportedAs:)`
* type conforms to `com.apple.package` because this is a package type

Now in `DocumentGroup(editing:,contenttype:)` we can use `.flashCards`

Note that `DocumentGroup` sets up model container infrastructure on a per-document basis, we don't have to set this up ourselves.

## Finish document-based application setup - 16:51
```swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        #if os(iOS) || os(macOS)
        DocumentGroup(editing: Card.self, contentType: .flashCards) {
            ContentView()
        }
        #else
        WindowGroup {
            ContentView()
                .modelContainer(for: Card.self)
        }
        #endif
    }
}
```


# Resources
https://developer.apple.com/documentation/SwiftUI/Building-a-document-based-app-using-SwiftData