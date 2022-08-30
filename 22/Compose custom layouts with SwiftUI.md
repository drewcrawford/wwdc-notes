#swiftui 

Text, images, graphics, to create custom composite views.  To arrange in sophisticated groupings, we provide layout tools.  Hstack, vstack.  

Whiel view modifiers provide additional control over spacing and alignment.  New tools to make common layouts easier, and make more complicated layouts possible.

1.  Avatars
2. sliders (leaderboard)
3. buttons for voting

## Leaderboard
2d grid of elements with rows for each ocntender, and columns that display each item.

1.  Two text columns should be onlny as wide as they need
2. progress views in the center should have as much space as they can
3. Names to be leading aligned, but amounts for trailing aligned

Lazy grids => great for scrollable content.  Efficient when you have many views.  On the other hand, that means the container can't automatically size itself in both dimensions.  e.x. lazy Hgrid can figure out how wide ot make each column because it can measure the whole column.  So tou must provide info about one dimension at init time

[[Stacks, Grids, and Outlines in SwiftUI]]

For this, we have a `Grid` view.


# Grid
Loads all views at once.  Automatically size and align its cells across both columns and rows.  

```swift
struct Leaderboard: View {
    var body: some View {
        Grid {
            GridRow {
                Text("Cat")
                ProgressView(value: 0.5)
                Text("25")
            }
            GridRow {
                Text("Goldfish")
                ProgressView(value: 0.2)
                Text("9")
            }
            GridRow {
                Text("Dog")
                ProgressView(value: 0.3)
                Text("16")
            }
        }
    }
}
```

Grid allocates as much space to reach row/col as it needs to hold its largest view.  Flexible views take as much space as offered.  

Data model

```swift
struct Pet: Identifiable, Equatable {
    let type: String
    var votes: Int = 0
    var id: String { type }

    static var exampleData: [Pet] = [
        Pet(type: "Cat", votes: 25),
        Pet(type: "Goldfish", votes: 9),
        Pet(type: "Dog", votes: 16)
    ]
}
```

Include `Identifiable` to use in a foreach. `Equatable` to animate changes.

Set of example data to use.

```swift
struct Leaderboard: View {
    var pets: [Pet]
    var totalVotes: Int

    var body: some View {
        Grid(alignment: .leading) {
            ForEach(pets) { pet in
                GridRow {
                    Text(pet.type)
                    ProgressView(
                        value: Double(pet.votes),
                        total: Double(totalVotes))
                    Text("\(pet.votes)")
                        .gridColumnAlignment(.trailing)
                }

                Divider()
            }
        }
        .padding()
    }
}
```

divider problems.  Because my divider is a flexible view, it causes my first item to take more space.

What I really want is for the divider to span all columns in the grid.  so I can say `.gridCellColumns(3)`  Single view spans the columns.

When it spans the entire grid, I can just put the divider outside the row.

On the one hand, I odn't want to bias my participants with small buttons for choices.  But I alsodon't want buttons to grow as large as their container.  Instead each should be equal to the widest text.


with hstack, each button sizes itself to its text level.  This default stack behavior is waht you want in a lot of cases but it doesn't meet what i need]

[[Building Custom Views in SwiftUI]]

What can I change?

First, the stack's container proposes a size to the HStack.  Stack proposes a sie to its three buttons.  Each button passes the size through to its text label.  Text calculates size they want, and report that to the buttom.  button passes to hstack.  Stack sizes itself with that information, and reports its own size to its container.

Wrap each textview in a flexible frame and allow it to grow?  Button sees a flexible subview which takes as much space as offered.  Stack distributes its space qually, so buttons are the same size, but their actual size depends on the container.  Stack will expand to whatever size is offered.  What I want is a custom stack type that asks for each button, finds the widest, and offers that space for each one.

# Layout
For a basic layout, all I need are the two required methods.
`sizethatfits` => calculate how large my container is.  I get a proposed size input.  And I can propose sizes to the subviews.  I can't access the subviews directly, instead it's a collection of proxies.

Collect all those resopnses and use them to do some calculations.  Return a concrete size for our container.

