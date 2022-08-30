#swiftui 

We've always strived to make decisions based on clearly-defined principles.  Today we'll highlight one of them.  Progressive disclosure.

We spent time thinking about and building new API.  The moment you build a an abstraction, you are an API designer.

Peel back the curtainso n our decision process, share what we've learned about progressive disclosure.

What progressive disclosure means.  It's not unique to the design of APIs.  You can see it in action in one fo the most common macOS apis => the save dialog.  When you're first shown a save dialog, you have a default location.  The dialog shows you a dropdown with common locations you're likely to select.  Finally, if you need to browse the filesystem, you can expand the dialog to reveal a more complex/powerful UI.

Different layers of complexity that can be revealed when needed.  We want to provide this experience with your APIs.

**Make APIs feel great to use**.

Declaration site.  We have to look at it from the call site instead.  Progressive disclosure is designing APIU sot he complexity of the callsite grows with complexity of the useccase.  Simple, approachable, able to accommodate powerful usecases.

```swift
struct BookView: View {
    let pageNumber: Int
    let book: Book

    init(book: Book, pageNumber: Int) {
        self.book = book
        self.pageNumber = pageNumber
    }

    var body: some View { ... }
}
```

# Benefits
* minimizes time to first build and run
* Lowers the learning curve
* Tightens the feedback loop

Refinement cycle.  How can we design specific API so they embrace that principle?

# Consider common use cases
When you create a button, we require you to provide a label.  Usually, that's juts some text.
Can customize the button further by taking an arbitrary view as a label.  Build complex functionality out of this simple control.

Because this API careuflly considers common usecases, 99% of the time, you only need the simple version.
# Provide intelligent defaults
All the things we don't specify explicitly.  
```swift
VStack {
    Text("Hello WWDC22!")
    Text("Call to Code.")
}
```
 Text 
 * localization
 * color scheme
 * scale up or down depending on current accessibility dt size.
 * Two texts in a stack: space is automatically adjusted for current context.
When these aren't relevant to your usecase they don't appear at the callsite.  Simple cases is minimal.

ex, toolbar.  Here we have a toolbar with many buttons.  Toolbar buttonsa re placed according to playform convention.  On macOS, leading edge.  But on iOS, navigation bar.  

```swift
.toolbar {
    Button {
        addItem()
    } label: {
        Label("Add", systemImage: "plus")
    }

    Button {
        sort()
    } label: {
        Label("Sort", systemImage: "arrow.up.arrow.down")
    }

    Button {
        openShareSheet()
    }: label: {
        Label("Share", systemImage: "square.and.arrow.up")
    }
}
```

if you do need more control, explicitly specify

```swift
.toolbar {
    ToolbarItemGroup(placement: .navigationBarLeading) {
        Button {
            addItem()
        } label: {
            Label("Add", systemImage: "plus")
        }

        Button {
            sort()
        } label: {
            Label("Sort", systemImage: "arrow.up.arrow.down")
        }

        Button {
            openShareSheet()
        }: label: {
            Label("Share", systemImage: "square.and.arrow.up")
        }
    }
}
```


# Optimize the callsite
If using those APIS feels clunky or unrefined, it can ruin the effect.

Let's look at Table.  Multi-column tables are a feature-rich control.  Lots to configure, etc.

most tables are simpler.

```swift
@State var sortOrder = [KeyPathComparator(\Book.title)]

var body: some View {
    Table(sortOrder: $sortOrder) {
        TableColumn("Title", value: \Book.title) { book in
            Text(book.title).bold()
        }
        TableColumn("Author", value: \Book.author) { book in
            Text(book.author).italic()
        }
    } rows: {
        Section("Favorites") {
            ForEach(favorites) { book in
                TableRow(book)
            }
        }
        Section("Currently Reading") {
            ForEach(currentlyReading) { book in
                TableRow(book)
            }
        }
    }
    .onChange(of: sortOrder) { newValue in
        favorites.sort(using: newValue)
        currentlyReading.sort(using: newValue)
    }
}
```

