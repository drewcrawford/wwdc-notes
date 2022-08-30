# Introducing AppIntents
iOS 10 - sirikit intents.

Now we're introducing a new framework called #appintents

3 key components

* Intents
* entities
* app shortcuts

Everyone can use your app's features through siri, without setting up first.
Intents appear in spotlight when peopel search fory our app and when your app is suggested.
Focus filters letting customers customize for specific focus.  

[[Meet Focus Filters]]

Shortcuts app.  Don't need to add manually.  Valuable for customers because they can run shortcuts and take advantag of features from various places.  Homescreen, menu bar, etc.  Run automatically with automations.  Connect to the entire shortcuts ecosystem.

A shortcut can combine actions from multiple apps letting users invent new features and capabilities.

[[Design great actions for Shortcuts, Siri, and Suggestions]]

adopting is
1.  conscise
2. modern
3. easy
4. maintainable
# Intents and parameters
## Intent
single piece of app functionality
performed manually or automatically
return a result or an error

## intents include
* metadata
* parameters
* perform method

My users visit the currently reading shelf all the time, so I expose an app intent to make opening it quicker and more convenient.

```swift
struct OpenCurrentlyReading: AppIntent {
    static var title: LocalizedStringResource = "Open Currently Reading"

    @MainActor
    func perform() async throws -> some PerformResult {
        Navigator.shared.openShelf(.currentlyReading)
        return .finished
    }

    static var openAppWhenRun: Bool = true
}
```

All I need to get a basic app intent working.  

Add support for app shortcuts

```swift
public struct LibraryAppShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OpenCurrentlyReading(),
            phrases: ["Open Currently Reading"],
            systemImageName: "books.vertical.fill"
        )
    }
}
```

[[Implement App Shortcuts with app Intents]]

parameterize per shelf.  Conform our enum to AppEnum

```swift
public enum Shelf: String, AppEnum {
    case currentlyReading
    case wantToRead
    case read

    static var typeDisplayName: LocalizedStringResource = "Shelf"

    static var caseDisplayRepresentations: [Shelf: DisplayRepresentation] = [
	    //this must be written as a dictionary literal
	    //because the COMPILER will read this code at build time??
        .currentlyReading: "Currently Reading",
        .wantToRead: "Want to Read",
        .read: "Read",
    ]
}
```

In intents, we declare things with `@Parameter` property wrapper.  

```swift
struct OpenShelf: AppIntent {
    static var title: LocalizedStringResource = "Open Shelf"

    @Parameter(title: "Shelf")
    var shelf: Shelf

    @MainActor
    func perform() async throws -> some PerformResult {
        Navigator.shared.openShelf(shelf)
        return .finished
    }

    static var parameterSummary: some ParameterSummary {
        Summary("Open \(\.$shelf)")
    }

    static var openAppWhenRun: Bool = true
}
```
Support many types including numbers, strings, files, etc.  entities, enums, etc.

For best results in shortcuts, always provide a parameter summary for every intent you create.  Can define which parameters show up below the fold vs above.  

Many APIs, when, otherwise, switch, case, etc.

For intents that open something in the UI, we need to set `openAppWhenRun`.

What if I want to build an intent that opens a dynamic set?  For that we need,
# Entities, queries, and results
Use an entity instead of an enum when values are dynamic or user-defined.  To provide instances of entities, your app can implement queries, and return entities as resutls from intents.

When peopel tap on the parameter, they get a picker.  Including suggestions you provided.  And search.

First we create the entity and the corresopnding query
* identifier
* display representation
* type name
```swift
struct BookEntity: AppEntity, Identifiable {
    var id: UUID
    var title: String
  
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: LocalizedStringResource(stringLiteral: title))
    }

    static var typeDisplayName: LocalizedStringResource = "Book"
  
    static var defaultQuery = BookQuery()
}
```

Use this identifier to refer to your entity as it's passed between your apps and other parts of the system. Stable and persistent, could be saved in a shortcut, etc.

## Queries
Queries are an interface for retrieving entities from your app
All queries look up entities by ID
StringQuery looks up entities using a search string
propertyQuery looks up entities based on other criteria
Queries can provide suggested entities to pick from.