`placeSubviews` => tell my layout's subviews where to appear.  Takes same size proposal and subviews inputs, and a bounds input that represents the region to place subviews in.  Rectangle that has the size that I asked for.  Remember that views pick their own size in swiftUI, so my layout container will get the size that it asks for.

don't assume origin is 0,0.  A nonzero origin allows for composition.  Call another function's layout.

One other parameter.  cache.  Could use to share the results of intermediate calculations across method calls.  For simple layouts you won't need this.  however if profiling your app shows taht you need to improve efficiency of layout, you can look into adding one, see docs.

```swift
struct MyEqualWidthHStack: ViewLayout {
    func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) -> CGSize {
        // Return a size.
    }

    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Void
    ) {
        // Place child views.
    }
}
```

 sizeThatFits => code listing missing.  Refactored into


```swift
private func maxSize(subviews: Subviews) -> CGSize {
    let subviewSizes = subviews.map { $0.sizeThatFits(.unspecified) }
    let maxSize: CGSize = subviewSizes.reduce(.zero) { currentMax, subviewSize in
        CGSize(
            width: max(currentMax.width, subviewSize.width),
            height: max(currentMax.height, subviewSize.height))
    }

    return maxSize
}
```
View can prefer differnet values of spacing for different edges, or based on the kind of view.  VAlues can vary by platform, etc.  You can ignore these preferences.  Which is essentially waht's happening when you initialize a builtin stack with a custom spacing.  but respecting these preferences is a good way to follow interface guidelines.

```swift
private func spacing(subviews: Subviews) -> [CGFloat] {
    subviews.indices.map { index in
        guard index < subviews.count - 1 else { return 0 }
        return subviews[index].spacing.distance(
            to: subviews[index + 1].spacing,
            along: .horizontal)
    }
}
```

A builtin layout container uses the larger of the two preferences, so that's what we'll do here.

```swift
func sizeThatFits(
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) -> CGSize {
    // Return a size.
    guard !subviews.isEmpty else { return .zero }

    let maxSize = maxSize(subviews: subviews)
    let spacing = spacing(subviews: subviews)
    let totalSpacing = spacing.reduce(0) { $0 + $1 }

    return CGSize(
        width: maxSize.width * CGFloat(subviews.count) + totalSpacing,
        height: maxSize.height)
}
```

Now I need to implement `placeSubviews`.


```swift
func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    // Place child views.
    guard !subviews.isEmpty else { return }
  
    let maxSize = maxSize(subviews: subviews)
    let spacing = spacing(subviews: subviews)

    let placementProposal = ProposedViewSize(width: maxSize.width, height: maxSize.height)
    var x = bounds.minX + maxSize.width / 2
  
    for index in subviews.indices {
        subviews[index].place(
            at: CGPoint(x: x, y: bounds.midY),
            anchor: .center,
            proposal: placementProposal)
        x += maxSize.width + spacing[index]
    }
}
```

Size proposal for each subview.  Based on sie that I want them to have, rather than their ideal size.  I want all buttons to be the same size.  

Starting position in the horizontal dimensions.  We start from `minX` so we don't require 0.

It looks like we're spacing based on center here.


```swift
MyEqualWidthHStack {
    ForEach($pets) { $pet in
        Button {
            pet.votes += 1
        } label: {
            Text(pet.type)
                .frame(maxWidth: .infinity)
        }
        .buttonStyle(.bordered)
    }
}
```
```swift
struct Buttons: View {
    @Binding var pets: [Pet]

    var body: some View {
        ForEach($pets) { $pet in
            Button {
                pet.votes += 1
            } label: {
                Text(pet.type)
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.bordered)
        }
    }
}
```

## geometry reader
A tool for measuring view sizes.  But not the best choice here.
It measures the container view and reports the size to the subview.  Subview uses the information to draw its own content.  Intended use, ifnormation flows downward to subviews.

This is great for things like drawing to scale with container e.g. a path.  If container changes size, so does the path, becuase the geometry reader passes along the new size.

For my buttons, I need to measure the text view and then use that to set the frame as the textview's container.  I could add a geometry reader to the overlay, and then send the measurement data outside.  But notice that if I do this, I'm bypassing the layout engine which might result in a loop.  Layout changes frame, changes layout, etc.

