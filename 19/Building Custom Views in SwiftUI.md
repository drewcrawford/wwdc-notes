#swiftui 
```swift
struct ContentView : View {
var body: some View {
 	Text("Hello World")
 }
}

```

1.  `ContentView`, which always has the same bounds as its body
2.  `Text`, the body
3.  Finally, the "root view".  Which in this case has the dimensions of the device, minus the safe area insets.

Top layer of any view with a body is always *layout neutral*, so its bounds are defined by the bounds of the body.

So there is really only "Root View" and "Text" here.

1.  Parent (root view) proposes size for child.  Because it's at the root, it offers the size of the whole safe area.
2.  Text replies that it only wants the text size.  In SwiftUI, there's no way to force a size on a child – the parent has to respect that choice.
3.  Now the root view places child in parent's coordinate space.  By default, the middle?

Step 2 means that your views have sizing behavior.  Every view controls its own size.  e.g., the text size never goes beyond the content, ever.

We support some unusual specifications though, like `aspectRatio(1)`, which means we enforce an aspect ratio, but not a size as such.

Note tht SwiftUI rounds coordinates to nearest pixel.  You always get crisp clear edges.  

Background/border color is a good way to debug.  Evidently, the way this works is we place color as a "secondary child", which is sort of an "also path" in the ownership graph.

![[Screen Shot 2020-07-17 at 8.01.05 PM.png]]

```swift
struct Toast : View {
var body: some View {
    Text("Avocado Toast")
    .padding(10)
    .background(Color.green)
} 
}
```

1.  Root View proposes entire size to background view.
2.  Background is layout neutral, so it just passes the size proposal along to the padding view
3.  Padding view knows it's going to add 10 points to the child, so it offers that much less
4.  Text takes the width it needs and returns that
5.  ...which knows it should be bigger than its child by 10 points.  And it centers the text
6.  Background is layout neutral so it will report it upwards...
7.  ...but first it offers the size to the secondary child (Color)
8.  Colors are very compliant.  They accept the size offered to them.
9.  Finally the background view reports its size to the root view
10.  Root view centers it

# Size of frame is not size of image
Frame is not a constraint in SwiftUI
It's just a view.  Which you can think of as a picture frame.  It proposes fixed dimension to its child, but like every other view, child ultimately chooses its own size.

SwiftUI layout uses a lighter touch.

However, there are no underconstrained or overconstrained systems.
No such thing as an incorrect layout unless you don't like the result.

# HStack and VStack
Various auto spaceline behaviors
If you've been 

## How stacks work
* children have to compete for space on an equal footing.

1.  Stack figures out the internal spacing requirements (such as spacing between elements)
2.  Subtracts taht from the proposed width (hstack).  To put together an amount of unallocated space.
3.  Now we have 3 children whose size we don't know.  So we divide the remaining space in 3 equal parts.  We propose one of these as the size for the least-flexible child.
4.  The image is fixed-size, so that's the least flexible.  Whatever size it claimed, we deduct that from the unallocated space and repeat.
5.  Now that all the children have sizes, the stack lines them up with the spacing from step 1.  Since the code didn't specify an alignment, the default of centering is in effect.
6.  Stack chooses its own size which exactly centers the children.

`.layoutPriority(1)` raises from the default of 0.
When children in a stack have different layout priorities, stack divides among children with the highest priority value.

I'm kind of unclear on exactly what size we first propose to an item with high layout priority.  Obviously we reserve some amount for spacing between views, but his statement suggests to me we might also reserve the "minimum" size for the other views in some way (like an elipsis for a text view).  

## Alignments

`.lastTextBaseline`.  

We can compute a text baseline for an image with `Image.alignmentGuide(.lastTextBaseline) { d in d[.bottom] * 0.927}`

Custom alignments

```swift
 
extension VerticalAlignment {
    private enum MidStarAndTitle : AlignmentID {
    static func defaultValue(in d: ViewDimensions) -> Length {
		//doesn't really matter what we pick here
        return d[.bottom]
    }
}

static let midStarAndTitle = VerticalAlignment(MidStarAndTitle.self)
}
```

# Graphical effects

* `animatableData`
* custom transition: `ViewModifier`
* `AnyTransition`
* `.drawingGroup()`
* 