```swift
struct BookQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
        identifiers.compactMap { identifier in
            Database.shared.book(for: identifier)
        }
    }
}
```

Resolve entities tgiven an array of identifiers.  Fidning any books matching those identifiers.  I need to hook up the query to the entity by implementing the `defaultQuery` and returning the instance.

When the user picks a book its identifier is saved into the shortcut.

Now that the type conforms, I can use itas a parameter.  `perform` uses my navigator to navigate to the book.

In order to support book picker, my query needs to provide suggested results.  Need to implement one more methord returning all books.

```swift
struct OpenBook: AppIntent {
    @Parameter(title: "Book")
    var book: BookEntity

    static var title: LocalizedStringResource = "Open Book"

    static var openAppWhenRun = true

    @MainActor
    func perform() async throws -> some PerformResult {
        guard try await $book.requestConfirmation(for: book, dialog: "Are you sure you want to clear read state for \(book)?") else {
            return .finished
        }
        Navigator.shared.openBook(book)
        return .finished
    }

    static var parameterSummary: some ParameterSummary {
        Summary("Open \(\.$book)")
    }
  
    init() {}

    init(book: BookEntity) {
        self.book = book
    }
}
```

I should run this search in my app process agianst my app database directly.  Adopting `EntitySTringQuery` subprotocol lets me return results.

```swift
struct BookQuery: EntityStringQuery {
    func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
        identifiers.compactMap { identifier in
            Database.shared.book(for: identifier)
        }
    }

    func suggestedEntities() async throws -> [BookEntity] {
        Database.shared.books
    }

    func entities(matching string: String) async throws -> [BookEntity] {
        Database.shared.books.filter { book in
            book.title.lowercased().contains(string.lowercased())
        }
    }
}
```

I can return just the favorites and rely on `entities(matching` to let users search against longer list.

I can use the same entity and query to build more.  I need to build an intent to add books.

Building intents to manipulate data without UI can be powerful.

```swift
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"

    @Parameter(title: "Title")
    var title: String

    @Parameter(title: "Author Name")
    var authorName: String?

    @Parameter(title: "Recommended By")
    var recommendedBy: String?

    func perform() async throws -> some PerformResult {
        guard var book = await BooksAPI.shared.findBooks(named: title, author: authorName).first else {
            throw Error.notFound
        }
        book.recommendedBy = recommendedBy
        Database.shared.add(book: book)

        return .finished(
            value: book,
            showResultIntent: OpenBook(book: book)
        )
    }

    enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
        case notFound

        var localizedStringResource: LocalizedStringResource {
            switch self {
                case .notFound: return "Book Not Found"
            }
        }
    }
}
```

Includes an optional parameter recommended.

Localize the error by conforming to `CustomLocalizedStringResourceConvertible`.  

Combine with other intents.  With a bit of work, I can combine the add book with open book, pasisng the result from one to the other.  To do so, I'll have the add book return a value as part of its result.  Connect the result of this value to other intents.

Open intent.  If I add an open intent, they get a new switch called "Open When Run".  Off, part of a shortcut in the background.  if they leave the switch on, the newly added book will be opened.  To do this, we adopt `OpensIntent` protocol from our return value.

# Properties, finding, and filtering
Allow customers to find and filter.  So far, I've added the basic requirements to my book entity.  To let people integrate books more deepl,y I need to expose more.

Entities support properties which hold additional info on the entity.  In this case, I'll add the author, publishing date, etc.
```swift
struct BookEntity: AppEntity, Identifiable {
    var id: UUID

    @Property(title: "Title")
    var title: String

    @Property(title: "Publishing Date")
    var datePublished: Date

    @Property(title: "Read Date")
    var dateRead: Date?

    var recommendedBy: String?

    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: LocalizedStringResource(stringLiteral: title))
    }

    static var typeDisplayName: LocalizedStringResource = "Book"

    static var defaultQuery = BookQuery()

    init(id: UUID) {
        self.id = id
    }

    init(id: UUID, title: String) {
        self.id = id
        self.title = title
    }
}
```

All the same types as `@Parameter`, and take the localized title.  Cutsomers can now use magic variables to pull out each new piece of information with a book entity.