it is possible to make this work, but I could end up crashing my app. Not recommended.  Layout protocol gives you a better way to solve this problem from inside the layout engine.

My custom stack doesn't constrain the button widths, but just lets them have the ideal size.  I could modify the layout to do something more complicated when they don't fit, taking into account.  or I could use,
# ViewThatFits
Picks the first view that fits in the space, from the list of views I give it.

```swift
struct StackedButtons: View {
    @Binding var pets: [Pet]

    var body: some View {
        ViewThatFits {
            MyEqualWidthHStack {
                Buttons(pets: $pets)
            }
            MyEqualWidthVStack {
                Buttons(pets: $pets)
            }
        }
    }
}
```

Custom layout for avatars.

```swift
func sizeThatFits(
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
)  -> CGSize {
    // Take whatever space is offered.
    return proposal.replacingUnspecifiedDimensions()
}
```

```swift
func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    let radius = min(bounds.size.width, bounds.size.height) / 3.0
    let angle = Angle.degrees(360.0 / Double(subviews.count)).radians
    let offset = 0 // This depends on rank...

    for (index, subview) in subviews.enumerated() {
        var point = CGPoint(x: 0, y: -radius)
            .applying(CGAffineTransform(
                rotationAngle: angle * Double(index) + offset))

        point.x += bounds.midX
        point.y += bounds.midY

        subview.place(at: point, anchor: .center, proposal: .unspecified)
    }
}
```

Layout protocol lets you store values on each subviews and read values from inside the protocol methods.

```swift
private struct Rank: ViewLayoutKey {
    static let defaultValue: Int = 1
}

extension View {
    func rank(_ value: Int) -> some View {
        layoutValue(key: Rank.self, value: value)
    }
}
```

Default value establishes the associated value's type, e.g. an integer

set value with `layoutValue` method.

read from each subviews:
```swift
func placeSubviews(
    in bounds: CGRect,
    proposal: ProposedViewSize,
    subviews: Subviews,
    cache: inout Void
) {
    let radius = min(bounds.size.width, bounds.size.height) / 3.0
    let angle = Angle.degrees(360.0 / Double(subviews.count)).radians

    let ranks = subviews.map { subview in
        subview[Rank.self]
    }
    let offset = getOffset(ranks)

    for (index, subview) in subviews.enumerated() {
        var point = CGPoint(x: 0, y: -radius)
            .applying(CGAffineTransform(
                rotationAngle: angle * Double(index) + offset))
        point.x += bounds.midX
        point.y += bounds.midY
        subview.place(at: point, anchor: .center, proposal: .unspecified)
    }
}
```

What happens if there's a 3-way tie?  I'd have to substitute completely different layout logic for that case.  

But I'd really like to transition to built-in HStack.  And it turns out I can.

# AnyLayout

```swift
struct Profile: View {
    var pets: [Pet]
    var isThreeWayTie: Bool

    var body: some View {
        let layout = isThreeWayTie ? AnyViewLayout(HStack()) : AnyViewLayout(MyRadialLayout())

        Podium() // Creates the background that shows ranks.
            .overlay(alignment: .top) {
                layout() {
                    ForEach(pets) { pet in
                        Avatar(pet: pet)
                            .rank(rank(pet))
                    }
                }
                .animation(.default, value: pets)
            }
    }
}
```

Because id remains the same, swiftUI sees this as a view that changes, rather than a new view.  By adding the modifier I get animations between all the states of the radial layout.  Becaus eit depends on the same data.

# Wrap up
* grid
* layout
* ViewThatFits
* AnyLayout


* https://developer.apple.com/forums/tags/wwdc2022-10056
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=434030
* https://developer.apple.com/documentation/SwiftUI/Layout-Containers
* https://developer.apple.com/documentation/SwiftUI/ViewThatFits
* https://developer.apple.com/documentation/SwiftUI/AnyLayout
* https://developer.apple.com/documentation/SwiftUI/Layout
* https://developer.apple.com/documentation/SwiftUI/Grid
* https://developer.apple.com/documentation/swiftui/composing_custom_layouts_with_swiftui
* 