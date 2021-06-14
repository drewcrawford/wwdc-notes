#swiftui 

# SwiftUI

1.  Declarative UI
2.  100% SwiftUI apps
3.  Deeper adoption

If you haven't yet had a chance, that's ok.  Here are a few tips to keep in mind.

* Create new features in an existing app.  
* Avatar picker in macOS
* You can mix in swiftUI alongside existing UIKit or appkit code.
* Useful tool for expanding your app to new platforms.
* Used to build shortcuts on macos.
* Share common code between platforms

Apple pay purchase flow was redesigned with swiftUI
And help viewer on macOS
tips app on watchOS
Weather!

Several examples of how swiftUI is helping build the next genertion of apps.

We'd like to share great new APIs that made it possible

# Better lists
Critical features for organizing and displaying data within apps.

Making it easy to write rich, interactive lists and grids.

SwiftUI allows images async!

```swift
struct ContentView: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      ScrollView {
        LazyVGrid(columns: [GridItem(.adaptive(minimum: 420))]) {
          ForEach(photoStore.photos) { photo in
            AsyncImage(url: photo.url)
              .frame(width: 400, height: 266)
              .mask(RoundedRectangle(cornerRadius: 16))
          }
        }
        .padding()
      }
      .navigationTitle("Superhero Recruits")
    }
    .navigationViewStyle(.stack)
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}
```

Give it a URL and SwiftUI automatically fetches and displays the remote image, even with a default placeholder.

```swift
```swift
struct ContentView: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      ScrollView {
        LazyVGrid(columns: [GridItem(.adaptive(minimum: 420))]) {
          ForEach(photoStore.photos) { photo in
            AsyncImage(url: photo.url) { image in
              image
                .resizable()
                .aspectRatio(contentMode: .fill)
            } placeholder: {
              randomPlaceholderColor()
                .opacity(0.2)
            }
            .frame(width: 400, height: 266)
            .mask(RoundedRectangle(cornerRadius: 16))
          }
        }
        .padding()
      }
      .navigationTitle("Superhero Recruits")
    }
    .navigationViewStyle(.stack)
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}

func randomPlaceholderColor() -> Color {
  placeholderColors.randomElement()!
}

let placeholderColors: [Color] = [
  .red, .blue, .orange, .mint, .purple, .yellow, .green, .pink
]
```


Also customized, adding modifiers to loaded image.  Custom placeholders.

Even add custom animations and error handling!

```swift
struct Contentiew: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      ScrollView {
        LazyVGrid(columns: [GridItem(.adaptive(minimum: 420))]) {
          ForEach(photoStore.photos) { photo in
            AsyncImage(url: photo.url, transaction: .init(animation: .spring())) { phase in
              switch phase {
              case .empty:
                randomPlaceholderColor()
                  .opacity(0.2)
                  .transition(.opacity.combined(with: .scale))
              case .success(let image):
                image
                  .resizable()
                  .aspectRatio(contentMode: .fill)
                  .transition(.opacity.combined(with: .scale))
              case .failure(let error):
                ErrorView(error)
              @unknown default:
                ErrorView()
              }
            }
            .frame(width: 400, height: 266)
            .mask(RoundedRectangle(cornerRadius: 16))
          }
        }
        .padding()
      }
      .navigationTitle("Superhero Recruits")
    }
    .navigationViewStyle(.stack)
  }
}

struct ErrorView: View {
  var error: Error?

  init(_ error: Error? = nil) {
    self.error = error
  }

  var body: some View {
    Text("Error") // Display the error
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}

func randomPlaceholderColor() -> Color {
  placeholderColors.randomElement()!
}

let placeholderColors: [Color] = [
  .red, .blue, .orange, .mint, .purple, .yellow, .green, .pink
]
```

Available on all platforms.

Loads content immediately, but sometimes your app needs to load on request, e.g. feed.  New refreshable modifier.
```swift
struct ContentView: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      List {
        ForEach(photoStore.photos) { photo in
          AsyncImage(url: photo.url)
            .frame(minHeight: 200)
            .mask(RoundedRectangle(cornerRadius: 16))
            .listRowSeparator(.hidden)
        }
      }
      .listStyle(.plain)
      .navigationTitle("Superhero Recruits")
      .refreshable {
        await photoStore.update()
      }
    }
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]

  func update() async {
    // Fetch new photos
  }
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}
```

You may have noticed `await`.  New concurrency language features in Swift 5.5.  Indicates `updateItems` is an async action.

Refresh our list without blocking the UI.

`.task`.  Lets you attach an async task to the lifetime of your view.  That means the task will kick off and cancel when the view is removed.

Great way for us to load the first batch of photos automatically.

```swift
struct ContentView: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      List {
        ForEach(photoStore.photos) { photo in
          AsyncImage(url: photo.url)
            .frame(minHeight: 200)
            .mask(RoundedRectangle(cornerRadius: 16))
            .listRowSeparator(.hidden)
        }
      }
      .listStyle(.plain)
      .navigationTitle("Superhero Recruits")
      .refreshable {
        await photoStore.update()
      }
      .task {
        await photoStore.update()
      }
    }
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]

  func update() async {
    // Fetch new photos
  }
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}
```

Here I have a task for loading newest photos as they become available.

NewestCandidates is now an `AsyncSequence`.  Will wait for the newest candidate asynchronously, iterating only when the next candidate is available.  Packing a ton of functionality into this single modifier.

View will update every time a new candidate becomes available, cancelling the task when the view disappears.

```swift
struct ContentView: View {
  @StateObject private var photoStore = PhotoStore()

  var body: some View {
    NavigationView {
      List {
        ForEach(photoStore.photos) { photo in
          AsyncImage(url: photo.url)
            .frame(minHeight: 200)
            .mask(RoundedRectangle(cornerRadius: 16))
            .listRowSeparator(.hidden)
        }
      }
      .listStyle(.plain)
      .navigationTitle("Superhero Recruits")
      .refreshable {
        await photoStore.update()
      }
      .task {
        for await photo in photoStore.newestPhotos {
          photoStore.push(photo)
        }
      }
    }
  }
}

class PhotoStore: ObservableObject {
  @Published var photos: [Photo] = [/* Default photos */]

  var newestPhotos: NewestPhotos {
    NewestPhotos()
  }

  func update() async {
    // Fetch new photos from remote service
  }

  func push(_ photo: Photo) {
    photos.append(photo)
  }
}

struct NewestPhotos: AsyncSequence {
  struct AsyncIterator: AsyncIteratorProtocol {
    func next() async -> Photo? {
      // Fetch next photo from remote service
    }
  }

  func makeAsyncIterator() -> AsyncIterator {
    AsyncIterator()
  }
}

struct Photo: Identifiable {
  var id: URL { url }
  var url: URL
}
```

[[Discover concurrency in SwiftUI]]
[[Swift concurrency update a sample app]]

## Interactive collections

non-interactive example

```swift
struct ContentView: View {
  @State var directions: [Direction] = [
    Direction(symbol: "car", color: .mint, text: "Drive to SFO"),
    Direction(symbol: "airplane", color: .blue, text: "Fly to SJC"),
    Direction(symbol: "tram", color: .purple, text: "Ride to Cupertino"),
    Direction(symbol: "bicycle", color: .orange, text: "Bike to Apple Park"),
    Direction(symbol: "figure.walk", color: .green, text: "Walk to pond"),
    Direction(symbol: "lifepreserver", color: .blue, text: "Swim to the center"),
    Direction(symbol: "drop", color: .indigo, text: "Dive to secret airlock"),
    Direction(symbol: "tram.tunnel.fill", color: .brown, text: "Ride through underground tunnels"),
    Direction(symbol: "key", color: .red, text: "Enter door code:"),
  ]

