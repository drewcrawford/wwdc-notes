#swiftui 

NotificationCenter in macOS was implemented in swiftui.

New additions to layout primitives.  

# Stacks
```swift
// Sandwich model and gallery item view

struct Sandwich: Identifiable {
    var id = UUID()
    var name: String
    var rating: Int
    var heroImage: Image { … }
}

struct HeroView: View {
    var sandwich: Sandwich
    var body: some View {
        sandwich.heroImage
            .resizable()
            .aspectRatio(contentMode: .fit)
            .overlay(BannerView(sandwich: sandwich))
    }
}
```
```swift
// Banner overlay view for sandwich info

struct BannerView: View {
    var sandwich: Sandwich
    var body: some View {
        VStack(alignment: .leading, spacing: 10) {
            Spacer()
            TitleView(title: sandwich.name)
            RatingView(rating: sandwich.rating)
        }
        .padding(…)
        .background(…)
    }
}
```

```swift
// Sandwich rating view

struct RatingView: View {
    var rating: Int
    var body: some View {
        HStack {
            ForEach(0..<5) { starIndex in
                StarImage(isFilled: rating > starIndex)
            }
            Spacer()
        }
    }
}
```

```swift
// Fetch sandwiches from the sandwich store
let sandwiches: [Sandwich] = …

ScrollView {
    VStack(spacing: 0) {
        ForEach(sandwiches) { sandwich in
            HeroView(sandwich: sandwich)
        }
    }
}
```



* LazyVStack and LazyHStack

```swift
// Fetch sandwiches from the sandwich store
let sandwiches: [Sandwich] = …

ScrollView {
    LazyVStack(spacing: 0) {
        ForEach(sandwiches) { sandwich in
            HeroView(sandwich: sandwich)
        }
    }
}
```

Since I made my outer stack lazy, should these stacks be lazy too?
I want the VStack to be lazy because it scrolls, most content can't be seen.
Not true for the other stacks, they all have to be loaded at once.

If you aren't sure which type of stack to use, use VStack/HStack.  Adopt lazy stacks as a way to resolve performance bottlenecks.
# Grids
On iPad, I want more sandwiches on the screen.  
`LazyVGrid` `LazyHGrid`.

```swift
// Fetch sandwiches from the sandwich store
let sandwiches: [Sandwich] = …

// Define grid columns
var columns = [
    GridItem(spacing: 0), //3 columns.
    GridItem(spacing: 0), //each one with equal width?
    GridItem(spacing: 0)
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 0) {
        ForEach(sandwiches) { sandwich in
            HeroView(sandwich: sandwich)
        }
    }
}
```

Can also adapt to space available to get a variable number of columns.

```swift
// Fetch sandwiches from the sandwich store
let sandwiches: [Sandwich] = …

// Define grid columns
var columns = [
    GridItem(.adaptive(minimum: 300), spacing: 0) //as many equally-wide columns as it can, while maintaining the specified width
]

ScrollView {
    LazyVGrid(columns: columns, spacing: 0) {
        ForEach(sandwiches) { sandwich in
            HeroView(sandwich: sandwich)
        }
    }
}
```


# Lists
Always loaded lazily.
```swift
struct GraphicsList: View {
    var graphics: [Graphic]
    var body: some View {
        List(
            graphics,
            children: \.children
        ) { graphic in
            GraphicRow(graphic)
        }
        .listStyle(SidebarListStyle())
    }
}
```


# Outlines
```swift
// Customizing your outlines

List {
    ForEach(canvases) { canvas in
        Section(header: Text(canvas.name)) {
		//similar to ForEach, except instead of iterating over
		//a flat collection, it traverses a tree.
            OutlineGroup(canvas.graphics, children: \.children)
            { graphic in
                GraphicRow(graphic)
            }
        }
    }
}
```

An `OutlineGroup` inside a `List`, is the same as a List that uses the `children` parameter.

Sometimes you want to disclose things that don't folow a regular hierarchy.

## DisclosureGroup

```swift
// Progressive display of information
Form {
	//this binding is optional and the motivation is to open to (predetermined) expanded state
    DisclosureGroup(isExpanded: $areFillControlsShowing) {
       Toggle("Fill shape?", isOn: isFilled)
       ColorRow("Fill color", color: fillColor)
    } label: {
	//label is optional and motivation is to display icon
       Label("Fill", …)
    }
    …
}
```

Provides a disclosure indicator, a label, and content.

How does it work?  Consider

```swift
OutlineGroup(graphics, children: \.children)
```

swiftui expands to

```swift
//swiftui expands to

ForEach(graphics) {graphics in
	DisclosureGroup {
		OutlineGroup(graphics[0].children, children: \.children) {
			GraphicRow($0)
		}
	} label: {
		GraphicRow(graphics[0])
	}
}
```

This unwinding continues until we find a graphic with no children.  It's DisclosureGroups all the way down!

Because SwiftUI only evalautes the content of a disclosure group after someone executes it, we don't do this eagerly

# Wrap up
* HStack & VStack
* LazyHStack and LazyVStack
* LazyHGrid and LazyVGrid
* List
* Form (settings and other lists of controls)
* OutlineGropu and DisclosureGroup let you tailor progressive information


Experiment with the code for ShapeEdit
[[App essentials in SwiftUI]]
[[Data Essentials in SwiftUI]]
