#sfsymbols #swiftui 

Note that everything here is available on all apple platforms, some new this year.
# Fundamentals
```swift
// System symbol image
Image(systemName: "heart")

// System symbol label
// This is a generic representation of image
//and string, uses different renderingsin different contexts
Label("Heart", systemImage: "heart")

// Custom symbol image
Image("queen.heart")

// Custom symbol label
Label("Queen of Hearts", image: "queen.heart")
```

When possible, SwiftUI provides a label based on system symbols content.

Might be able to add more specific information about how you use a symbol.  Use an accessibility label to provide information.


```swift
Image(systemName: "heart")
    .accessibilityLabel("Ace of Hearts")

Image(systemName: "person.circle")
    .accessibilityLabel("Profile")

Image("queen.heart")

// Localizeable.strings
"queen.heart" = "Queen of Hearts";
```

as part of text

```swift
Text("""
    Thalia, Paul, and
    3 others \(Image(systemName: "chevron.forward"))
""")
```

When you want a symbol to re-flow around your text.

```swift
Label("Heart", systemImage: "heart")

Label("Heart", systemImage: "heart")
    .foregroundStyle(.red)

Label("Heart", systemImage: "heart")
    .foregroundStyle(.tint)

Label("Heart", systemImage: "heart")
    .foregroundStyle(.secondary)
```

Default to black or white, but you can set foregroundStyle to your own color.

Font modifier can change textsize

```swift
Label("Heart", systemImage: "heart")
    .font(.body)

Label("Heart", systemImage: "heart")
    .font(.caption)

Label("Heart", systemImage: "heart")
    .font(.system(size: 10))
```


# Variants
iOS tab bars should use filled variants.

New this year, tab bars and other views now opt into variants like this.
```swift
TabView {
    Text("Cards").tabItem {
        Label("Cards", systemImage: "rectangle.portrait.on.rectangle.portrait")
    }
    Text("Rules").tabItem {
        Label("Rules", systemImage: "character.book.closed")
    }
    Text("Profile").tabItem {
        Label("Profile", systemImage: "person.circle")
    }
    Text("Magic").tabItem {
        Label("Magic", systemImage: "sparkles")
    }
}
```

Just use the base version, get the right variant without extra work.  By not overspecifying you get more reusable code.

e.g. on macOS, you get outlines.

```swift
List {
    Label("Ace of Hearts", systemImage: "suit.heart")
    Label("Ace of Spades", systemImage: "suit.spade")
    Label("Ace of Diamonds", systemImage: "suit.diamond")
    Label("Ace of Clubs", systemImage: "suit.club")
    Label("Queen of Hearts", image: "queen.heart")
}
.symbolVariant(.fill)
```

Large list of variants.

Works on custom symbols as well.  Provide same naming patterns used by system symbols.

# Rendering modes

[[What's new in SF Symbols]]

## Monochrome

```swift
List {
    Label("Ace of Hearts", systemImage: "suit.heart")
    Label("Ace of Spades", systemImage: "suit.spade")
    Label("Ace of Diamonds", systemImage: "suit.diamond")
    Label("Ace of Clubs", systemImage: "suit.club")
    Label("Queen of Hearts", image: "queen.heart")
}
.symbolVariant(.fill)
```

## multicolor
```swift
List {
    Label("Ace of Hearts", systemImage: "suit.heart")
    Label("Ace of Spades", systemImage: "suit.spade")
    Label("Ace of Diamonds", systemImage: "suit.diamond")
    Label("Ace of Clubs", systemImage: "suit.club")
    Label("Queen of Hearts", image: "queen.heart")
}
.symbolVariant(.fill)
.symbolRenderingMode(.multicolor)
```

[[SF Symbols app overview]]

## Hierarchical

```swift
HStack {
    Button(action: {}) {
        Image(systemName: "square.3.stack.3d.top.fill")
    }
    Button(action: {}) {
        Image(systemName: "square.3.stack.3d.bottom.fill")
    }
}
.symbolRenderingMode(.hierarchical)
```

Multiple levels of opacity to emphasize the key elements.

## Palette

```swift
Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .yellow, .red)
```

Up to 3 styles

If the symbol only has 2 levels, we only use the first 2.  

Since most symbols only use 2 layers, we can specify 2 layers.  Last style for secondary onwards.

This is a style modifier, works with any shape style

```swift
Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .red)

Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.white, .secondary)

Button(action: {}) {
    Image(systemName: "arrow.uturn.backward")
}
.symbolVariant(.circle.fill)
.foregroundStyle(.red, .regularMaterial)
```

[[Add rich graphics to your SwiftUI app]]

# Wrap up
* Creating symbols
* Symbol modifiers
* Variants
* Rendering modes
* Foreground styles