simpler example
```swift
@State var sortOrder = [KeyPathComparator(\Book.title)]

var body: some View {
    Table(sortOrder: $sortOrder) {
        TableColumn("Title", value: \Book.title) { book in
            Text(book.title)
        }
        TableColumn("Author", value: \Book.author) { book in
            Text(book.author)
        }
    } rows: {
        ForEach(currentlyReading) { book in
            TableRow(book)
        }
    }
    .onChange(of: sortOrder) { newValue in
        currentlyReading.sort(using: newValue)
    }
}
```

Resort the table's data whenever sort order changes.

How to optimize this callsite to embrace progressive disclosure?  One common usecase has to od with rows.  Most of the time, the rows field will look just like this example… foreach over collection.

```swift
@State var sortOrder = [KeyPathComparator(\Book.title)]

var body: some View {
    Table(currentlyReading, sortOrder: $sortOrder) {
        TableColumn("Title", value: \.title) { book in
            Text(book.title)
        }
        TableColumn("Author", value: \.author) { book in
            Text(book.author)
        }
    }
    .onChange(of: sortOrder) { newValue in
        currentlyReading.sort(using: newValue)
    }
}
```

Pass the collection directly to table, to provide `ForEach` behind the scenes.  Still can be simplified further.

Usually, I"ll just use a text for columns.
```swift
@State var sortOrder = [KeyPathComparator(\Book.title)]

var body: some View {
    Table(currentlyReading, sortOrder: $sortOrder) {
        TableColumn("Title", value: \.title)
        TableColumn("Author", value: \.author)
    }
    .onChange(of: sortOrder) { newValue in
        currentlyReading.sort(using: newValue)
    }
}
```
Not all tables are sorted.  So we provide a verison which doesn't sort.
```swift
var body: some View {
    Table(currentlyReading) {
        TableColumn("Title", value: \.title)
        TableColumn("Author", value: \.author)
    }
}
```

Every character serves a clear purpose.  Two key questions at each step

1.  What are the common usecases?
2. what is the essential information that should always be required?

If you don't think through implications, they can lead you astray.

# Compose, don't enumerate
Let's talk about the design of part of swiftuI"s layout system, stacks.  In particular, HStack.

* essential information
	* what content should be in the stack?
	* how should the content be arranged?

Maybe I want to show boxes one after an other, with the leading edge?
or center?
or trailing?

VStack as an API similar to this, alignment.  Might seem tempting to create a similar enum for the arrangement of elements within the stack.  This supports all cases.  Now I can select what I want.

But what if I want to space them out evenly?  Put spacing only between elements? Only before the last element?  This is unsustainable.  I have to add an enum case for every behavior, and that might not be all of them.

When you find yourself enumerating common cases, try breaking your API apart into composable pieces that can build a solution.

You can use `Spacer` to compose different layouts.

```swift
struct StackExample: View {
    var body: some View {
        HStack { // leading
            Box().tint(.red)
            Box().tint(.green)
            Box().tint(.blue)
        }
    }
}
```

```swift
struct StackExample: View {
    var body: some View {
        HStack { // centered
            Spacer()
            Box().tint(.red)
            Box().tint(.green)
            Box().tint(.blue)
            Spacer()
        }
    }
}
```

```swift
struct StackExample: View {
    var body: some View {
        HStack { // evenly spaced
            Spacer()
            Box().tint(.red)
            Spacer()
            Box().tint(.green)
            Spacer()
            Box().tint(.blue)
            Spacer()
        }
    }
}
```

```swift
struct StackExample: View {
    var body: some View {
        HStack { // space only between elements
            Box().tint(.red)
            Spacer()
            Box().tint(.green)
            Spacer()
            Box().tint(.blue)
        }
    }
}
```

```swift
struct StackExample: View {
    var body: some View {
        HStack { // space only before last element
            Box().tint(.red)
            Box().tint(.green)
            Spacer()
            Box().tint(.blue)
        }
    }
}
```

```swift
struct StackExample: View {
    var body: some View {
        HStack { // space only after first element
            Box().tint(.red)
            Spacer()
            Box().tint(.green)
            Box().tint(.blue)
        }
    }
}
```

Also involved careful thought about how that callsite should scale to handle additional cases through composition.

apply the same considerationf or components you create

# Wrap up
* consider common use cases
* Provide itnelligent defaults
* Optimize the callsite
* Compose, don't enumerate
* You are an API designer

* https://developer.apple.com/forums/tags/wwdc2022-10059
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=517030
