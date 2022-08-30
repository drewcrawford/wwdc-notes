Easy for your app to integrate with #applemusic #musickit 

Today I'd like tot alk about major enhancement
# Discovering catalog additions
curators, radio shows
search enhancements
top charts
new attributes

## Curators and radio shows
variety of attributes.  Name, url, artwork, kind, many more
kind: editorial vs external.  Apple or third party.
have playlists

radio show
Like curator type, they have a playlist relationship

both have reverse relationships.

## Catalog search enhancements
* support for new item types
* top results
* suggestions
```swift
// Loading catalog search top results

var searchRequest = MusicCatalogSearchRequest(
  term: "Hello",
  types: [
    Artist.self,
    Album.self,
    Song.self
  ]
)

let searchResponse = try await searchRequest.response()
print("\(searchResponse)")
```

```swift
// Loading catalog search top results

var searchRequest = MusicCatalogSearchRequest(
  term: "Hello",
  types: [
    Artist.self,
    Album.self,
    Song.self
  ]
)
//!!!!!!!!!!vvvv
searchRequest.includeTopResults = true
let searchResponse = try await searchRequest.response()
print("\(searchResponse.topResults)")
```


Mix of songs, artists, albums in a single collection ordered by relevancy.

```swift
// Loading suggestions

let request = MusicCatalogSearchSuggestionsRequest(term: "shaz")
let response = try await request.response()
print("\(response)")
```

Fetch the corresponding ersults by performing a search result with the search term.

## Catalog charts
* top charts
* city charts
* daily top 100
* filter for a specific genre

```swift
// Loading catalog top charts.

let request = MusicCatalogChartsRequest(
  kinds: [.dailyGlobalTop, .mostPlayed, .cityTop],
  types: [Song.self, Playlist.self]
)
let response = try await request.response()
print("\(response.playlistCharts.first)")
```

MusicKit has pagination support built in.

In 2021, we itnroduced new audio experiences, spatial audio, dolby atmos.

## Audio quality metadata
audio variants
* spatial audio with dolby atmos
* lossless audio

apple digital master

```swift
// Loading audio variants

let album = …
let detailedAlbum = try await album.with(.audioVariants)
print("\(detailedAlbum.debugDescription)")
```

Indicate in your UI the available audio resources.  Also exposing the *active* audio variant.  Visual indication of the quality of audio for currently playing item.  Music/player automatically chooses quality based on settings, etc.

```swift
// Showing currently playing audio variants

@ObservedObject var musicPlayerQueue = ApplicationMusicPlayer.shared.queue
@ObservedObject var musicPlayerState = ApplicationMusicPlayer.shared.state

var body: some View {
  if let currentEntry = musicPlayerQueue.currentEntry {
    VStack {
      MyPlayerEntryView(currentEntry)
      if musicPlayerState.audioVariant == .dolbyAtmos {
        Image("dolby-atmos-badge")
      }
    }
  }
}
```

Since playback state is an observed object, this automatically updates.

# Fetching personalized content
## Recently played items
* fetch music people already enjoy
* maintain history
```swift
// Loading recently played containers

let request = MusicRecentlyPlayedContainerRequest()
let response = try await request.response()
print("\(response)")
```

```swift
// Loading recently played songs

let request = MusicRecentlyPlayedRequest<Song>()
let response = try await request.response()
print("\(response)")
```


## Personal recommendations
based on library and listening history
organized by themes
```swift
// Loading personal recommendations

let request = MusicPersonalRecommendationsRequest()
let response = try await request.response()
print("\(response.recommendations.first)")
```

```swift
// Loading personal recommendations

let request = MusicPersonalRecommendationsRequest()
let response = try await request.response()
print("\(response.recommendations[1])")
```



# Exploring the library
Fetching items
* library requests
* library sectioned requests
* library search
* enhancements to load additonal properties

Playlist from the user's library.  Store in a local variable
```swift
@MainActor
private func loadLibraryPlaylists() async throws {
  let request = MusicLibraryRequest<Playlist>()
  let response = try await request.response()
  self.response = response
}
```

```swift
List {
  Section(header: Text("Library Playlists").fontWeight(.semibold)) {
    ForEach(response.items) { playlist in
      PlaylistCell(playlist)
    }
  }
}
```

## Library requests
Do not actually load data.  Load items from the copy stored on device.
Generic over type of item
Filter by specific properties
Built-in sorting options
Fetch already downloaded items
```swift
// Fetching all albums in the library

let request = MusicLibraryRequest<Album>()

let response = try await request.response()
print("\(response)")
```

Filtering
```swift
// Fetching all compilations in the library

var request = MusicLibraryRequest<Album>()
request.filter(matching: \.isCompilation, equalTo: true)

let response = try await request.response()
print("\(response)")
```

Chain multiple filters, giving you a more refined request with each addition.  

```swift
// Fetching all dance compilations in the library

var request = MusicLibraryRequest<Album>()
request.filter(matching: \.isCompilation, equalTo: true) 
request.filter(matching: \.genres, contains: danceGenre)

let response = try await request.response()
print("\(response)")
```

downloaded content only
```swift
// Fetching all downloaded dance compilations in the library

var request = MusicLibraryRequest<Album>()
request.filter(matching: \.isCompilation, equalTo: true) 
request.filter(matching: \.genres, contains: danceGenre)
request.includeDownloadedContentOnly = true

let response = try await request.response()
print("\(response)")
```

## Sectioned request
Fetch items grouped by sections
Generic over type of section and item
Similar to capabilities to library request

```swift
// Fetching all albums sectioned by genre

var request = MusicLibrarySectionedRequest<Genre, Album>()

let response = try await request.response()
print("\(response)")
```

sort response
```swift
// Fetching all albums sectioned by genre sorted by artist name

var request = MusicLibrarySectionedRequest<Genre, Album>()
request.sortItems(by: \.artistName, ascending: true)

let response = try await request.response()
print("\(response)")
```

## Library search
* `MusicLibrarySearchRequest`
* search user's library
* only requires a term and types

## additional properties
enhancement to the `with(...)` method
prefererred source parameter.
* catalog
* library

For properties that only live in one of the other, will be fetched regardless of preferred source.
Works with any item

```swift
// Fetching relationships using the with method

let album = …
let detailedAlbum = try await album.with(.tracks)
print("\(album.tracks)")
```

```swift
// Fetching relationships using the with method

let album = …
let detailedAlbum = try await album.with(.tracks, preferredSource: .library)
print("\(album.tracks)")
```
# Interacting with the library
Add to playlist.

```swift
Task { 
  try await MusicLibrary.shared(add: selectedTrack, to: playlist)
  isShowingPlaylistPicker = false
}
```


* add content to the library
* create playlists
* edit playlists (metadata, track list)

## adding to library
include items in the library tab of the apple music app
sync across all devices
keep users engaged in your app
ties in with library requests

## Playlist manipulation
* create multiple palylists
* add content to playlists
* edit playlists

# Wrap up
* catalog enhancements for your existing apps
* library functionality unlocking new capabilities
* different apps can benefit from using MusicKit

[[22/What's new in Swift]]
[[Meet MusicKit for Swift]]
[[Meet Apple Music API and MusicKit]]

* https://developer.apple.com/forums/tags/wwdc2022-110347
* https://developer.apple.com/forums/create/question?&tag1=173&tag2=536030
* https://developer.apple.com/documentation/musickit/explore_more_content_with_musickit
* https://developer.apple.com/documentation/musickit
