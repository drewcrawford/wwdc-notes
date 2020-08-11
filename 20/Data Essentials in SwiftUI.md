#swiftui 

# Getting started
## Data for a new view
1.  What data does the view need to do its job?
2.  How will the view manipulate that data?
3.  Where will the data come from?  (*source of truth*).  Ultimately this question si the most important one.

```swift
struct BookCard : View {
	//we have this 'let' since view only displays and doesnt modify
	//data comes in through superview during init
    let book: Book
    let progress: Double

    var body: some View {
        HStack {
            Cover(book.coverName)
            VStack(alignment: .leading) {
                TitleText(book.title)
                AuthorText(book.author)
            }
            Spacer()
            RingProgressView(value: progress)              
        }
    }
}
```

## next
let's consider a detail view for a book, with a sheet that can update progress.

```swift
struct EditorConfig {
    var isEditorPresented = false
    var note = ""
    var progress: Double = 0
    mutating func present(initialProgress: Double) {
        progress = initialProgress
        note = ""
        isEditorPresented = true
    }
}
struct BookView: View {
    @State private var editorConfig = EditorConfig()
    func presentEditor() { editorConfig.present(…) }
    var body: some View {
        …
        Button(action: presentEditor) { … }
        …
    }
}
```

`@State` -> source of truth

Recall that after SwiftUI completes a rendering pass, the structs (views) themselves go away.  But because we marked `@State`, swiftuI maintains this value.  Next time SwiftUI renders this, we have a new struct, with reconnected storage.

## progress editor
This is the sheet we present.

```swift
struct EditorConfig {
    var isEditorPresented = false
    var note = ""
    var progress: Double = 0
}
struct BookView: View {
    @State private var editorConfig = EditorConfig()
    var body: some View {
        …
        ProgressEditor(editorConfig: $editorConfig) //$ creates binding from state
        …
    }
}

struct ProgressEditor: View {
	//creates a data dependency between progress editor and parent
    @Binding var editorConfig: EditorConfig
    …
        TextEditor($editorConfig.note) //this is a *new* binding.
    …
}
```
* Properties for data that isn't mutated
* `@State` for transient data owned by the view
* `@Binding` for mutating data owned by another view

# Designing your model
## `ObservableObject`
* Manage life cycle of your data
* Handle side-effects
* Integrate with existing components

Can only be adopted by reference type.  Single requirement, `objectWillChange` which is a `Publisher`.  This publisher fires *before* changes are made to the object.

By default, there is an "out of the box" publisher, but evidently you can customize.  They see to use `associatedType ObjectWillChangePublisher: Publisher = defaultValue` type syntax, which I haven't seen before.

When your type conforms, you are introducing a new source of truth.  

SwiftUI will establish a dependency between data and view.  You can think of ObservableObject as the "surface area" between swiftUI and your real model.  But it may be (probably is) a subset of the true model?  Consider modeling data with a value type, and managing lifecycle with a reference type.

Ex, you may have a *single* observable object, which is your entire modelset, and is shared by your entire app.  This gives you a single place for all your logic.  Or you can have 2-3 specific observableobject to more tightly scope.

```swift
/// The current reading progress for a specific book.
class CurrentlyReading: ObservableObject {
    let book: Book
    @Published var progress: ReadingProgress

    // …
}

struct ReadingProgress {
    struct Entry : Identifiable {
        let id: UUID
        let progress: Double
        let time: Date
        let note: String?
    }

    var entries: [Entry]
}
```

## `@Published`
* Automatically work with `ObservableObject`
* Publishes every time the value changes in `willSet`
* `projectedValue` is a `Publisher`

## How to create an `ObservableObject` dependency
* `@ObservedObject` -> A property wrapper to annotate types in your view that also conform to `@ObservableObject`.  Tracks `@ObservableObject` as a dependency, doesn't own the instance.
* `@StateObject`
* `@EnvironmentObject` (new)

## `@ObservedObject`
```swift
struct BookView: View {
    @ObservedObject var currentlyReading: CurrentlyReading

    var body: some View {
        VStack {
            BookCard(
                currentlyReading: currentlyReading)

            //…

            ProgressDetailsList(
                progress: currentlyReading.progress)
        }
    }
}
```

Whenever you use `@ObservableObject`, SwiftUI will subscribe to the `objectWillChange`.  Then when it will change, the view that depends on it will be updated.

Why `willChange`, why not `didChange`?  SwiftUI needs to know when something is about to change so it can coalesce every change into a single update.  How do we produce mutation?

Many of the components vended by SwiftUI take a binding to a piece of data.  

## Binding from `@ObservableObject`
 * SwiftUI components accept `Binding`
 * `Binding` can be derived from `State` and `ObservableObject`
 * Derive a binding by using the `$` prefix

```swift
class CurrentlyReading: ObservableObject {
    let book: Book
    @Published var progress = ReadingProgress()
    @Published var isFinished = false

    var currentProgress: Double {
        isFinished ? 1.0 : progress.progress
    }
}
```
```swift
struct BookView: View {
    @ObservedObject var currentlyReading: CurrentlyReading

    var body: some View {
        VStack {
            BookCard(
                currentlyReading: currentlyReading)

            HStack {
                Button(action: presentEditor) { /* … */ }
                    .disabled(currentlyReading.isFinished)

                Toggle(
                    isOn: $currentlyReading.isFinished
                ) {
                    Label(
                        "I'm Done",
                        systemImage: "checkmark.circle.fill")
                }
            }
            //…
        }
    }
}
```

Often, you want to tie the lifecycle of your observableobject to your view.  e.x., keep an expensive resource (ex image) around only while it's onscreen.  Because `@ObservedObject` does not own / manage the lifecycle of the underlying `@ObservableObject`, this is a bit complex to tie view lifecycle to object lifecycle.