Now customers can find/filter.  Using the sort by and limit options, can support even more advanced queries.  Customer can use these building blocks to find most common authors, etc.  To enable, adopt another kind of query called a property query.  Find entities based on properties.

## Implementing property queries
* declare query properties
* Declare sorting options
* Impelment `entities(matching:)`.

contains, equalTo, less than, etc.  Here, I liss "less than" aand "greater than" comparators.  "contains" and "equal to" for my title property.  Query properties map each combination fo property and comparator into a type of your choice called the comparator mapping type.  Here, I'll use `NSPredicate`.  Custom database, rest api, I could design my own comparator type.

```swift
struct BookQuery: EntityPropertyQuery {
    static var sortingOptions = SortingOptions {
        SortableBy(\BookEntity.$title)
        SortableBy(\BookEntity.$dateRead)
        SortableBy(\BookEntity.$datePublished)
    }

    static var properties = EntityQueryProperties {
        Property(keyPath: \BookEntity.title) {
            EqualToComparator { NSPredicate(format: "title = %@", $0) }
            ContainsComparator { NSPredicate(format: "title CONTAINS %@", $0) }
        }
        Property(keyPath: \BookEntity.datePublished) {
            LessThanComparator { NSPredicate(format: "datePublished < %@", $0 as NSDate) }
            GreaterThanComparator { NSPredicate(format: "datePublished > %@", $0 as NSDate) }
        }
        Property(keyPath: \BookEntity.dateRead) {
            LessThanComparator { NSPredicate(format: "dateRead < %@", $0 as NSDate) }
            GreaterThanComparator { NSPredicate(format: "dateRead > %@", $0 as NSDate) }
        }
    }

    func entities(for identifiers: [UUID]) async throws -> [BookEntity] {
        identifiers.compactMap { identifier in
            Database.shared.book(for: identifier)
        }
    }

    func suggestedEntities() async throws -> [BookEntity] {
        Model.shared.library.books.map { BookEntity(id: $0.id, title: $0.title) }
    }

    func entities(matching string: String) async throws -> [BookEntity] {
        Database.shared.books.filter { book in
            book.title.lowercased().contains(string.lowercased())
        }
    }

    func entities(
        matching comparators: [NSPredicate],
        mode: ComparatorMode,
        sortedBy: [Sort<BookEntity>],
        limit: Int?
    ) async throws -> [BookEntity] {
        Database.shared.findBooks(matching: comparators, matchAll: mode == .and, sorts: sortedBy.map { (keyPath: $0.by, ascending: $0.order == .ascending) })
    }
}
```

Similar definition for sorting.  Provides all properties that I sort books by.  Title, date read, etc.  Finally, I impelment entities(matching).  Takes an array of comparator mapping type I mentioned previously.  

Use these parameters to perform a query.  

examples
* pick a random book
* find old books
* make my app more useful by connecting it to others.
* export all books to a CSV file
* graphing app to make a chart of books
* just the beginning!



# User interactions
## Dialog
Spoken or textual response to the person.  Provide dialog for intents to work well in a voice experience.

`needsVAlueDialog`

## Snippets
Visual equaiivalent of diaglog.  ADd a visual representation.  Add swiftUI as a trailing closure.
```swift
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"

    @Parameter(title: "Title")
    var title: String

    @Parameter(title: "Author Name")
    var authorName: String?

    @Parameter(title: "Recommended By")
    var recommendedBy: String?

    func perform() async throws -> some PerformResult {
        guard var book = await BooksAPI.shared.findBooks(named: title, author: authorName).first else {
            throw Error.notFound
        }
        book.recommendedBy = recommendedBy
        Database.shared.add(book: book)

        return .finished(value: book) {
            CoverView(book: book)
        }
    }

    enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
        case notFound

        var localizedStringResource: LocalizedStringResource {
            switch self {
                case .notFound: return "Book Not Found"
            }
        }
    }
}
```

Archived and sent to shortcuts or siri.