  var body: some View {
    NavigationView {
      List(directions) { direction in
        Label {
          Text(direction.text)
        } icon: {
          DirectionsIcon(direction)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Secret Hideout")
    }
  }
}

struct Direction: Identifiable {
  var id = UUID()
  var symbol: String
  var color: Color
  var text: String
}

private struct DirectionsIcon: View {
  var direction: Direction

  init(_ direction: Direction) {
    self.direction = direction
  }

  var body: some View {
    Image(systemName: direction.symbol)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .foregroundStyle(.white)
      .padding(6)
      .frame(width: 33, height: 33)
      .background(direction.color, in: RoundedRectangle(cornerRadius: 8))
  }
}
```

interactive example

```swift
struct ContentView: View {
  @State var directions: [Direction] = [
    Direction(symbol: "car", color: .mint, text: "Drive to SFO"),
    Direction(symbol: "airplane", color: .blue, text: "Fly to SJC"),
    Direction(symbol: "tram", color: .purple, text: "Ride to Cupertino"),
    Direction(symbol: "bicycle", color: .orange, text: "Bike to Apple Park"),
    Direction(symbol: "figure.walk", color: .green, text: "Walk to pond"),
    Direction(symbol: "lifepreserver", color: .blue, text: "Swim to the center"),
    Direction(symbol: "drop", color: .indigo, text: "Dive to secret airlock"),
    Direction(symbol: "tram.tunnel.fill", color: .brown, text: "Ride through underground tunnels"),
    Direction(symbol: "key", color: .red, text: "Enter door code:"),
  ]

  var body: some View {
    NavigationView {
      List($directions) { $direction in
        Label {
          TextField("Instructions", text: $direction.text)
        } icon: {
          DirectionsIcon(direction)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Secret Hideout")
    }
  }
}

struct Direction: Identifiable {
  var id = UUID()
  var symbol: String
  var color: Color
  var text: String
}

private struct DirectionsIcon: View {
  var direction: Direction

  init(_ direction: Direction) {
    self.direction = direction
  }

  var body: some View {
    Image(systemName: direction.symbol)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .foregroundStyle(.white)
      .padding(6)
      .frame(width: 33, height: 33)
      .background(direction.color, in: RoundedRectangle(cornerRadius: 8))
  }
}
```

In situations like this, it can be tricky to figure out  how to get a binding for each row.  One common approach is to iterate over indices of collection using a subscript.  However, this technique is not recommended, since SwiftUI will reload the whole list when anything changes.

[[Demystify SwiftUI]]

For now, let's do another solution.  This year, we're providing bindings for individual elements, simply pass a binding into the list with normal `$directions` operator, swift will pass back a binding to each element.  But now we can easily add interactive controls.

New syntax is part of swift language.  For example, we can use it in ForEach.

```swift
struct ContentView: View {
  @State var directions: [Direction] = [
    Direction(symbol: "car", color: .mint, text: "Drive to SFO"),
    Direction(symbol: "airplane", color: .blue, text: "Fly to SJC"),
    Direction(symbol: "tram", color: .purple, text: "Ride to Cupertino"),
    Direction(symbol: "bicycle", color: .orange, text: "Bike to Apple Park"),
    Direction(symbol: "figure.walk", color: .green, text: "Walk to pond"),
    Direction(symbol: "lifepreserver", color: .blue, text: "Swim to the center"),
    Direction(symbol: "drop", color: .indigo, text: "Dive to secret airlock"),
    Direction(symbol: "tram.tunnel.fill", color: .brown, text: "Ride through underground tunnels"),
    Direction(symbol: "key", color: .red, text: "Enter door code:"),
  ]

  var body: some View {
    NavigationView {
      List {
        ForEach($directions) { $direction in
          Label {
            TextField("Instructions", text: $direction.text)
          } icon: {
            DirectionsIcon(direction)
          }
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Secret Hideout")
    }
  }
}

struct Direction: Identifiable {
  var id = UUID()
  var symbol: String
  var color: Color
  var text: String
}

private struct DirectionsIcon: View {
  var direction: Direction

  init(_ direction: Direction) {
    self.direction = direction
  }

  var body: some View {
    Image(systemName: direction.symbol)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .foregroundStyle(.white)
      .padding(6)
      .frame(width: 33, height: 33)
      .background(direction.color, in: RoundedRectangle(cornerRadius: 8))
  }
}
```

Can even backdeploy to prior releases!

List row separator tint

## Color of row separators
```swift
struct ContentView: View {
  @State var directions: [Direction] = [
    Direction(symbol: "car", color: .mint, text: "Drive to SFO"),
    Direction(symbol: "airplane", color: .blue, text: "Fly to SJC"),
    Direction(symbol: "tram", color: .purple, text: "Ride to Cupertino"),
    Direction(symbol: "bicycle", color: .orange, text: "Bike to Apple Park"),
    Direction(symbol: "figure.walk", color: .green, text: "Walk to pond"),
    Direction(symbol: "lifepreserver", color: .blue, text: "Swim to the center"),
    Direction(symbol: "drop", color: .indigo, text: "Dive to secret airlock"),
    Direction(symbol: "tram.tunnel.fill", color: .brown, text: "Ride through underground tunnels"),
    Direction(symbol: "key", color: .red, text: "Enter door code:"),
  ]

  var body: some View {
    NavigationView {
      List {
        ForEach($directions) { $direction in
          Label {
            TextField("Instructions", text: $direction.text)
          } icon: {
            DirectionsIcon(direction)
          }
          .listRowSeparatorTint(direction.color)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Secret Hideout")
    }
  }
}

struct Direction: Identifiable {
  var id = UUID()
  var symbol: String
  var color: Color
  var text: String
}

private struct DirectionsIcon: View {
  var direction: Direction

  init(_ direction: Direction) {
    self.direction = direction
  }

  var body: some View {
    Image(systemName: direction.symbol)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .foregroundStyle(.white)
      .padding(6)
      .frame(width: 33, height: 33)
      .background(direction.color, in: RoundedRectangle(cornerRadius: 8))
  }
}
```

These separators seem distracting.  Maybe remove the separator?

```swift
struct ContentView: View {
  @State var directions: [Direction] = [
    Direction(symbol: "car", color: .mint, text: "Drive to SFO"),
    Direction(symbol: "airplane", color: .blue, text: "Fly to SJC"),
    Direction(symbol: "tram", color: .purple, text: "Ride to Cupertino"),
    Direction(symbol: "bicycle", color: .orange, text: "Bike to Apple Park"),
    Direction(symbol: "figure.walk", color: .green, text: "Walk to pond"),
    Direction(symbol: "lifepreserver", color: .blue, text: "Swim to the center"),
    Direction(symbol: "drop", color: .indigo, text: "Dive to secret airlock"),
    Direction(symbol: "tram.tunnel.fill", color: .brown, text: "Ride through underground tunnels"),
    Direction(symbol: "key", color: .red, text: "Enter door code:"),
  ]

  var body: some View {
    NavigationView {
      List {
        ForEach($directions) { $direction in
          Label {
            TextField("Instructions", text: $direction.text)
          } icon: {
            DirectionsIcon(direction)
          }
          .listRowSeparator(.hidden)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Secret Hideout")
    }
  }
}

struct Direction: Identifiable {
  var id = UUID()
  var symbol: String
  var color: Color
  var text: String
}

private struct DirectionsIcon: View {
  var direction: Direction

  init(_ direction: Direction) {
    self.direction = direction
  }

  var body: some View {
    Image(systemName: direction.symbol)
      .resizable()
      .aspectRatio(contentMode: .fit)
      .foregroundStyle(.white)
      .padding(6)
      .frame(width: 33, height: 33)
      .background(direction.color, in: RoundedRectangle(cornerRadius: 8))
  }
}
```

Now our directions feel less cluttered.

## Swipe actions

This app uses swipe actions to pin/delete characters without cluttering UI.
```swift
struct ContentView: View {
  @State private var characters = CharacterStore(StoryCharacter.previewData)

  var body: some View {
    NavigationView {
      List {
        if !characters.pinned.isEmpty {
          Section("Pinned") {
            sectionContent(for: $characters.pinned)
          }
        }
        Section("Heroes & Villains") {
          sectionContent(for: $characters.unpinned)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Characters")
    }
  }

  @ViewBuilder
  private func sectionContent(for characters: Binding<[StoryCharacter]>) -> some View {
    ForEach(characters) { $character in
      CharacterProfile(character)
        .swipeActions {
          Button {
            togglePinned(for: $character)
          } label: {
            if character.isPinned {
              Label("Unpin", systemImage: "pin.slash")
            } else {
              Label("Pin", systemImage: "pin")
            }
          }
          .tint(.yellow)
        }
    }
  }

  private func togglePinned(for character: Binding<StoryCharacter>) {
    withAnimation {
      var tmp = character.wrappedValue
      tmp.isPinned.toggle()
      tmp.lastModified = Date()
      character.wrappedValue = tmp
    }
  }

  private func delete<C: RangeReplaceableCollection & MutableCollection>(
    _ character: StoryCharacter, in characters: Binding<C>
  ) where C.Element == StoryCharacter {
    withAnimation {
      if let i = characters.wrappedValue.firstIndex(where: {
        $0.id == character.id
      }) {
        characters.wrappedValue.remove(at: i)
      }
    }
  }
}

struct CharacterProfile: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    NavigationLink {
      Text(character.name)
    } label: {
      HStack {
        HStack {
          let symbol = Image(systemName: character.symbol)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .foregroundStyle(.white)
            .padding(6)
            .frame(width: 33, height: 33)

          if character.isVillain {
            symbol
              .background(character.color, in: RoundedRectangle(cornerRadius: 8))
          } else {
            symbol
              .background(character.color, in: Circle())
          }
        }
        VStack(alignment: .leading, spacing: 2) {
          HStack(alignment: .center) {
            Text(character.name)
              .bold()
              .foregroundStyle(.primary)
          }
          HStack(spacing: 4) {
            Text(character.isVillain ? "VILLAIN" : "HERO")
              .bold()
              .font(.caption2.weight(.heavy))
              .foregroundStyle(.white)
              .padding(.vertical, 1)
              .padding(.horizontal, 3)
              .background(.quaternary, in: RoundedRectangle(cornerRadius: 3))

            Text(character.powers)
              .font(.footnote)
              .foregroundStyle(.secondary)
          }
        }
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```

Just like any other kind fo menu in swiftUI.  Also customize their color adding new tint modifier.

By default, swiftUI shows swipe actions on the trailing edge.  Can switch to leading edge.  

```swift
struct ContentView: View {
  @State private var characters = CharacterStore(StoryCharacter.previewData)

  var body: some View {
    NavigationView {
      List {
        if !characters.pinned.isEmpty {
          Section("Pinned") {
            sectionContent(for: $characters.pinned)
          }
        }
        Section("Heroes & Villains") {
          sectionContent(for: $characters.unpinned)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Characters")
    }
  }

  @ViewBuilder
  private func sectionContent(for characters: Binding<[StoryCharacter]>) -> some View {
    ForEach(characters) { $character in
      CharacterProfile(character)
        .swipeActions(edge: .leading) {
          Button {
            togglePinned(for: $character)
          } label: {
            if character.isPinned {
              Label("Unpin", systemImage: "pin.slash")
            } else {
              Label("Pin", systemImage: "pin")
            }
          }
          .tint(.yellow)
        }
    }
  }

  private func togglePinned(for character: Binding<StoryCharacter>) {
    withAnimation {
      var tmp = character.wrappedValue
      tmp.isPinned.toggle()
      tmp.lastModified = Date()
      character.wrappedValue = tmp
    }
  }

  private func delete<C: RangeReplaceableCollection & MutableCollection>(
    _ character: StoryCharacter, in characters: Binding<C>
  ) where C.Element == StoryCharacter {
    withAnimation {
      if let i = characters.wrappedValue.firstIndex(where: {
        $0.id == character.id
      }) {
        characters.wrappedValue.remove(at: i)
      }
    }
  }
}

struct CharacterProfile: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    NavigationLink {
      Text(character.name)
    } label: {
      HStack {
        HStack {
          let symbol = Image(systemName: character.symbol)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .foregroundStyle(.white)
            .padding(6)
            .frame(width: 33, height: 33)

          if character.isVillain {
            symbol
              .background(character.color, in: RoundedRectangle(cornerRadius: 8))
          } else {
            symbol
              .background(character.color, in: Circle())
          }
        }
        VStack(alignment: .leading, spacing: 2) {
          HStack(alignment: .center) {
            Text(character.name)
              .bold()
              .foregroundStyle(.primary)
          }
          HStack(spacing: 4) {
            Text(character.isVillain ? "VILLAIN" : "HERO")
              .bold()
              .font(.caption2.weight(.heavy))
              .foregroundStyle(.white)
              .padding(.vertical, 1)
              .padding(.horizontal, 3)
              .background(.quaternary, in: RoundedRectangle(cornerRadius: 3))

            Text(character.powers)
              .font(.footnote)
              .foregroundStyle(.secondary)
          }
        }
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```

Can even support mixed leading/trailing.

```swift
struct ContentView: View {
  @State private var characters = CharacterStore(StoryCharacter.previewData)

  var body: some View {
    NavigationView {
      List {
        if !characters.pinned.isEmpty {
          Section("Pinned") {
            sectionContent(for: $characters.pinned)
          }
        }
        Section("Heroes & Villains") {
          sectionContent(for: $characters.unpinned)
        }
      }
      .listStyle(.sidebar)
      .navigationTitle("Characters")
    }
  }

  @ViewBuilder
  private func sectionContent(for characters: Binding<[StoryCharacter]>) -> some View {
    ForEach(characters) { $character in
      CharacterProfile(character)
        .swipeActions(edge: .leading) {
          Button {
            togglePinned(for: $character)
          } label: {
            if character.isPinned {
              Label("Unpin", systemImage: "pin.slash")
            } else {
              Label("Pin", systemImage: "pin")
            }
          }
          .tint(.yellow)
        }
        .swipeActions(edge: .trailing) {
          Button(role: .destructive) {
            delete(character, in: characters)
          } label: {
            Label("Delete", systemImage: "trash")
          }
          Button {
            // Open "More" menu
          } label: {
            Label("More", systemImage: "ellipsis.circle")
          }
          .tint(Color(white: 0.8))
        }
    }
  }

  private func togglePinned(for character: Binding<StoryCharacter>) {
    withAnimation {
      var tmp = character.wrappedValue
      tmp.isPinned.toggle()
      tmp.lastModified = Date()
      character.wrappedValue = tmp
    }
  }

  private func delete<C: RangeReplaceableCollection & MutableCollection>(
    _ character: StoryCharacter, in characters: Binding<C>
  ) where C.Element == StoryCharacter {
    withAnimation {
      if let i = characters.wrappedValue.firstIndex(where: {
        $0.id == character.id
      }) {
        characters.wrappedValue.remove(at: i)
      }
    }
  }
}

struct CharacterProfile: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    NavigationLink {
      Text(character.name)
    } label: {
      HStack {
        HStack {
          let symbol = Image(systemName: character.symbol)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .foregroundStyle(.white)
            .padding(6)
            .frame(width: 33, height: 33)

          if character.isVillain {
            symbol
              .background(character.color, in: RoundedRectangle(cornerRadius: 8))
          } else {
            symbol
              .background(character.color, in: Circle())
          }
        }
        VStack(alignment: .leading, spacing: 2) {
          HStack(alignment: .center) {
            Text(character.name)
              .bold()
              .foregroundStyle(.primary)
          }
          HStack(spacing: 4) {
            Text(character.isVillain ? "VILLAIN" : "HERO")
              .bold()
              .font(.caption2.weight(.heavy))
              .foregroundStyle(.white)
              .padding(.vertical, 1)
              .padding(.horizontal, 3)
              .background(.quaternary, in: RoundedRectangle(cornerRadius: 3))

            Text(character.powers)
              .font(.footnote)
              .foregroundStyle(.secondary)
          }
        }
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```

Available multiplatform.  Let's check in on #macOS 

List does feel plain.  How to spruce it up?

## Basic macOS list
```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData
  @State private var selection = Set<StoryCharacter.ID>()

  var body: some View {
    List(selection: $selection) {
      ForEach(characters) { character in
        Label {
          Text(character.name)
        } icon: {
          CharacterIcon(character)
        }
        .padding(.leading, 4)
      }
    }
    .listStyle(.inset)
    .navigationTitle("All Characters")
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(4)
        .frame(width: 20, height: 20)

      if character.isVillain {
        symbol
          .background(character.color, in: RoundedRectangle(cornerRadius: 4))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```

inset list style alternating row backgrounds

```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData
  @State private var selection = Set<StoryCharacter.ID>()

  var body: some View {
    List(selection: $selection) {
      ForEach(characters) { character in
        Label {
          Text(character.name)
        } icon: {
          CharacterIcon(character)
        }
        .padding(.leading, 4)
      }
    }
    .listStyle(.inset(alternatesRowBackgrounds: true))
    .navigationTitle("All Characters")
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(4)
        .frame(width: 20, height: 20)

      if character.isVillain {
        symbol
          .background(character.color, in: RoundedRectangle(cornerRadius: 4))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```
Each row clearly distinguished from the other.

But for a macOS app, it still feels like we can do more.


# Beyond lists
## Tables
four lists for the price of one!

> So little code it fits on a single slide

lies

```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData

  var body: some View {
    Table(characters) {
      TableColumn("􀟈") { CharacterIcon($0) }
        .width(20)
      TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
        .width(40)
      TableColumn("Name", value: \.name)
      TableColumn("Powers", value: \.powers)
    }
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(4)
        .frame(width: 20, height: 20)

      if character.isVillain {
        symbol
          .background(character.color, in: RoundedRectangle(cornerRadius: 4))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```


Unlike a list, a table is made up of `TableColumns` that define content in each visual column.  Use data from the collection to define the visual content.

Just displaying text.

Tables are interactive, supporting row selection and multiple row selection.

```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData

  // Single selection
  @State private var singleSelection: StoryCharacter.ID?

  // Multiple selection
  @State private var multipleSelection: Set<StoryCharacter.ID>()

  var body: some View {
    Table(characters, selection: $singleSelection) { // or `$multipleSelection`
      TableColumn("􀟈") { CharacterIcon($0) }
        .width(20)
      TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
        .width(40)
      TableColumn("Name", value: \.name)
      TableColumn("Powers", value: \.powers)
    }
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(4)
        .frame(width: 20, height: 20)

      if character.isVillain {
        symbol
          .background(character.color, in: RoundedRectangle(cornerRadius: 4))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```



Also support sorting.  Keypaths, etc.

```swift
struct ContentView: View {
  @State private var characters = StoryCharacter.previewData
  @State private var selection = Set<StoryCharacter.ID>()

  @State private var sortOrder = [KeyPathComparator(\StoryCharacter.name)]
  @State private var sorted: [StoryCharacter]?

  var body: some View {
    Table(sorted ?? characters, selection: $selection, sortOrder: $sortOrder) {
      TableColumn("􀟈") { CharacterIcon($0) }
        .width(20)
      TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
        .width(40)
      TableColumn("Name", value: \.name)
      TableColumn("Powers", value: \.powers)
    }
    .onChange(of: characters) { sorted = $0.sorted(using: sortOrder) }
    .onChange(of: sortOrder) { sorted = characters.sorted(using: $0) }
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(4)
        .frame(width: 20, height: 20)

      if character.isVillain {
        symbol
          .background(character.color, in: RoundedRectangle(cornerRadius: 4))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    get {
      all.prefix { $0.isPinned }
    }
    set {
      if let end = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(all.startIndex..<end, with: newValue)
      }
    }
  }

  var unpinned: [StoryCharacter] {
    get {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        return Array(all.suffix(from: start))
      } else {
        return []
      }
    }
    set {
      if let start = all.firstIndex(where: { !$0.isPinned }) {
        all.replaceSubrange(start..<all.endIndex, with: newValue)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```



Several other features, including visual styles, fine-tuning appearance, etc.  Let's talk about data though.

## CoreData tables
```swift
@FetchRequest(sortDescriptors: [SortDescriptor(\.name)])
private var characters: FetchedResults<StoryCharacter>
@State private var selection = Set<StoryCharacter.ID>()

Table(characters, selection: $selection, sortOrder: $characters.sortDescriptors) {
  TableColumn("􀟈") { CharacterIcon($0) }
    .width(20)
  TableColumn("Villain") { Text($0.isVillain ? "Villain" : "Hero") }
    .width(40)
  TableColumn("Name", value: \.name)
  TableColumn("Powers", value: \.powers)
}
```

Sectioned fetch requests.  Complex, multi-section lists to be driven by a single request.  In this example, we partition into sections based on whether or not they're pinned
Use multiple sort descriptors
Animation
Sections/rows dynamically based on the results of the request.



```swift
@SectionedFetchRequest(
  sectionIdentifier: \.isPinned,
  sortDescriptors: [
    SortDescriptor(\.isPinned, order: .reverse),
    SortDescriptor(\.lastModified)
  ],
  animation: .default)
private var characters: SectionedFetchResults<...>

List {
  ForEach(characters) { section in
    Section(section.id ? "Pinned" : "Heroes & Villains") {
      ForEach(section) { character in
        CharacterRowView(character)
      }
    }
  }
}
```

[[SwiftUI on the Mac Build the fundamentals]]
[[SwiftUI on the Mac The finishing touches]]
[[Bring Core Data concurrency to Swift and SwiftUI]]

Let's step back and think about finding what you need.  

## Search
Since search is a multiplatform problem, it needs a multiplatform solution.

Luckily, we implemented this.
`searchable()` modifier
```swift
struct ContentView: View {
  @State private var characters = CharacterStore(StoryCharacter.previewData)

  var body: some View {
    NavigationView {
      List {
        if characters.filterText.isEmpty {
          if !characters.pinned.isEmpty {
            Section("Pinned") {
              sectionContent(for: characters.pinned)
            }
          }
          Section("Heroes & Villains") {
            sectionContent(for: characters.unpinned)
          }
        } else {
          sectionContent(for: characters.filtered)
        }
      }
      .listStyle(.sidebar)
      .searchable(text: $characters.filterText)
      .navigationTitle("Characters")
    }
  }

  @ViewBuilder
  private func sectionContent(for characters: [StoryCharacter]) -> some View {
    ForEach(characters) { character in
      CharacterProfile(character)
    }
  }
}

struct CharacterProfile: View {
  var character: StoryCharacter

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    NavigationLink {
      Text(character.name)
    } label: {
      HStack {
        HStack {
          let symbol = Image(systemName: character.symbol)
            .resizable()
            .aspectRatio(contentMode: .fit)
            .foregroundStyle(.white)
            .padding(6)
            .frame(width: 33, height: 33)

          if character.isVillain {
            symbol
              .background(character.color, in: RoundedRectangle(cornerRadius: 8))
          } else {
            symbol
              .background(character.color, in: Circle())
          }
        }
        VStack(alignment: .leading, spacing: 2) {
          HStack(alignment: .center) {
            Text(character.name)
              .bold()
              .foregroundStyle(.primary)
          }
          HStack(spacing: 4) {
            Text(character.isVillain ? "VILLAIN" : "HERO")
              .bold()
              .font(.caption2.weight(.heavy))
              .foregroundStyle(.white)
              .padding(.vertical, 1)
              .padding(.horizontal, 3)
              .background(.quaternary, in: RoundedRectangle(cornerRadius: 3))

            Text(character.powers)
              .font(.footnote)
              .foregroundStyle(.secondary)
          }
        }
      }
    }
  }
}

struct CharacterStore {
  var all: [StoryCharacter] {
    get { _all }
    set { _all = newValue; sortAll() }
  }
  var _all: [StoryCharacter]

  var pinned: [StoryCharacter] {
    all.prefix { $0.isPinned }
  }

  var unpinned: [StoryCharacter] {
    if let start = all.firstIndex(where: { !$0.isPinned }) {
      return Array(all.suffix(from: start))
    } else {
      return []
    }
  }

  var filterText: String = ""
  var filtered: [StoryCharacter] {
    if filterText.isEmpty {
      return all
    } else {
      return all.filter {
        $0.name.contains(filterText) || $0.powers.contains(filterText)
      }
    }
  }

  init(_ characters: [StoryCharacter]) {
    _all = characters
    sortAll()
  }

  private mutating func sortAll() {
    _all.sort { lhs, rhs in
      if lhs.isPinned && !rhs.isPinned {
        return true
      } else if !lhs.isPinned && rhs.isPinned {
        return false
      } else {
        return lhs.lastModified < rhs.lastModified
      }
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
}

extension StoryCharacter {
  static let previewData: [StoryCharacter] = [
    StoryCharacter(
      id: 0,
      name: "The View Builder",
      symbol: "hammer",
      color: .pink,
      powers: "Conjures objects on-demand.",
      isPinned: true),
    StoryCharacter(
      id: 1,
      name: "The Truth Duplicator",
      symbol: "eyes",
      color: .blue,
      powers: "Distorts reality.",
      isVillain: true),
    StoryCharacter(
      id: 2,
      name: "The Previewer",
      symbol: "viewfinder",
      color: .indigo,
      powers: "Reveals the future.",
      isPinned: true),
    StoryCharacter(
      id: 3,
      name: "The Type Eraser",
      symbol: "eye.slash",
      color: .black,
      powers: "Steals identities.",
      isVillain: true,
      isPinned: true),
    StoryCharacter(
      id: 4,
      name: "The Environment Modifier",
      symbol: "leaf",
      color: .green,
      powers: "Controls the physical world."),
    StoryCharacter(
      id: 5,
      name: "The Unstable Identifier",
      symbol: "shuffle",
      color: .brown,
      powers: "Shape-shifter, uncatchable.",
      isVillain: true),
    StoryCharacter(
      id: 6,
      name: "The Stylizer",
      symbol: "wand.and.stars.inverse",
      color: .red,
      powers: "Quartermaster of heroes."),
    StoryCharacter(
      id: 7,
      name: "The Singleton",
      symbol: "diamond",
      color: .purple,
      powers: "An evil robotic hive mind.",
      isVillain: true),
    StoryCharacter(
      id: 8,
      name: "The Geometry Reader",
      symbol: "ruler",
      color: .orange,
      powers: "Instantly scans any structure."),
    StoryCharacter(
      id: 9,
      name: "The Opaque Typist",
      symbol: "app.fill",
      color: .teal,
      powers: "Creates impenetrable disguises."),
    StoryCharacter(
      id: 10,
      name: "The Unobservable Man",
      symbol: "hand.raised.slash",
      color: .black,
      powers: "Impervious to detection.",
      isVillain: true),
  ]
}
```

SwiftUI will add a search field to the appropriate location, and show suggestions in a pltform and context-appropriate way.

Allowing you to filter your data
[[Craft search experiences in SwiftUI]]

## Share data
Drag and drop.

New this year, you can add custom drag previews.  This preview is shown instead of the view while it's being dragged.  Drag/drop is powered by item provider `NSItemProvider`.




Drag previews
```swift
struct ContentView: View {
  let character = StoryCharacter(
    id: 0,
    name: "The View Builder",
    symbol: "hammer",
    color: .pink,
    powers: "Conjures objects on-demand.",
    isPinned: true
  )

  var body: some View {
    CharacterIcon(character)
      .controlSize(.large)
      .padding()
      .onDrag {
        character.itemProvider
      } preview: {
        Label {
          Text(character.name)
        } icon: {
          CharacterIcon(character)
            .controlSize(.small)
        }
        .padding(.vertical, 8)
        .frame(width: 150)
        .background(.white, in: RoundedRectangle(cornerRadius: 8))
      }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()

  var itemProvider: NSItemProvider {
    let item = NSItemProvider()
    item.registerObject(name as NSString, visibility: .all)
    return item
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  #if os(iOS) || os(macOS)
  @Environment(\.controlSize) private var controlSize
  #endif

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(symbolPadding)
        .frame(width: symbolLength, height: symbolLength)

      if character.isVillain {
        symbol
          .background(
            character.color, in: RoundedRectangle(cornerRadius: cornerRadius))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }

  var symbolPadding: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 10
    default: return 6
    }
  }

  var symbolLength: CGFloat {
    switch controlSize {
    case .small: return 20
    case .large: return 60
    default: return 33
    }
  }

  var cornerRadius: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 16
    default: return 8
    }
  }
}
```

This year, SwiftUI is providing several more ways to use item proveders.

`importsItemProviders()` modifier

```swift
import UniformTypeIdentifiers

@main
private struct Catalog: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    .commands {
      ImportFromDevicesCommands()
    }
  }
}

struct ContentView: View {
  @State private var character: StoryCharacter = StoryCharacter(
    id: 0,
    name: "The View Builder",
    symbol: "hammer",
    color: .pink,
    powers: "Conjures objects on-demand.",
    isPinned: true
  )

  var body: some View {
    VStack {
      CharacterIcon(character)
        .controlSize(.large)
        .onDrag {
          character.itemProvider
        } preview: {
          Label {
            Text(character.name)
          } icon: {
            CharacterIcon(character)
              .controlSize(.small)
          }
          .padding(.vertical, 8)
          .frame(width: 150)
          .background(.white, in: RoundedRectangle(cornerRadius: 8))
        }

      if let headerImage = character.headerImage {
        headerImage
          .resizable()
          .aspectRatio(contentMode: .fill)
          .frame(width: 150, height: 150)
          .mask(RoundedRectangle(cornerRadius: 16, style: .continuous))
      }
    }
    .padding()
    .importsItemProviders(StoryCharacter.headerImageTypes) { itemProviders in
      guard let first = itemProviders.first else { return false }
      async {
        character.headerImage = await StoryCharacter.loadHeaderImage(from: first)
      }
      return true
    }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
  var headerImage: Image?

  static var headerImageTypes: [UTType] {
    NSImage.imageTypes.compactMap { UTType($0) }
  }

  var itemProvider: NSItemProvider {
    let item = NSItemProvider()
    item.registerObject(name as NSString, visibility: .all)
    return item
  }

  static func loadHeaderImage(from itemProvider: NSItemProvider) async -> Image? {
    for type in Self.headerImageTypes.map(\.identifier) {
      if itemProvider.hasRepresentationConforming(toTypeIdentifier: type) {
        return await withCheckedContinuation { continuation in
          itemProvider.loadDataRepresentation(forTypeIdentifier: type) { data, error in
            guard let data = data, let image = NSImage(data: data) else { return }
            continuation.resume(returning: Image(nsImage: image))
          }
        }
      }
    }
    return nil
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  #if os(iOS) || os(macOS)
  @Environment(\.controlSize) private var controlSize
  #endif

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(symbolPadding)
        .frame(width: symbolLength, height: symbolLength)

      if character.isVillain {
        symbol
          .background(
            character.color, in: RoundedRectangle(cornerRadius: cornerRadius))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }

  var symbolPadding: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 10
    default: return 6
    }
  }

  var symbolLength: CGFloat {
    switch controlSize {
    case .small: return 20
    case .large: return 60
    default: return 33
    }
  }

  var cornerRadius: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 16
    default: return 8
    }
  }
}
```

Can pair this with continuity camera.  By adding `ImportFromDevicesCommands()` can now use iPhone/iPad to take photos.

( Demo.)

Exporting data allows you to trigger shortcuts, etc
`exportsItemProvider` modifier

```swift
import UniformTypeIdentifiers

@main
private struct Catalog: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
    }
    .commands {
      ImportFromDevicesCommands()
    }
  }
}

struct ContentView: View {
  @State private var character: StoryCharacter = StoryCharacter(
    id: 0,
    name: "The View Builder",
    symbol: "hammer",
    color: .pink,
    powers: "Conjures objects on-demand.",
    isPinned: true
  )

  var body: some View {
    VStack {
      CharacterIcon(character)
        .controlSize(.large)
        .onDrag {
          character.itemProvider
        } preview: {
          Label {
            Text(character.name)
          } icon: {
            CharacterIcon(character)
              .controlSize(.small)
          }
          .padding(.vertical, 8)
          .frame(width: 150)
          .background(.white, in: RoundedRectangle(cornerRadius: 8))
        }

      if let headerImage = character.headerImage {
        headerImage
          .resizable()
          .aspectRatio(contentMode: .fill)
          .frame(width: 150, height: 150)
          .mask(RoundedRectangle(cornerRadius: 16, style: .continuous))
      }
    }
    .padding()
    .importsItemProviders(StoryCharacter.headerImageTypes) { itemProviders in
      guard let first = itemProviders.first else { return false }
      async {
        character.headerImage = await StoryCharacter.loadHeaderImage(from: first)
      }
      return true
    }
    .exportsItemProviders(StoryCharacter.contentTypes) { [character.itemProvider] }
  }
}

struct StoryCharacter: Identifiable, Equatable {
  var id: Int64
  var name: String
  var symbol: String
  var color: Color
  var powers: String
  var isVillain: Bool = false
  var isPinned: Bool = false
  var lastModified = Date()
  var headerImage: Image?

  static var contentTypes: [UTType] { [.utf8PlainText] }
  static var headerImageTypes: [UTType] {
    NSImage.imageTypes.compactMap { UTType($0) }
  }

  var itemProvider: NSItemProvider {
    let item = NSItemProvider()
    item.registerObject(name as NSString, visibility: .all)
    return item
  }

  static func loadHeaderImage(from itemProvider: NSItemProvider) async -> Image? {
    for type in Self.headerImageTypes.map(\.identifier) {
      if itemProvider.hasRepresentationConforming(toTypeIdentifier: type) {
        return await withCheckedContinuation { continuation in
          itemProvider.loadDataRepresentation(forTypeIdentifier: type) { data, error in
            guard let data = data, let image = NSImage(data: data) else { return }
            continuation.resume(returning: Image(nsImage: image))
          }
        }
      }
    }
    return nil
  }
}

struct CharacterIcon: View {
  var character: StoryCharacter

  #if os(iOS) || os(macOS)
  @Environment(\.controlSize) private var controlSize
  #endif

  init(_ character: StoryCharacter) {
    self.character = character
  }

  var body: some View {
    HStack {
      let symbol = Image(systemName: character.symbol)
        .resizable()
        .aspectRatio(contentMode: .fit)
        .foregroundStyle(.white)
        .padding(symbolPadding)
        .frame(width: symbolLength, height: symbolLength)

      if character.isVillain {
        symbol
          .background(
            character.color, in: RoundedRectangle(cornerRadius: cornerRadius))
      } else {
        symbol
          .background(character.color, in: Circle())
      }
    }
  }

  var symbolPadding: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 10
    default: return 6
    }
  }

  var symbolLength: CGFloat {
    switch controlSize {
    case .small: return 20
    case .large: return 60
    default: return 33
    }
  }

  var cornerRadius: CGFloat {
    switch controlSize {
    case .small: return 4
    case .large: return 16
    default: return 8
    }
  }
}
```

ex, allowing it to be used by services and shorcuts.  Demo.

See quick actions in Services menu when I selected one of my characters.  e.g. send to shortcut.

> This adorable image is a great segue to the next section

# Advanced graphics
## Symbols
Not only are there many new symbols, but they have many new features.

 hierarchical uses the current foreground style, with multiple levels of opacity.
Palette gives you control over individual layers.

[[What's new in SF symbols]]

Set of colors thata re optimized.  light/dark, platform, etc.  Symbols come in different shapes.

Symbol rendering modes. 

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            HStack { symbols }
                .symbolRenderingMode(.monochrome)
            HStack { symbols }
                .symbolRenderingMode(.multicolor)
            HStack { symbols }
                .symbolRenderingMode(.hierarchical)
            HStack { symbols }
                .symbolRenderingMode(.palette)
                .foregroundStyle(Color.cyan, Color.purple)
        }
        .foregroundStyle(.blue)
        .font(.title)
    }
    @ViewBuilder var symbols: some View {
        Group {
            Image(systemName: "exclamationmark.triangle.fill")
            Image(systemName: "pc")
            Image(systemName: "phone.down.circle")
            Image(systemName: "hourglass")
            Image(systemName: "heart.fill")
            Image(systemName: "airplane.circle.fill")
        }
        .frame(width: 40, height: 40)
    }
}
```

Symbol variants

iOS HIG describes how in tab bars, filled variants should be preferred.  So you had to include `.filled` in the name.

This year, SwiftUI will automatically choose the right variant.  By not overspecifying, you get reusable code.

ex, if we get the same code on macOS, we get outlines

[[SF Symbols in SwiftUI]]

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            HStack { symbols }
            HStack { symbols }
                .symbolVariant(.fill)
        }
        .foregroundStyle(.blue)
    }
    @ViewBuilder var symbols: some View {
        let heart = Image(systemName: "heart")
        Group {
            heart
            heart.symbolVariant(.slash)
            heart.symbolVariant(.circle)
            heart.symbolVariant(.square)
            heart.symbolVariant(.rectangle)
        }
        .frame(width: 40, height: 40)
    }
}
```

TAb symbol variants: iOS 13

```swift
struct TabExample: View {
    var body: some View {
        TabView {
            CardsView().tabItem {
                Label("Cards", systemImage: "rectangle.portrait.on.rectangle.portrait.fill")
            }
            RulesView().tabItem {
                Label("Rules", systemImage: "character.book.closed.fill")
            }
            ProfileView().tabItem {
                Label("Profile", systemImage: "person.circle.fill")
            }
            SearchPlayersView().tabItem {
                Label("Magic", systemImage: "sparkles")
            }
        }
    }
}

struct CardsView: View {
    var body: some View { Color.clear }
}
struct RulesView: View {
    var body: some View { Color.clear }
}
struct ProfileView: View {
    var body: some View { Color.clear }
}
struct SearchPlayersView: View {
    var body: some View { Color.clear }
}
```

Tab symbol variants

```swift
@main
struct SnippetsApp: App {
    var body: some Scene {
        WindowGroup {
            #if os(iOS)
            TabExample()
            #else
            VStack{
                Text("Open Preferences")
                Text("⌘,").font(.title.monospaced())
            }
            .fixedSize()
            .scenePadding()
            #endif
        }

        #if os(macOS)
        Settings {
            TabExample()
        }
        #endif
    }
}


struct TabExample: View {
    var body: some View {
        TabView {
            CardsView().tabItem {
                Label("Cards", systemImage: "rectangle.portrait.on.rectangle.portrait")
            }
            RulesView().tabItem {
                Label("Rules", systemImage: "character.book.closed")
            }
            ProfileView().tabItem {
                Label("Profile", systemImage: "person.circle")
            }
            SearchPlayersView().tabItem {
                Label("Magic", systemImage: "sparkles")
            }
        }
    }
}

struct CardsView: View {
    var body: some View { Color.clear }
}
struct RulesView: View {
    var body: some View { Color.clear }
}
struct ProfileView: View {
    var body: some View { Color.clear }
}
struct SearchPlayersView: View {
    var body: some View { Color.clear }
}
```

## Canvas
Lots of elements that don't need tracking or invalidation.

Each of them into their own frame.  Canvas works on every platform, and can also attach gestures, information, etc.  Adapt to dark mode.


canvas

```swift
struct ContentView: View {
    let symbols = Array(repeating: Symbol("swift"), count: 3166)

    var body: some View {
        Canvas { context, size in
            let metrics = gridMetrics(in: size)
            for (index, symbol) in symbols.enumerated() {
                let rect = metrics[index]
                let image = context.resolve(symbol.image)
                context.draw(image, in: rect.fit(image.size))
            }
        }
    }

    func gridMetrics(in size: CGSize) -> SymbolGridMetrics {
        SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}

struct SymbolGridMetrics {
    let symbolWidth: CGFloat
    let symbolsPerRow: Int
    let numberOfSymbols: Int
    let insetProportion: CGFloat

    init(size: CGSize, numberOfSymbols: Int, insetProportion: CGFloat = 0.1) {
        let areaPerSymbol = (size.width * size.height) / CGFloat(numberOfSymbols)
        self.symbolsPerRow = Int(size.width / sqrt(areaPerSymbol))
        self.symbolWidth = size.width / CGFloat(symbolsPerRow)
        self.numberOfSymbols = numberOfSymbols
        self.insetProportion = insetProportion
    }

    /// Returns the frame in the grid for the symbol at `index` position.
    /// It is not valid to pass an index less than `0` or larger than the number of symbols the grid metrics was created for.
    subscript(_ index: Int) -> CGRect {
        precondition(index >= 0 && index < numberOfSymbols)
        let row = index / symbolsPerRow
        let column = index % symbolsPerRow
        let rect = CGRect(
            x: CGFloat(column) * symbolWidth,
            y: CGFloat(row) * symbolWidth,
            width: symbolWidth, height: symbolWidth)
        return rect.insetBy(dx: symbolWidth * insetProportion, dy: symbolWidth * insetProportion)
    }
}

extension CGRect {
    /// Returns a rect with the aspect ratio of `otherSize`, fitting within `self`.
    func fit(_ otherSize: CGSize) -> CGRect {
        let scale = min(size.width / otherSize.width, size.height / otherSize.height)
        let newSize = CGSize(width: otherSize.width * scale, height: otherSize.height * scale)
        let newOrigin = CGPoint(x: midX - newSize.width/2, y: midY - newSize.height/2)
        return CGRect(origin: newOrigin, size: newSize)
    }
}
```

canvas with gesture

Update frame/opacity based on that.  Can drag around and every symbol updates.

```swift
struct ContentView: View {
    let symbols = Array(repeating: Symbol("swift"), count: 3166)
    @GestureState private var focalPoint: CGPoint? = nil

    var body: some View {
        Canvas { context, size in
            let metrics = gridMetrics(in: size)
            for (index, symbol) in symbols.enumerated() {
                let rect = metrics[index]
                let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint)

                context.opacity = opacity
                let image = context.resolve(symbol.image)
                context.draw(image, in: sRect.fit(image.size))
            }
        }
        .gesture(DragGesture(minimumDistance: 0).updating($focalPoint) { value, focalPoint, _ in
            focalPoint = value.location
        })
    }

    func gridMetrics(in size: CGSize) -> SymbolGridMetrics {
        SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}

struct SymbolGridMetrics {
    let symbolWidth: CGFloat
    let symbolsPerRow: Int
    let numberOfSymbols: Int
    let insetProportion: CGFloat

    init(size: CGSize, numberOfSymbols: Int, insetProportion: CGFloat = 0.1) {
        let areaPerSymbol = (size.width * size.height) / CGFloat(numberOfSymbols)
        self.symbolsPerRow = Int(size.width / sqrt(areaPerSymbol))
        self.symbolWidth = size.width / CGFloat(symbolsPerRow)
        self.numberOfSymbols = numberOfSymbols
        self.insetProportion = insetProportion
    }

    /// Returns the frame in the grid for the symbol at `index` position.
    /// It is not valid to pass an index less than `0` or larger than the number of symbols the grid metrics was created for.
    subscript(_ index: Int) -> CGRect {
        precondition(index >= 0 && index < numberOfSymbols)
        let row = index / symbolsPerRow
        let column = index % symbolsPerRow
        let rect = CGRect(
            x: CGFloat(column) * symbolWidth,
            y: CGFloat(row) * symbolWidth,
            width: symbolWidth, height: symbolWidth)
        return rect.insetBy(dx: symbolWidth * insetProportion, dy: symbolWidth * insetProportion)
    }
}

extension CGRect {
    /// Returns a rect with the aspect ratio of `otherSize`, fitting within `self`.
    func fit(_ otherSize: CGSize) -> CGRect {
        let scale = min(size.width / otherSize.width, size.height / otherSize.height)
        let newSize = CGSize(width: otherSize.width * scale, height: otherSize.height * scale)
        let newOrigin = CGPoint(x: midX - newSize.width/2, y: midY - newSize.height/2)
        return CGRect(origin: newOrigin, size: newSize)
    }

    /// Returns a transformed rect and relative opacity based on a fish eye effect centered around `point`.
    /// The rectangles closer to the center of that point will be larger and brighter, and those further away will be smaller, up to a distance of `radius`.
    func fishEyeTransform(around point: CGPoint?, radius: CGFloat = 300, zoom: CGFloat = 1.0) -> (frame: CGRect, opacity: CGFloat) {
        guard let point = point else {
            return (self, 1.0)
        }

        let deltaX = midX - point.x
        let deltaY = midY - point.y
        let distance = sqrt(deltaX*deltaX + deltaY*deltaY)
        let theta = atan2(deltaY, deltaX)

        let scaledClampedDistance = pow(min(1, max(0, distance/radius)), 0.7)
        let scale = (1.0 - scaledClampedDistance)*zoom + 0.5

        let newOffset = distance * (2.0 - scaledClampedDistance)*sqrt(zoom)
        let newDeltaX = newOffset * cos(theta)
        let newDeltaY = newOffset * sin(theta)

        let newSize = CGSize(width: size.width * scale, height: size.height * scale)
        let newOrigin = CGPoint(x: (newDeltaX + point.x) - newSize.width/2, y: (newDeltaY + point.y) - newSize.height/2)

        // Clamp the opacity to be 0.1 at the lowest
        let opacity = max(0.1, 1.0 - scaledClampedDistance)
        return (CGRect(origin: newOrigin, size: newSize), opacity)
    }
}
```

canvas with accessiblity children.  Can refine how it comes across through accessibility features.

This modifier isn't restricted to canvas, but any view.



```swift
struct ContentView: View {
    let symbols = Array(repeating: Symbol("swift"), count: 3166)
    @GestureState private var focalPoint: CGPoint? = nil

    var body: some View {
        Canvas { context, size in
            let metrics = gridMetrics(in: size)
            for (index, symbol) in symbols.enumerated() {
                let rect = metrics[index]
                let (sRect, opacity) = rect.fishEyeTransform(around: focalPoint)

                context.opacity = opacity
                let image = context.resolve(symbol.image)
                context.draw(image, in: sRect.fit(image.size))
            }
        }
        .gesture(DragGesture(minimumDistance: 0).updating($focalPoint) { value, focalPoint, _ in
            focalPoint = value.location
        })
        .accessibilityLabel("Symbol Browser")
        .accessibilityChildren {
            List(symbols) {
                Text($0.name)
            }
        }
    }

    func gridMetrics(in size: CGSize) -> SymbolGridMetrics {
        SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}

struct SymbolGridMetrics {
    let symbolWidth: CGFloat
    let symbolsPerRow: Int
    let numberOfSymbols: Int
    let insetProportion: CGFloat

    init(size: CGSize, numberOfSymbols: Int, insetProportion: CGFloat = 0.1) {
        let areaPerSymbol = (size.width * size.height) / CGFloat(numberOfSymbols)
        self.symbolsPerRow = Int(size.width / sqrt(areaPerSymbol))
        self.symbolWidth = size.width / CGFloat(symbolsPerRow)
        self.numberOfSymbols = numberOfSymbols
        self.insetProportion = insetProportion
    }

    /// Returns the frame in the grid for the symbol at `index` position.
    /// It is not valid to pass an index less than `0` or larger than the number of symbols the grid metrics was created for.
    subscript(_ index: Int) -> CGRect {
        precondition(index >= 0 && index < numberOfSymbols)
        let row = index / symbolsPerRow
        let column = index % symbolsPerRow
        let rect = CGRect(
            x: CGFloat(column) * symbolWidth,
            y: CGFloat(row) * symbolWidth,
            width: symbolWidth, height: symbolWidth)
        return rect.insetBy(dx: symbolWidth * insetProportion, dy: symbolWidth * insetProportion)
    }
}

extension CGRect {
    /// Returns a rect with the aspect ratio of `otherSize`, fitting within `self`.
    func fit(_ otherSize: CGSize) -> CGRect {
        let scale = min(size.width / otherSize.width, size.height / otherSize.height)
        let newSize = CGSize(width: otherSize.width * scale, height: otherSize.height * scale)
        let newOrigin = CGPoint(x: midX - newSize.width/2, y: midY - newSize.height/2)
        return CGRect(origin: newOrigin, size: newSize)
    }

    /// Returns a transformed rect and relative opacity based on a fish eye effect centered around `point`.
    /// The rectangles closer to the center of that point will be larger and brighter, and those further away will be smaller, up to a distance of `radius`.
    func fishEyeTransform(around point: CGPoint?, radius: CGFloat = 300, zoom: CGFloat = 1.0) -> (frame: CGRect, opacity: CGFloat) {
        guard let point = point else {
            return (self, 1.0)
        }

        let deltaX = midX - point.x
        let deltaY = midY - point.y
        let distance = sqrt(deltaX*deltaX + deltaY*deltaY)
        let theta = atan2(deltaY, deltaX)

        let scaledClampedDistance = pow(min(1, max(0, distance/radius)), 0.7)
        let scale = (1.0 - scaledClampedDistance)*zoom + 0.5

        let newOffset = distance * (2.0 - scaledClampedDistance)*sqrt(zoom)
        let newDeltaX = newOffset * cos(theta)
        let newDeltaY = newOffset * sin(theta)

        let newSize = CGSize(width: size.width * scale, height: size.height * scale)
        let newOrigin = CGPoint(x: (newDeltaX + point.x) - newSize.width/2, y: (newDeltaY + point.y) - newSize.height/2)

        // Clamp the opacity to be 0.1 at the lowest
        let opacity = max(0.1, 1.0 - scaledClampedDistance)
        return (CGRect(origin: newOrigin, size: newSize), opacity)
    }
}
```

Canvas with timeline view.  LIke a screensaver.  Timeline view is created with a schedule (e.g. animation schedule)

Use that time to update the focal point in the transform, creating our screensaver.



```swift
struct ContentView: View {
    let symbols = Array(repeating: Symbol("swift"), count: 3166)

    var body: some View {
        TimelineView(.animation) {
           let time = $0.date.timeIntervalSince1970
           Canvas { context, size in
              let metrics = gridMetrics(in: size)
              let focalPoint = focalPoint(at: time, in: size)
              for (index, symbol) in symbols.enumerated() {
                let rect = metrics[index]
                let (sRect, opacity) = rect.fishEyeTransform(
                   around: focalPoint, at: time)

                 context.opacity = opacity
                 let image = context.resolve(symbol.image)
                 context.draw(image, in: sRect.fit(image.size))
              }
           }
        }
    }

    func gridMetrics(in size: CGSize) -> SymbolGridMetrics {
        SymbolGridMetrics(size: size, numberOfSymbols: symbols.count)
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}

struct SymbolGridMetrics {
    let symbolWidth: CGFloat
    let symbolsPerRow: Int
    let numberOfSymbols: Int
    let insetProportion: CGFloat

    init(size: CGSize, numberOfSymbols: Int, insetProportion: CGFloat = 0.1) {
        let areaPerSymbol = (size.width * size.height) / CGFloat(numberOfSymbols)
        self.symbolsPerRow = Int(size.width / sqrt(areaPerSymbol))
        self.symbolWidth = size.width / CGFloat(symbolsPerRow)
        self.numberOfSymbols = numberOfSymbols
        self.insetProportion = insetProportion
    }

    /// Returns the frame in the grid for the symbol at `index` position.
    /// It is not valid to pass an index less than `0` or larger than the number of symbols the grid metrics was created for.
    subscript(_ index: Int) -> CGRect {
        precondition(index >= 0 && index < numberOfSymbols)
        let row = index / symbolsPerRow
        let column = index % symbolsPerRow
        let rect = CGRect(
            x: CGFloat(column) * symbolWidth,
            y: CGFloat(row) * symbolWidth,
            width: symbolWidth, height: symbolWidth)
        return rect.insetBy(dx: symbolWidth * insetProportion, dy: symbolWidth * insetProportion)
    }
}

extension CGRect {
    /// Returns a rect with the aspect ratio of `otherSize`, fitting within `self`.
    func fit(_ otherSize: CGSize) -> CGRect {
        let scale = min(size.width / otherSize.width, size.height / otherSize.height)
        let newSize = CGSize(width: otherSize.width * scale, height: otherSize.height * scale)
        let newOrigin = CGPoint(x: midX - newSize.width/2, y: midY - newSize.height/2)
        return CGRect(origin: newOrigin, size: newSize)
    }

    /// Returns a transformed rect and relative opacity based on a fish eye effect centered around `point`.
    /// The rectangles closer to the center of that point will be larger and brighter, and those further away will be smaller, up to a distance of `radius`.
    func fishEyeTransform(around point: CGPoint?, radius: CGFloat = 200, zoom: CGFloat = 3.0) -> (frame: CGRect, opacity: CGFloat) {
        guard let point = point else {
            return (self, 1.0)
        }

        let deltaX = midX - point.x
        let deltaY = midY - point.y
        let distance = sqrt(deltaX*deltaX + deltaY*deltaY)
        let theta = atan2(deltaY, deltaX)

        let scaledClampedDistance = pow(min(1, max(0, distance/radius)), 0.7)
        let scale = (1.0 - scaledClampedDistance)*zoom + 0.5

        let newOffset = distance * (2.0 - scaledClampedDistance)*sqrt(zoom)
        let newDeltaX = newOffset * cos(theta)
        let newDeltaY = newOffset * sin(theta)

        let newSize = CGSize(width: size.width * scale, height: size.height * scale)
        let newOrigin = CGPoint(x: (newDeltaX + point.x) - newSize.width/2, y: (newDeltaY + point.y) - newSize.height/2)

        // Clamp the opacity to be 0.1 at the lowest
        let opacity = max(0.1, 1.0 - scaledClampedDistance)
        return (CGRect(origin: newOrigin, size: newSize), opacity)
    }

    /// Returns a transformed rect and relative opacity based on a fish eye effect centered around `point`, based on a preset path indexed using `time`.
    func fishEyeTransform(around point: CGPoint, at time: TimeInterval) -> (frame: CGRect, opacity: CGFloat) {
        // Arbitrary zoom and radius calculation based on time
        let zoom = cos(time) + 3.0
        let radius = ((cos(time/5) + 1)/2) * 150 + 150
        return fishEyeTransform(around: point, radius: radius, zoom: zoom)
    }
}

/// Returns a focal point within `size` based on a preset path, indexed using `time`.
func focalPoint(at time: TimeInterval, in size: CGSize) -> CGPoint {
   let offset: CGFloat = min(size.width, size.height)/4
   let distance = ((sin(time/5) + 1)/2) * offset + offset
   let scalePoint = CGPoint(x: size.width / 2 + distance * cos(time / 2), y: size.height / 2 + distance * sin(time / 2))
   return scalePoint
}
```

In watchOS your app dims by default, and you have more control.  Tiemline view can preload the time of your views at future dates.  As we move into the future, they will be displayed on screen without foregrounding your app.

Timeline schedule `TimelineSchedule.everyMinute`.  Several other kinds of schedules.  periodic, explicit, animation, `CustomSchedule`.

## Privacy sensitive

```swift
Button {
    showFavoritePicker = true
} label: {
    VStack(alignment: .center) {
        Text("Favorite Symbol")
            .foregroundStyle(.secondary)
        Image(systemName: favoriteSymbol)
            .font(.title2)
            .privacySensitive(true)
   }
}
.tint(.purple)
```

By adding this modifier, it's redacted when entering the state

[[what's new in watchOS 8]]

Privacy sensitive (widgets)

```swift
VStack(alignment: .leading) {
    Text("Favorite Symbol")
        .textCase(.uppercase)
        .font(.caption.bold())

    ContainerRelativeShape()
        .fill(.quaternary)
        .overlay {
            Image(systemName: favoriteSymbol)
                .font(.system(size: 40))
                .privacySensitive(true)
        }
}
```

Added to lcokscreen will sue this to hide sensitive information when locked

[[Principles of great widgets]]

## materials
Create beautiful visual effects that emphasize their content.

ADding color/materials to symbol browser
material-backed overlay.



```swift
struct ColorList: View {
    let symbols = Array(repeating: Symbol("swift"), count: 3166)

    var body: some View {
        ZStack {
            gradientBackground
            materialOverlay
        }
    }

    var materialOverlay: some View {
        VStack {
           Text("Symbol Browser")
              .font(.largeTitle.bold())
           Text("\(symbols.count) symbols 🎉")
              .foregroundStyle(.secondary)
              .font(.title2.bold())
        }
        .padding()
        .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16.0))
    }

    var gradientBackground: some View {
        LinearGradient(
            gradient: Gradient(colors: [.red, .orange, .yellow, .green, .blue, .indigo, .purple]),
            startPoint: .leading, endPoint: .trailing)
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}
```

`.ultrathinmaterial` Emojis are excluded so they look as they should.

System context now has vibrancy as well.



## safe area inset
Place content on top of scrollable view and still have content position start and end as expected
```swift
struct ContentView: View {
    let newSymbols = Array(repeating: Symbol("swift"), count: 645)
    let systemColors: [Color] = [.red, .orange, .yellow, .green, .mint, .teal, .cyan, .blue, .indigo, .purple, .pink, .gray, .brown]

    var body: some View {
        ScrollView {
            symbolGrid
        }
        .safeAreaInset(edge: .bottom, spacing: 0) {
            VStack(spacing: 0) {
                Divider()
                VStack(spacing: 0) {
                    Text("\(newSymbols.count) new symbols")
                        .foregroundStyle(.primary)
                        .font(.body.bold())
                    Text("\(systemColors.count) system colors")
                        .foregroundStyle(.secondary)
                }
                .padding()
            }
            .background(.regularMaterial)
        }
    }

    var symbolGrid: some View {
        LazyVGrid(columns: [.init(.adaptive(minimum: 40, maximum: 40))]) {
            ForEach(0 ..< newSymbols.count, id: \.self) { index in
                newSymbols[index].image
                    .foregroundStyle(.white)
                    .frame(width: 40, height: 40)
                    .background(systemColors[index % systemColors.count])
            }
        }
        .padding()
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}
```

[[Add rich graphics to your SwiftUI app]]


## preview orientation
Use landscape for previews

```swift
struct ColorList_Previews: PreviewProvider {
    static var previews: some View {
        ColorList()
            .previewInterfaceOrientation(.portrait)

        ColorList()
            .previewInterfaceOrientation(.landscapeLeft)
    }
}

struct ColorList: View {
    let newSymbols = Array(repeating: Symbol("swift"), count: 645)
    let systemColors: [Color] = [.red, .orange, .yellow, .green, .mint, .teal, .cyan, .blue, .indigo, .purple, .pink, .gray, .brown]

    var body: some View {
        ScrollView {
            symbolGrid
        }
        .safeAreaInset(edge: .bottom, spacing: 0) {
            VStack(spacing: 0) {
                Divider()
                VStack(spacing: 0) {
                    Text("\(newSymbols.count) new symbols")
                        .foregroundStyle(.primary)
                        .font(.body.bold())
                    Text("\(systemColors.count) system colors")
                        .foregroundStyle(.secondary)
                }
                .padding()
            }
            .background(.regularMaterial)
        }
    }

    var symbolGrid: some View {
        LazyVGrid(columns: [.init(.adaptive(minimum: 40, maximum: 40))]) {
            ForEach(0 ..< newSymbols.count, id: \.self) { index in
                newSymbols[index].image
                    .foregroundStyle(.white)
                    .frame(width: 40, height: 40)
                    .background(systemColors[index % systemColors.count])
            }
        }
        .padding()
    }
}

struct Symbol: Identifiable {
    let name: String
    init(_ name: String) { self.name = name }

    var image: Image { Image(systemName: name) }
    var id: String { name }
}
```

Property navigator has accessiblity modifiers.

New accessibility preview tab will show textual representation of accessibility elements.  Same information that powes accessibility features in a familiar format

[[SwiftUI accessibility beyond the basics]]

# Text and keyboard
## Markdown
hello world

```swift
Text("Hello, World!")
```

markdown text: strong emphasis

```swift
Text("**Hello**, World!")
```

markdown text: links

```swift
Text("**Hello**, World!")
Text("""
Have a *happy* [WWDC](https://developer.apple.com/wwdc21/)!
""")
```

markdown text: inline code

```swift
Text("""
Is this *too* meta?

`Text("**Hello**, World!")`
`Text(\"\"\"`
`Have a *happy* [WWDC](https://developer.apple.com/wwdc21/)!`
`\"\"\")`
""")
```

Attributed string

[[What's new in Foundation]]
```swift
struct ContentView: View {
    var body: some View {
        Text(formattedDate)
    }

    var formattedDate: AttributedString {
        var formattedDate: AttributedString = Date().formatted(Date.FormatStyle().day().month(.wide).weekday(.wide).attributed)

        let weekday = AttributeContainer.dateField(.weekday)
        let color = AttributeContainer.foregroundColor(.orange)
        formattedDate.replaceAttributes(weekday, with: color)

        return formattedDate
    }
}
```

Also localizes content.  Also true of markdown support.

XCode13 now uses swift compiler to generate strings and localization catalogs.

[[Localize your SwiftUI App]]

## Dynamic type
Supported this forever, and has a new year to restrict the range of type sizes `.dynnamicTypeSize(large ... extraLarge ?)`

Headers stay the same size at the small size.  

While macOS doesn't support dynamic type, it doessupport selectable text.


## text selection

Here, we support selectable header text.

Text can now be copied on iOS/macOS
```swift
struct ContentView: View {
    var activity: Activity = .sample

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            ActivityHeader(activity)
            Divider()
            Text(activity.info)
                .textSelection(.enabled)
                .padding()
            Spacer()
        }
        .background()
        .navigationTitle(activity.name)
    }
}

struct ActivityHeader: View {
    var activity: Activity
    init(_ activity: Activity) { self.activity = activity }

    var body: some View {
        VStack(alignment: alignment.horizontal, spacing: 8) {
            HStack(alignment: .firstTextBaseline) {
#if os(macOS)
                Text(activity.name)
                    .font(.title2.bold())
                Spacer()
#endif
                Text(activity.date.formatted(.dateTime.weekday(.wide).day().month().hour().minute()))
                    .foregroundStyle(.secondary)
            }
            HStack(alignment: .firstTextBaseline) {
                Image(systemName: "person.2")
                Text(activity.people.map(\.nameComponents).formatted(.list(memberStyle: .name(style: .short), type: .and)))
            }
        }
#if os(macOS)
        .padding()
#else
        .padding([.horizontal, .bottom])
#endif
        .frame(maxWidth: .infinity, alignment: alignment)
        .background(activity.tint.opacity(0.1).ignoresSafeArea())
    }

    private var alignment: Alignment {
#if os(macOS)
        .leading
#else
        .center
#endif
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")
}

struct Person {
    var givenName: String
    var familyName: String = ""

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

text selection: view hierarchy

```swift
struct ContentView: View {
    var activity: Activity = .sample

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            ActivityHeader(activity)
            Divider()
            Text(activity.info)
                .padding()
            Spacer()
        }
        .textSelection(.enabled)
        .background()
        .navigationTitle(activity.name)
    }
}

struct ActivityHeader: View {
    var activity: Activity
    init(_ activity: Activity) { self.activity = activity }

    var body: some View {
        VStack(alignment: alignment.horizontal, spacing: 8) {
            HStack(alignment: .firstTextBaseline) {
#if os(macOS)
                Text(activity.name)
                    .font(.title2.bold())
                Spacer()
#endif
                Text(activity.date.formatted(.dateTime.weekday(.wide).day().month().hour().minute()))
                    .foregroundStyle(.secondary)
            }
            HStack(alignment: .firstTextBaseline) {
                Image(systemName: "person.2")
                Text(activity.people.map(\.nameComponents).formatted(.list(memberStyle: .name(style: .short), type: .and)))
            }
        }
#if os(macOS)
        .padding()
#else
        .padding([.horizontal, .bottom])
#endif
        .frame(maxWidth: .infinity, alignment: alignment)
        .background(activity.tint.opacity(0.1).ignoresSafeArea())
    }

    private var alignment: Alignment {
#if os(macOS)
        .leading
#else
        .center
#endif
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")
}

struct Person {
    var givenName: String
    var familyName: String = ""

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

## text formatting

list

`people.map(\.namecomponents).formatted(.list(...))`

Altogether, creatin ga performant and type-safe expression that handles any number of people.

```swift
struct ContentView: View {
    var activity: Activity = .sample

    var body: some View {
        Text(activity.people.map(\.nameComponents).formatted(.list(memberStyle: .name(style: .short), type: .and)))
            .scenePadding()
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")
}

struct Person {
    var givenName: String
    var familyName: String = ""

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

text field formatting

```swift
struct ContentView: View {
    @State private var newAttendee = PersonNameComponents()

    var body: some View {
       TextField("New Person", value: $newAttendee,
          format: .name(style: .medium))
    }
}
```

text field prompts and labels

```swift
struct ContentView: View {
    @State var activity: Activity = .sample

    var body: some View {
        Form {
            TextField("Name:", text: $activity.name, prompt: Text("New Activity"))
            TextField("Location:", text: $activity.location)
            DatePicker("Date:", selection: $activity.date)
        }
        .frame(minWidth: 250)
        .padding()
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")
}

struct Person {
    var givenName: String
    var familyName: String = ""

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

[[What's new in Foundation]]

Prompt is used as placeholder content on macOS.



# keyboards

## `onSubmit`

Supplementary actions for e.g. return key.

Applied to entire form of controls.

Can add a `.submitLabel` modifier.  On soft keyboards, will be used as the label for return keys.


```swift
struct ContentView: View {
    @State private var activity: Activity = .sample
    @State private var newAttendee = PersonNameComponents()

    var body: some View {
        TextField("New Person", value: $newAttendee,
            format: .name(style: .medium)
        )
        .onSubmit {
            activity.append(Person(newAttendee))
            newAttendee = PersonNameComponents()
        }
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

text field submission: submit label

```swift
struct ContentView: View {
    @State private var activity: Activity = .sample
    @State private var newAttendee = PersonNameComponents()

    var body: some View {
        TextField("New Person", value: $newAttendee,
            format: .name(style: .medium)
        )
        .onSubmit {
            activity.append(Person(newAttendee))
            newAttendee = PersonNameComponents()
        }
        .submitLabel(.done)
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

keyboard toolbar

Shown as an accessory view on iOS, or in touchbar on macOS.

```swift
struct ContentView: View {
    @State private var activity: Activity = .sample
    @FocusState private var focusedField: Field?

    var body: some View {
        Form {
            TextField("Name", text: $activity.name, prompt: Text("New Activity"))
            TextField("Location", text: $activity.location)
            DatePicker("Date", selection: $activity.date)
        }
        .toolbar {
            ToolbarItemGroup(placement: .keyboard) {
                Button(action: selectPreviousField) {
                    Label("Previous", systemImage: "chevron.up")
                }
                .disabled(!hasPreviousField)

                Button(action: selectNextField) {
                    Label("Next", systemImage: "chevron.down")
                }
                .disabled(!hasNextField)
            }
        }
    }

    private func selectPreviousField() {
       focusedField = focusedField.map {
          Field(rawValue: $0.rawValue - 1)!
       }
    }

    private var hasPreviousField: Bool {
        if let currentFocusedField = focusedField {
            return currentFocusedField.rawValue > 0
        } else {
            return false
        }
    }

    private func selectNextField() {
       focusedField = focusedField.map {
          Field(rawValue: $0.rawValue + 1)!
       }
    }

    private var hasNextField: Bool {
        if let currentFocusedField = focusedField {
            return currentFocusedField.rawValue < Field.allCases.count
        } else {
            return false
        }
    }
}

private enum Field: Int, Hashable, CaseIterable {
   case name, location, date, addAttendee
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

Keyboards also help us navigate

## Focus

Sometimes we can create refinements in focus state.

New tool `@FocusState`.  Reflects the state of focus and provides precise control.

Can reflect a boolean value tied to focusable view.
```swift
struct ContentView: View {
    @State private var activity: Activity = .sample
    @State private var newAttendee = PersonNameComponents()
    @FocusState private var addAttendeeIsFocused: Bool

    var body: some View {
        VStack {
            Form {
                TextField("Name:", text: $activity.name, prompt: Text("New Activity"))
                TextField("Location:", text: $activity.location)
                DatePicker("Date:", selection: $activity.date)
            }

            TextField("New Person", value: $newAttendee, format: .name(style: .medium))
                .focused($addAttendeeIsFocused)
        }
        .frame(minWidth: 250)
        .scenePadding()
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

Value also written to, to control focus.

focus state: setting focus

```swift
struct ContentView: View {
    @State private var activity: Activity = .sample
    @State private var newAttendee = PersonNameComponents()
    @FocusState private var addAttendeeIsFocused: Bool

    var body: some View {
        VStack {
            Form {
                TextField("Name:", text: $activity.name, prompt: Text("New Activity"))
                TextField("Location:", text: $activity.location)
                DatePicker("Date:", selection: $activity.date)
            }

            VStack(alignment: .leading) {
                TextField("New Person", value: $newAttendee, format: .name(style: .medium))
                    .focused($addAttendeeIsFocused)

                ControlGroup {
                    Button {
                        addAttendeeIsFocused = true
                    } label: {
                       Label("Add Attendee", systemImage: "plus")
                    }
                }
                .fixedSize()
            }
        }
        .frame(minWidth: 250)
        .scenePadding()
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

focus state: hashable value
THIS IS MY FINAL FORM
Increased flexibility


```swift
private enum Field: Int, Hashable, CaseIterable {
   case name, location, date, addAttendee
}

struct ContentView: View {
    @State private var activity: Activity = .sample
    @State private var newAttendee = PersonNameComponents()
    @FocusState private var focusedField: Field?

    var body: some View {
        VStack {
            Form {
                TextField("Name:", text: $activity.name, prompt: Text("New Activity"))
                    .focused($focusedField, equals: .name)
                TextField("Location:", text: $activity.location)
                    .focused($focusedField, equals: .location)
                DatePicker("Date:", selection: $activity.date)
                    .focused($focusedField, equals: .date)
            }

            VStack(alignment: .leading) {
                TextField("New Person", value: $newAttendee, format: .name(style: .medium))
                    .focused($focusedField, equals: .addAttendee)

                ControlGroup {
                    Button {
                        focusedField = .addAttendee
                    } label: {
                       Label("Add Attendee", systemImage: "plus")
                    }
                }
                .fixedSize()
            }
        }
        .frame(minWidth: 250)
        .scenePadding()
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

focus state: back/forward controls

Idea here is our accessory view lets us move focus up or down, and loops around at the end?

```swift
private enum Field: Int, Hashable, CaseIterable {
   case name, location, date, addAttendee
}

struct ContentView: View {
    @State private var activity: Activity = .sample
    @FocusState private var focusedField: Field?

    var body: some View {
        Form {
            TextField("Name", text: $activity.name, prompt: Text("New Activity"))
            TextField("Location", text: $activity.location)
            DatePicker("Date", selection: $activity.date)
        }
        .toolbar {
            ToolbarItemGroup(placement: .keyboard) {
                Button(action: selectPreviousField) {
                    Label("Previous", systemImage: "chevron.up")
                }
                .disabled(!canSelectPreviousField)

                Button(action: selectNextField) {
                    Label("Next", systemImage: "chevron.down")
                }
                .disabled(!canSelectNextField)
            }
        }
    }

    private func selectPreviousField() {
       focusedField = focusedField.map {
          Field(rawValue: $0.rawValue - 1)!
       }
    }

    private var canSelectPreviousField: Bool {
        if let currentFocusedField = focusedField {
            return currentFocusedField.rawValue > 0
        } else {
            return false
        }
    }

    private func selectNextField() {
       focusedField = focusedField.map {
          Field(rawValue: $0.rawValue + 1)!
       }
    }

    private var canSelectNextField: Bool {
        if let currentFocusedField = focusedField {
            return currentFocusedField.rawValue < Field.allCases.count
        } else {
            return false
        }
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

focus state: keyboard dismissal

Great way for iOS apps to dismiss by clearing the focus value

[[Direct and reflect focus in SwiftUI]]
```swift
private enum Field: Int, Hashable, CaseIterable {
   case name, location, date, addAttendee
}

struct ContentView: View {
    @State private var activity: Activity = .sample
    @FocusState private var focusedField: Field?

    var body: some View {
        Form {
            TextField("Name", text: $activity.name, prompt: Text("New Activity"))
            TextField("Location", text: $activity.location)
            DatePicker("Date", selection: $activity.date)
        }
    }

    func endEditing() {
        focusedField = nil
    }
}

struct Activity {
    var name: String
    var date: Date
    var location: String
    var people: [Person]
    var info: AttributedString
    var tint: Color = .purple

    static let sample = Activity(name: "What's New in SwiftUI", date: Date(), location: "Apple Park", people: [.init(givenName: "You")], info: "This is some info.")

    mutating func append(_ person: Person) {
        people.append(person)
    }
}

struct Person {
    var givenName: String
    var familyName: String

    init(givenName: String, familyName: String = "") {
        self.givenName = givenName
        self.familyName = familyName
    }

    init(_ nameComponents: PersonNameComponents) {
        givenName = nameComponents.givenName ?? ""
        familyName = nameComponents.familyName ?? ""
    }

    var nameComponents: PersonNameComponents {
        get {
            var components = PersonNameComponents()
            components.givenName = givenName
            if !familyName.isEmpty {
                components.familyName = familyName
            }
            return components
        }
        set {
            givenName = newValue.givenName ?? ""
            familyName = newValue.familyName ?? ""
        }
    }
}
```

# More buttons
In SwiftUI, buttons are used for many things.  e.g. swipe actions are composed out of buttons.

## bordered buttons

```swift
Button("Add") {
   // ... 
}
.buttonStyle(.bordered)
```

Like other modifiers, add this to a group of controls

bordered buttons: view hierarchy

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<10) { _ in
                    Button("Add") {
                        //...
                    }
                }
            }
        }
        .buttonStyle(.bordered)
    }
}
```

bordered buttons: tinting

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<10) { _ in
                    Button("Add") {
                        //...
                    }
                }
            }
        }
        .buttonStyle(.bordered)
        .tint(.green)
    }
}
```

Control size and prominence

```swift
struct ContentView: View {
    var entry: ButtonEntry = .sample

    var body: some View {
        HStack {
            ForEach(entry.tags) { tag in
                Button(tag.name) {
                    // ...
                }
                .tint(tag.color)
            }
        }
        .buttonStyle(.bordered)
        .controlSize(.small)
        .controlProminence(.increased)
    }
}

struct ButtonEntry {
    struct Tag: Identifiable {
        var name: String
        var color: Color
        var id: String { name }
    }

    var name: String
    var tags: [Tag]

    static let sample = ButtonEntry(name: "Stroopwafel", tags: [Tag(name: "1960s", color: .purple), Tag(name: "bronze", color: .yellow)])
}
```

large buttons
Now built-in to swiftui
By specifying large size, you get "beautiful rounded-rectangle button"
Most important one has increased prominence, giving it a high-contrast accent color.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Button(action: addToJar) {
                Text("Add to Jar").frame(maxWidth: 300)
            }
            .controlProminence(.increased)
            .keyboardShortcut(.defaultAction)

            Button(action: addToWatchlist) {
                Text("Add to Watchlist").frame(maxWidth: 300)
            }
            .tint(.accentColor)
        }
        .buttonStyle(.bordered)
        .controlSize(.large)
    }

    private func addToJar() {}
    private func addToWatchlist() {}
}
```
Give textLabel a max width so they don't get too large

One new addition is adding increased prominent tint support.

Non-prominent buttons do not display any tint, since chrome indicates interactivity on macOS.

Be careful adding too many prominent buttons onscreen, it's overwhelming.  Reserve for singular, primary actions.

Lower-prominence tint is a great alternative.
Automatically have press/disabled, dark mode, and dynamic type
Give consistency between apps

destructive buttons

```swift
struct ContentView: View {
    var entry: ButtonEntry = .sample

    var body: some View {
        ButtonEntryCell(entry)
            .contextMenu {
                Section {
                    Button("Open") {
                        // ...
                    }
                    Button("Delete...", role: .destructive) {
                        // ...
                    }
                }

                Section {
                    Button("Archive") {}

                    Menu("Move to") {
                        ForEach(Jar.allJars) { jar in
                            Button("\(jar.name)") {
                                //addTo(jar)
                            }
                        }
                    }
                }
            }
    }
}

struct ButtonEntryCell: View {
    var entry: ButtonEntry = .sample
    init(_ entry: ButtonEntry) { self.entry = entry }

    var body: some View {
        Text(entry.name)
            .padding()
    }
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }

    static let allJars = [Jar(name: "Secret Stash")]
}

struct ButtonEntry {
    struct Tag: Identifiable {
        var name: String
        var color: Color
        var id: String { name }
    }

    var name: String
    var tags: [Tag]

    static let sample = ButtonEntry(name: "Stroopwafel", tags: [Tag(name: "1960s", color: .purple), Tag(name: "bronze", color: .yellow)])
}
```

confirmation dialogs

```swift
struct ContentView: View {
    var entry: ButtonEntry = .sample
    @State private var showConfirmation: Bool = false

    var body: some View {
        ButtonEntryCell(entry)
            .contextMenu {
                Section {
                    Button("Open") {
                        // ...
                    }
                    Button("Delete...", role: .destructive) {
                        showConfirmation = true
                        // ...
                    }
                }

                Section {
                    Button("Archive") {}

                    Menu("Move to") {
                        ForEach(Jar.allJars) { jar in
                            Button("\(jar.name)") {
                                //addTo(jar)
                            }
                        }
                    }
                }
            }
            .confirmationDialog(
                "Are you sure you want to delete \(entry.name)?",
                isPresented: $showConfirmation
            ) {
                Button("Delete", role: .destructive) {
                    // delete the entry
                }
            } message: {
                Text("Deleting \(entry.name) will remove it from all of your jars.")
            }
    }
}

struct ButtonEntryCell: View {
    var entry: ButtonEntry = .sample
    init(_ entry: ButtonEntry) { self.entry = entry }

    var body: some View {
        Text(entry.name)
            .padding()
    }
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }

    static let allJars = [Jar(name: "Secret Stash")]
}

struct ButtonEntry {
    struct Tag: Identifiable {
        var name: String
        var color: Color
        var id: String { name }
    }

    var name: String
    var tags: [Tag]

    static let sample = ButtonEntry(name: "Stroopwafel", tags: [Tag(name: "1960s", color: .purple), Tag(name: "bronze", color: .yellow)])
}
```

iOS: action sheet
ipadOS: popvoer
macOS: alert

menu buttons

```swift
struct ContentView: View {
    var buttonEntry: ButtonEntry = .sample
    @StateObject private var jarStore = JarStore()

    var body: some View {
        Menu("Add") {
           ForEach(jarStore.allJars) { jar in
              Button("Add to \(jar.name)") {
                 jarStore.add(buttonEntry, to: jar)
              }
           }
        }
        .menuStyle(BorderedButtonMenuStyle())
        .scenePadding()
    }
}

class JarStore: ObservableObject {
    var allJars: [Jar] = Jar.allJars
    func add(_ entry: ButtonEntry, to jar: Jar) {}
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }
    static let allJars = [Jar(name: "Secret Stash")]
}

struct ButtonEntry {
    var name: String
    static let sample = ButtonEntry(name: "Stroopwafel")
}
```

Menu buttons: hidden indicator
So they don't have as much prominence

```swift
struct ContentView: View {
    var buttonEntry: ButtonEntry = .sample
    @StateObject private var jarStore = JarStore()

    var body: some View {
        Menu("Add") {
           ForEach(jarStore.allJars) { jar in
              Button("Add to \(jar.name)") {
                 jarStore.add(buttonEntry, to: jar)
              }
           }
        }
        .menuStyle(BorderedButtonMenuStyle())
        .menuIndicator(.hidden)
        .scenePadding()
    }
}

class JarStore: ObservableObject {
    var allJars: [Jar] = Jar.allJars
    func add(_ entry: ButtonEntry, to jar: Jar) {}
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }
    static let allJars = [Jar(name: "Secret Stash")]
}

struct ButtonEntry {
    var name: String
    static let sample = ButtonEntry(name: "Stroopwafel")
}
```

menu buttons: primary action.  Best of both worlds.
Main part triggers the primary action, and the indicator presents the menu.



```swift
struct ContentView: View {
    var buttonEntry: ButtonEntry = .sample
    @StateObject private var jarStore = JarStore()

    var body: some View {
        Menu("Add") {
           ForEach(jarStore.allJars) { jar in
              Button("Add to \(jar.name)") {
                 jarStore.add(buttonEntry, to: jar)
              }
           }
        } primaryAction: {
            jarStore.addToDefaultJar(buttonEntry)
        }
        .menuStyle(BorderedButtonMenuStyle())
        .scenePadding()
    }
}

class JarStore: ObservableObject {
    var allJars: [Jar] = Jar.allJars
    func add(_ entry: ButtonEntry, to jar: Jar) {}
    func addToDefaultJar(_ entry: ButtonEntry) {}
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }
    static let allJars = [Jar(name: "Secret Stash")]
}


struct ButtonEntry {
    var name: String
    static let sample = ButtonEntry(name: "Stroopwafel")
}
```

menu buttons: primary action, indicator hidden

Here a click does the action, and a long-press shows the menu.  iOS as well.

```swift
struct ContentView: View {
    var buttonEntry: ButtonEntry = .sample
    @StateObject private var jarStore = JarStore()

    var body: some View {
        Menu("Add") {
           ForEach(jarStore.allJars) { jar in
              Button("Add to \(jar.name)") {
                 jarStore.add(buttonEntry, to: jar)
              }
           }
        } primaryAction: {
            jarStore.addToDefaultJar(buttonEntry)
        }
        .menuStyle(BorderedButtonMenuStyle())
        .menuIndicator(.hidden)
        .scenePadding()
    }
}

class JarStore: ObservableObject {
    var allJars: [Jar] = Jar.allJars
    func add(_ entry: ButtonEntry, to jar: Jar) {}
    func addToDefaultJar(_ entry: ButtonEntry) {}
}

struct Jar: Identifiable {
    var name: String
    var id: String { name }
    static let allJars = [Jar(name: "Secret Stash")]
}


struct ButtonEntry {
    var name: String
    static let sample = ButtonEntry(name: "Stroopwafel")
}
```

toggle buttons
Visually turns on and off.  

```swift
Toggle(isOn: $showOnlyNew) {
    Label("Show New Buttons", systemImage: "sparkles")
}
.toggleStyle(.button)
```

control group

On IOS, these are a little tighter spacing.  On macOS, there are visual affordances `|`.

```swift
ControlGroup {
    Button(action: archive) {
        Label("Archive", systemImage: "archiveBox")
    }
    Button(action: delete) {
        Label("Delete", systemName: "trash")
    }
}
```

control group: backward/forward control

```swift
struct ContentView: View {
    @State var current: String = "More buttons"
    @State var history: [String] = ["Text and keyboard", "Advanced graphics", "Beyond lists", "Better lists"]
    @State var forwardHistory: [String] = []

    var body: some View {
        Color.clear
            .toolbar{
                ToolbarItem(placement: .navigation) {
                    ControlGroup {
                        Menu {
                            ForEach(history, id: \.self) { previousSection in
                                Button(previousSection) {
                                    goBack(to: previousSection)
                                }
                            }
                        } label: {
                            Label("Back", systemImage: "chevron.backward")
                        } primaryAction: {
                            goBack(to: history[0])
                        }
                        .disabled(history.isEmpty)

                        Menu {
                            ForEach(forwardHistory, id: \.self) { nextSection in
                                Button(nextSection) {
                                    goForward(to: nextSection)
                                }
                            }
                        } label: {
                            Label("Forward", systemImage: "chevron.forward")
                        } primaryAction: {
                            goForward(to: forwardHistory[0])
                        }
                        .disabled(forwardHistory.isEmpty)
                    }
                    .controlGroupStyle(.navigation)
                }
            }
            .navigationTitle(current)
    }

    private func goBack(to section: String) {
        guard let index = history.firstIndex(of: section) else { return }
        forwardHistory.insert(current, at: 0)
        forwardHistory.insert(contentsOf: history[...history.index(before: index)].reversed(), at: 0)
        history.removeSubrange(...index)
        current = section
    }

    private func goForward(to section: String) {
        guard let index = forwardHistory.firstIndex(of: section) else { return }
        history.insert(current, at: 0)
        history.insert(contentsOf: forwardHistory[...forwardHistory.index(before: index)].reversed(), at: 0)
        forwardHistory.removeSubrange(...index)
        current = section
    }
}
```

A lot of flexibility has opened up on these controls.

# Wrap up
* Better lists
* beyond lists
* advanced graphics
* text and keyboard
* more buttons