## `@StateObject` (new)
* SwiftUI owns the `ObservableObject`
* Creation and destruction is tied to the view's life cycle
* Instantiated just before `body`
* Will be kept alive for the full lifecycle of the view

```swift
class CoverImageLoader: ObservableObject {
    @Published public private(set) var image: Image? = nil

    func load(_ name: String) {
        // …
    }

    func cancel() {
        // …
    }

    deinit {
        cancel()
    }
}
```

```swift
struct BookCoverView: View {
	//note this is not instantiated when the view is loaded,
	//instead it's instantiated when we do `body`
	//and kept alive for the whole view lifecycle.  When the view is not
	//needed anymore, it will release the `CoverImageLoader`.
	//don't need to fiddle with `onAppear` and stuff anymore
    @StateObject var loader = CoverImageLoader()

    var coverName: String
    var size: CGFloat

    var body: some View {
        CoverImage(loader.image, size: size)
            .onAppear { loader.load(coverName) }
    }
}
```

In SwiftUI, views are very cheap, so you might have big/wide view trees.  It can be complicated to funnel data through intermediate views that don't need them, to get to child views that do.  In such cases, we have

## `EnvironmentObject` (new)
`.environmentObject(ObservableObject)` injects into the environment
`@EnvironmentObject var foo:` gets from the environment

## wrap up

`ObservableObject` as the data dependency surface
`@ObservedObject` creates a data dependency
`@StateObject` ties an `ObservableObject` to a view's life cycle
`@EnvironmentObject` adds ergonomics to access `@ObservableObject`.

# Techniques for your app
## View life cycle
* Views define a piece of UI
* SwiftUI manages identity and lifetime
* Lightweight and inexpensive

**Note that a lifetime of a view is separate from the lifetime of a struct that defines it**.  Struct you create actually has a very short lifetime – swiftUI uses it during the render, and then it's gone.


Lifecycle is a cycle.  Repeats many times.  Ideally, the cycle carries on smoothly.
But if there's expensive, blocking work at any of these points, performance will suffer.

"Slow update".  Drop frames, hang.

## Avoiding slow updates
* Make view initialization cheap
* Make `body` a pure function
* Avoid assumptions

## ex
this is tricky to follow, but I'll try
```swift
struct ReadingListViewer: View {
    var body: some View {
        NavigationView {
            ReadingList() //this causes the struct below to init
            Placeholder()
        }
    }
}

struct ReadingList: View {
    @ObservedObject var store = ReadingListStore() //and this constructor to run.

    var body: some View {
        // ...
    }
}
```

If the constructor is slow, this is no good.  This "repeated heap allocation" can cause a slow update.

View structs do not have a defined lifetime, so we will create a new copy every time.  

How to fix? `StateObject`

```swift
struct ReadingListViewer: View {
    var body: some View {
        NavigationView {
            ReadingList()
            Placeholder()
        }
    }
}

struct ReadingList: View {
    @StateObject var store = ReadingListStore()

    var body: some View {
        // ...
    }
}
```
will be instantiated at the right time, so you don't incur unnecessary allocations or lose data.

## Event sources
* User interaction
* Publisher (timer)
* `onReceive`
* `onChange`
* `onOpenURL`
* `onContinueUserActivity`

Each of these have a closure which is run "at the right time" to keep body cheap.  I think they're doing diffs and stuff under the hood.

However, these closures are run on the main thread, so consider dispatching to the background queue.

## Source of truth considerations
*Who owns the data*

* Lift data to a common ancestor
* Leverage `@StateObject`
* Consider placing global data in app

## Data lifetime
[[App essentials in SwiftUI]]

Use `State` or `StateObject` to define data lifetime by view lifetime.

Scenes.  Because each scene has a unique tree, you can hang important pieces of data off the root of the tree.  ex., put a source of truth on the view inside of a `WindowGroup`.    Therefore each of these has its own independent source of truth that serves as a lens onto the underlying data model.  Works for multiple windows.

Apps.  Use `@State` etc. in an app. 

```swift
@main
struct BookClubApp: App {
    @StateObject private var store = ReadingListStore() //app-wide source of truth.

    var body: some Scene {
        WindowGroup {
            ReadingListViewer(store: store)
        }
    }
}
```

## Source of Truth lifetime
`State` `StateObject` and `Constant` are tied to process lifetime.  So if app is killed, they won't come back.

"Extended lifetime" is `SceneStorage` and `AppStorage`.  These are *not* your model.  Rather they're lightweight storage.

## `SceneStorage`
per-scene scoped property wrapper.  SwiftUI managed, view **only**.  

What do we need to restore?  Maybe selection.  

```swift
struct ReadingListViewer: View {
    @SceneStorage("selection") var selection: String?

    var body: some View {
        NavigationView {
            ReadingList(selection: $selection)
            BookDetailPlaceholder()
        }
    }
}
```

Now if device restarts, etc., data is available at next launch.  

## `AppStorage`
Persisted with user defaults, usable anywhere.

```swift
struct BookClubSettings: View {
    @AppStorage("updateArtwork") private var updateArtwork = true
    @AppStorage("syncProgress") private var syncProgress = true

    var body: some View {
        Form {
            Toggle(isOn: $updateArtwork) {
                //...
            }

            Toggle(isOn: $syncProgress) {
                //...
            }
        }
    }
}
```

Note that `ObservableObject` could mean backing the data by a server, or by another service.  It's up to you!  So that's another element in the family.

# Wrap up

Not a one-size-fits-all tool.  Typical app will use a variety.

https://developer.apple.com/documentation/swiftui/state-and-data-flow
