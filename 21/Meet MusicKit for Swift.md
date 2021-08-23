#musickit

* Expressive APIs for music items in swift
* Leverages new concurrency syntax
* Designed for use in SwiftUI
* Accelerates integration with apple music API
* Easier to build apps that tie into apple music

# Requesting music content
* Fetch content from apple music API
* Structured request
* Responses contain collections of items
* Support pagination

## Music items
* title: string
* isCompilation: bool
* artwork: Artwork

Relationships => artists, genres, tracks

e.g. tracks => MusicItemCollection\<Track\>.

Assocations: similar to relatoinships but more ephemeral or "editorially-driven".

```swift
// Loading and accessing relationships

let detailedAlbum = try await album.with([.artists, .tracks, .relatedAlbums])
print("\(detailedAlbum)")

if let tracks = detailedAlbum.tracks {
    print("  Tracks:")
    tracks.prefix(2).forEach { track in
        print("    \(track)")
    }
}
```

Requesting music content with MusicKit.  Find and enjoy albums from apple music.  Search using the search field, connected ot code that matches search result with request.

Even works on media controls in lock screen.

Demo: Looking up by barcode.

## General-purpose data request
* Load data from arbitrary apple music API endpoint
* Unstructured response
* Decode with JSONDecoder
* Define Decodable response type can use music SDK items

```swift
// Loading top level genres

struct MyGenresResponse: Decodable {
    let data: [Genre]
}

let countryCode = try await MusicDataRequest.currentCountryCode
let url = URL(string: "https://api.music.apple.com/v1/catalog/\(countryCode)/genres")!

let dataRequest = MusicDataRequest(urlRequest: URLRequest(url: url))
let dataResponse = try await dataRequest.response()

let decoder = JSONDecoder()
let genresResponse = try decoder.decode(MyGenresResponse.self, from: dataResponse.data)
print("\(genresResponse.data[9])")
```

That's how you load content from any URL in apple music.

# Privacy and user consent
* Required to access apple musi cdata
* Music listening history
* Music library

User consent is given per-device, per-app.

```swift
// Requesting user consent for MusicKit

@State var isAuthorizedForMusicKit = false

func requestMusicAuthorization() {
    detach {
        let authorizationStatus = await MusicAuthorization.request()
        if authorizationStatus == .authorized {
            isAuthorizedForMusicKit = true
        } else {
            // User denied permission.
        }
    }
}
```


# Token management
* Apple music API requires a developer token
* For client-side, developer token is automatically generated
* Must enroll in developer portal.  Where you register your ap ID, specify this as a capabilitiy.
* Personalized Apple Music API endpoints require a user token
	* Also automatically generated


# Subscription information
Exposed as 3 distinct capabilities
* Can play catalog
* has cloud library enabled
* can become subscriber

Check for relevant capability before corresponding app action

```swift
// Using music subscription to drive state of a play button

@State var musicSubscription: MusicSubscription?

var body: some View {
    Button(action: handlePlayButtonSelected) {
        Image(systemName: "play.fill")
    }
    .disabled(!(musicSubscription?.canPlayCatalogContent ?? false))
    .task {
        for await subscription in MusicSubscription.subscriptionUpdates {
            musicSubscription = subscription
        }
    }
}
```


# Playback
Two music players => SystemMusicPlayer and ApplicationMusicPlayer.

Both players handle now playing info, remote commands
However, now playing app is reported as the player for SystemMusicPlayer.  While your app for ApplicatonMusicPlayer.

Playback queue ownership: Remote-control music app, vs your app owns the queue

Only application music player lets you insert items in the middle or remove items that have been added previously.

# Subscription offers
* Main message shown to user is configurable
* Contextual offers highlight one specific song, album, or playlist
* Get rewarded through the apple services performance partners program

```swift
// Showing contextual music subscription offer

@State var musicSubscription: MusicSubscription?
@State var isShowingOffer = false

var offerOptions: MusicSubscriptionOffer.Options {
    var offerOptions = MusicSubscriptionOffer.Options()
    offerOptions.itemID = album.id
    return offerOptions
}

var body: some View {
    Button("Show Subscription Offers", action: showSubscriptionOffer)
        .disabled(!(musicSubscription?.canBecomeSubscriber ?? false))
        .musicSubscriptionOffer(isPresented: $isShowingOffer, options: offerOptions)
}

func showSubscriptionOffer() {
    isShowingOffer = true
}
```

Demo by sigining out of account.  

# Wrap up
* many apps could benefit from using MusicKit
* Improve immersive games with background music
* Upbeat music keeps users motivated in fitness apps
* Engage users through music in social media apps

[[Explore ShazamKit]]
[[Discover concurrency in SwiftUI]]

https://developer.apple.com/documentation/musickit/using_musickit_to_integrate_with_apple_music
https://developer.apple.com/documentation/musickit
