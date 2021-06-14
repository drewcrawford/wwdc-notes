#tvOS #swiftui 

# Button styles and context menus
## CardButtonStyle
Raised when focused
Directional effects when dragging on the Siri remote
```swift
Button(albuLabel, action: playAlbum).buttonStyle(CardButtonStyle())
```

## Custom button styles
* No existing effects
* Easy to configure

```swift
struct MyNewButtonStyle: ButtonStyle {
	func makeBody(configuration: Configuration) -> some View {
		configuration.label.background(configuration.isPressed ? ... : ...) //custom styling
	}
}
```
```swift
Button(albumLabel, action: playAlbum).buttonStyle(MyNewButtonStyle())
```
## Context menus
* Invoked on long press gesture
* Add actions using buttons

```swift
AlbumView()
    .contextMenu {
        Button("Add to Favorites", action: addAlbumToFavorites)
        Button("View Artist", action: viewArtistPage)
        Button("Discover Similar Albums", action: viewSimilarAlbums)
    }
```

# Focus
## Focusable modifier
* Creates a focusable wrapper on top of an existing view
* Not meant for intrinsically focusable views
* Configure state using `onFocusChange` callback (tvOS 13)

## isFocused environment variable (tvOS 14)
Check if a view is within focus
View itself does not have to be focusable.  Will return true if the nearest focusable ancestor is focused.

```swift
struct SongView: View {
    var body: some View {
        Button(action: playSong) {
            VStack {
                Image(albumArt)
                DetailsView(...)
            }
        }.buttonStyle(MyCustomButtonStyle())
    }
}

struct DetailsView: View {
    ...
    @Environment(\.isFocused) var isFocused: Bool
    var body: some View {
        VStack {
            Text(songName)
            Text(isFocused ? artistAndAlbum : artistName)
        }
    }
}
```

## Login screen
tvOS will pick a view to focus by default.  Typically this is the topmost or leading.
However, what if we want to focus another view?

### new Default focus API
`prefersDefaultFocus` modifier.  Specify the view of focusing by default.
We want to make sure you don't acidentally screw everything up when you're working on a small view.
`focusScope` -> limits default scope

```swift
var body: some View {
    VStack {
        TextField("Username", text: $username)

        SecureField("Password", text: $password)

        Button("Log In", action: logIn)

    }

}
```

adding focus
```swift
@Namespace private var namespace //limit focus preferences just to the vstack we're working on.  @Namespace is a unique ID that can be added to any view.
@State private var areCredentialsFilled: Bool

var body: some View {
    VStack {
        TextField("Username", text: $username)
            .prefersDefaultFocus(!areCredentialsFilled, in: namespace) //prefer default focus when credentials are not filled, use namespace to limit scope           
        SecureField("Password", text: $password)

        Button("Log In", action: logIn)
           .prefersDefaultFocus(areCredentialsFilled, in: namespace) //when credentials ARE filled, use namespace to limit scope
    }
    .focusScope(namespace) //indicate that VStack belongs to the namespace
}
```
The effect of this is that *if focus is meant to be within the VStack*, we distinguish based on this code.  But if focus was intended to be somewhere else, this isn't involved.

`resetFocus` -> Resets the focus back to default.
Focus updates limited to namespace.

```swift
@Namespace private var namespace
@State private var areCredentialsFilled: Bool
@Environment(\.resetFocus) var resetFocus

var body: some View {
    VStack {
        TextField("Username", text: $username)
            .prefersDefaultFocus(!areCredentialsFilled, in: namespace)            
        SecureField("Password", text: $password)

        Button("Log In", action: logIn)
           .prefersDefaultFocus(areCredentialsFilled, in: namespace)

        Button("Clear", action: { 
            username = ""; password = ""
            areCredentialsFilled = false
            resetFocus(in: namespace)
        })
    }
    .focusScope(namespace)
}
```

Not immediately clear to me why focus is not a computed property of the layout, requiring all this management outside of swiftui, but ok

# Layouts
## Lazy grids
Arrange child views in a grid
Grid items can specify layout properties, size, alignment, spacing, etc.
[[20/what's new in swiftui]]
[[stacks, grids, and outlines in swiftUI]]

```swift
struct ShelfView: View {
    var body: some View {
        ScrollView([.horizontal]) {
            LazyHGrid(rows: [GridItem()]) { //each item loaded as needed
                ForEach(playlists, id: \.self) { playlist in                
                    Button(action: goToPlaylist) {
                        Image(playlist.coverImage)
                            .resizable()
                            .frame(…)
                    }
                    .buttonStyle(CardButtonStyle())
                }
            }
        }
    }
}
```

