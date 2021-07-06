#swiftui #async 
Managing concurrency in your Swift code.

# Before you get started
[[Meet asyncawait in Swift]]
[[Explore Structured Concurrency in Swift]]

My app will interact with a web service using a rest api.  Perfect use of the new concurrency features.

```swift
/// A SpacePhoto contains information about a single day's photo record
/// including its date, a title, description, etc.
struct SpacePhoto {
    /// The title of the astronomical photo.
    var title: String

    /// A description of the astronomical photo.
    var description: String

    /// The date the given entry was added to the catalog.
    var date: Date

    /// A link to the image contained within the entry.
    var url: URL
}


extension SpacePhoto: Codable {
    enum CodingKeys: String, CodingKey {
        case title
        case description = "explanation"
        case date
        case url
    }

    init(data: Data) throws {
        let decoder = JSONDecoder()
        decoder.dateDecodingStrategy =
            .formatted(SpacePhoto.dateFormatter)

        self = try JSONDecoder()
            .decode(SpacePhoto.self, from: data)
    }
}

extension SpacePhoto: Identifiable {
    var id: Date { date }
}

extension SpacePhoto {
    static let urlTemplate = "https://example.com/photos
    static let dateFormat = "yyyy-MM-dd"

    static var dateFormatter: DateFormatter {
        let formatter = DateFormatter()
        formatter.dateFormat = Self.dateFormat
        return formatter
    }

    static func requestFor(date: Date) -> URL {
        let dateString = SpacePhoto.dateFormatter.string(from: date)
        return URL(string: "\(SpacePhoto.urlTemplate)&date=\(dateString)")!
    }

    private static func parseDate(
        fromContainer container: KeyedDecodingContainer<CodingKeys>
    ) throws -> Date {
        let dateString = try container.decode(String.self, forKey: .date)
        guard let result = dateFormatter.date(from: dateString) else {
            throw DecodingError.dataCorruptedError(
                forKey: .date,
                in: container,
                debugDescription: "Invalid date format")
        }
        return result
    }

    private var dateString: String {
        Self.dateFormatter.string(from: date)
    }
}

extension SpacePhoto {
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        title = try container.decode(String.self, forKey: .title)
        description = try container.decode(String.self, forKey: .description)
        date = try Self.parseDate(fromContainer: container)
        url = try container.decode(URL.self, forKey: .url)
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(title, forKey: .title)
        try container.encode(description, forKey: .description)
        try container.encode(dateString, forKey: .date)
    }
}
```

## SwiftUI run loop
[[Data Essentials in SwiftUI]]
[[Protect mutable state with Swift actors]]

SwiftUI runloop receives events, lets you update your model, and renders your views.

Ticks of the runloop.

In swiftUI, observable objects can interact with the runloop in some interesting ways.

"Slow" updates when your view does too much work in-body.  Or if your model code does too much worko n the main actor.  

Suppose you block in SwiftUI body.  You miss the next runloop tick.  Visible as a hitch.

In the past, you might have dispatched to another queue.  This might seem to work fine.  But when you change observable object from off the actor, my changes and runloop can interleave.

1.  snapshot vs unchanged value
2.  runloop tick
3.  change posted.  But swiftui never sees it

So instead, we need

1. `objectWillChange`
2. the state changes
3. The run loop ticks

If I can ensure that these happen on the main actor, I can guarantee this ordering.

By using await from the main actor, I let other work continue on the main actor, while async work happens.

*yield*ing the main actor.

Can yield back to swiftUI during I/O.  When async work completes, swift re-enters the updateItems method, back on the main actor so I can update my state.

When I write `await`, the `updateItems` function yields control of the main actor so that the runloop can continue.

Main actor re-enters the function so I can safely update my published property, triggering `objectWillChange`.

```swift
/// An observable object representing a random list of space photos.
@MainActor
class Photos: ObservableObject {
    @Published private(set) var items: [SpacePhoto] = []

    /// Updates `items` to a new, random list of `SpacePhoto`.
    func updateItems() async {
        let fetched = await fetchPhotos()
        items = fetched
    }

    /// Fetches a new, random list of `SpacePhoto`.
    func fetchPhotos() async -> [SpacePhoto] {
        var downloaded: [SpacePhoto] = []
        for date in randomPhotoDates() {
            let url = SpacePhoto.requestFor(date: date)
            if let photo = await fetchPhoto(from: url) {
                downloaded.append(photo)
            }
        }
        return downloaded
    }

    /// Fetches a `SpacePhoto` from the given `URL`.
    func fetchPhoto(from url: URL) async -> SpacePhoto? {
        do {
            let (data, _) = try await URLSession.shared.data(from: url)
            return try SpacePhoto(data: data)
        } catch {
            return nil
        }
    }
}
```

Maybe you're as nervous as I am by these forced tries.  For now, I'll return nil when the download fails.

By adding `@MainActor`, the compiler will guarantee that the properties/methods on `photos` are only accessed from the Main Actor.

Next, connect views to the models.

# Concurrent data models
I want to call updateItems whenever something shows.  Formerly `onAppear`, but now you can use `task` modifier.

task starts at beginning of the view's lifetime.  async by default.

```swift
struct CatalogView: View {
    @StateObject private var photos = Photos()

    var body: some View {
        NavigationView {
            List {
                ForEach(photos.items) { item in
                    PhotoView(photo: item)
                        .listRowSeparator(.hidden)
                }
            }
            .navigationTitle("Catalog")
            .listStyle(.plain)
            .refreshable {
                await photos.updateItems()
            }
        }
        .task {
            await photos.updateItems()
        }
    }
}
```

Task lifetime is tied to view lifetime, so you can do things like waiting on an async sequence or responding to its values.  Task will be cancelled when lifetime ends.

For more on view lifetime, see demystify swiftUI.

I'll add bg materials behind the title.  Now, let's add the iamges.  Using new async image API, loading images from a remote server is easier than ever.

`AsyncImage` which has its own placeholder view.

Min width/height to make image flexible.  Using nonzero min/height ensures things work over title area.

```swift
struct SavePhotoButton: View {
    var photo: SpacePhoto
    @State private var isSaving = false

    var body: some View {
        Button {
            async {
                isSaving = true
                await photo.save()
                isSaving = false
            }
        } label: {
            Text("Save")
                .opacity(isSaving ? 0 : 1)
                .overlay {
                    if isSaving {
                        ProgressView()
                    }
                }
        }
        .disabled(isSaving)
        .buttonStyle(.bordered)
    }
}
```

Tells swiftUI that some content is refreshable.  Async closureto update the list.
# SwiftUI and the main actor
# New concurrency tools

# Wrap up
* Just use `await`
* ObservableObject with `@MainActor` for more robust checking that your object updates in a ways that work well
* `AsyncImage`
* `refreshable`
* Use concurrency in your own views

Concurrency is tricky.

https://developer.apple.com/documentation/SwiftUI/AsyncImage