## Request value
```swift
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"

    @Parameter(title: "Title")
    var title: String

    @Parameter(title: "Author Name")
    var authorName: String?

    @Parameter(title: "Recommended By")
    var recommendedBy: String?

    func perform() async throws -> some PerformResult {
        let books = await BooksAPI.shared.findBooks(named: title, author: authorName)
        guard !books.isEmpty else {
            throw Error.notFound
        }
        if books.count > 1 && authorName == nil {
            throw $authorName.requestValue("Who wrote the book?")
        }

        return .finished
    }

    enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
        case notFound

        var localizedStringResource: LocalizedStringResource {
            switch self {
                case .notFound: return "Book Not Found"
            }
        }
    }
}
```


## Disambiguation
choose between a set of values

```swift
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"

    @Parameter(title: "Title")
    var title: String

    @Parameter(title: "Author Name")
    var authorName: String?

    @Parameter(title: "Recommended By")
    var recommendedBy: String?

    func perform() async throws -> some PerformResult {
        let books = await BooksAPI.shared.findBooks(named: title, author: authorName)
        guard !books.isEmpty else {
            throw Error.notFound
        }
        if books.count > 1 {
            let chosenAuthor = try await $authorName.requestDisambiguation(among: books.map { $0.authorName }, dialog: "Which author?")
        }
        return .finished
    }

    enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
        case notFound

        var localizedStringResource: LocalizedStringResource {
            switch self {
                case .notFound: return "Book Not Found"
            }
        }
    }
}
```


## Confirmation
parameter value.  Assume the user meant the popular book.  To do that, I'll call `requestConfirmation` on the title parameter.  
```swift
struct AddBook: AppIntent {
    static var title: LocalizedStringResource = "Add Book"

    @Parameter(title: "Title")
    var title: String

    @Parameter(title: "Author Name")
    var authorName: String?

    @Parameter(title: "Recommended By")
    var recommendedBy: String?

    func perform() async throws -> some PerformResult {
        guard var book = await BooksAPI.shared.findBooks(named: title, author: authorName).first else {
            throw Error.notFound
        }
        let confirmed = try await $title.requestConfirmation(for: book.title, dialog: "Did you mean \(book)?")
        book.recommendedBy = recommendedBy
        Database.shared.add(book: book)
        return .finished(value: book)
    }

    enum Error: Swift.Error, CustomLocalizedStringResourceConvertible {
        case notFound

        var localizedStringResource: LocalizedStringResource {
            switch self {
                case .notFound: return "Book Not Found"
            }
        }
    }
}
```
Second kind is a confirmation of the reuslt of an intent.  Great for placing orders for example.
```swift
struct BuyBook: AppIntent {
    @Parameter(title: "Book")
    var book: BookEntity

    @Parameter(title: "Count")
    var count: Int

    static var title: LocalizedStringResource = "Buy Book"

    func perform() async throws -> some IntentPerformResult {
        let order = OrderEntity(book: book, count: count)
        try await requestConfirmation(output: .finished(value: order, dialog: "Are you ready to order?") {
            OrderPreview(order: order)
        })

        return .finished(value: order, dialog: "Thank you for your order!") {
            OrderConfirmation(order: order)
        }
    }
}
```

# Architecture and lifecycle
Actually there are two ways.
* app
* extension

Directly in your app => simplest.  
* no need for a framework/duplicated code
* No cross-process coordination
* higher memory limits
* ability to start audio session
* Intents run in the foreground if you set `openAppWhenRun`.
* Implement multi-scene supprot for best performance in the background => we use a special mode.  We strongly encourage you to also implement scene support.

Intents extension
* lighter-weight
* best performance
* Focus filter intents run immediately when focus changes

To create an intents extension, file=>new target.  

## Static extraction
**Your code is the only source of truth.**
Xcode extracts app intents at build-time.
Stored in a metadata file within your app/extension
Compile AppIntents code directly into app/extension, not a framework
Use same bundle for localized strings

## Upgrading from SiriKit
* widget or Siri Domain adopters: keep using SiriKit intents framework
* Adopters of Custom Intents for Siri and Shortcuts: upgrade to app intents
* To upgrade, use "Convert to app intent" button in intent definition file

# Wrap up
* adopting app intents opens up new use cases aand value from your app








* https://developer.apple.com/forums/tags/wwdc2022-10032
* https://developer.apple.com/forums/create/question?&tag1=392030&tag2=481030
* https://developer.apple.com/documentation/AppIntents
