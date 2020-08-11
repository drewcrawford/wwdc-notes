#xcode
In Xcode 12 you can add your views/modifiers to the xcode library
Benefits
* Discoverability.  Hard for the users of your code to understand .. library
* Learning.  Excellent way of educating users about how a view/modifier is intended to be used.
* Visual editing.  Allows Xcode previews to continue rendering.


```swift
public protocol LibraryContentProvider {
  //extends the "views" xcode library
  @LibraryContentBuilder
  var views: [LibraryItem] { get }

  //extends the "modifiers" xcode library
  @LibraryContentBuilder
  public func modifiers(base: ModifierBase) -> [LibraryItem]
}
```

```swift
LibraryItem(
  SmoothieRowView(smoothie: .lemonberry),
  visible: true,
  title: "Smoothie Row View",
  category: .control
)
```

# Demo
```swift
struct LibraryContent: LibraryContentProvider {
    @LibraryContentBuilder
    var views: [LibraryItem] {
        LibraryItem(
            SmoothieRowView(smoothie: .lemonberry),
            category: .control
        )
        
        LibraryItem(
            SmoothieRowView(smoothie: .lemonberry, showNearbyPopularity: true),
            title: "Smoothie Row View With Popularity",
            category: .control
        )
    }
}
```

By default, library items are inserted in an "app category" (library category for library targets?)  This is OK for targets that have a small number of items.  However, for larger targets you may want to use multiple categories.

SwiftUI's library has various categories.  Controls, layout, effects, etc.  We can do the same for our library items.

Cmd-Shift-L brings up the library.

Views dont' have to be 1:1 with library items, in the example above we create 2 library entries, one includes `showNearbyPopularity`.

We use several modifiers with image "as a unit".  Here's an extension that combines these modifiers

```swift
extension Image {
    func resizedToFill(width: CGFloat, height: CGFloat) -> some View {
        return self
            .resizable()
            .aspectRatio(contentMode: .fill)
            .frame(width: width, height: height)
    }
}
```

Let's add this to our library.  Note this requires a `base` argument.  Xcode needs a way to figure out which part is the modifier and which part is the thing it modifies.

Since our modifier is declared on `Image`, we set this type to image.

```swift
@LibraryContentBuilder
    func modifiers(base: Image) -> [LibraryItem] {
        LibraryItem(
            base.resizedToFill(width: 100.0, height: 100.0)
        )
    }
```

At no point did we have to build/run.  Xcode can harvest library definitions by scanning source for these conformances.

First, if your project is not runnable, it can still contribute content to the library.

There's no additional build config required to enable this feature.  Since it isn't executed, library will strip it when code is built for distribution.

Since Xcode scans all sourcefiles, including dependencies, it works well with swift packages as well.

