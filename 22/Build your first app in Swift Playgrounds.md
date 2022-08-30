Take things a step further and build apps too.  #playgrounds 

We'll show how to debug, submit, etc.

Build an app to help us at tea time.  Giving us al ist of teas to hepl us pick what to drink each day.  Swift playgrounds works great on mac and iPad.  I'm going to do it on mac.

Variety of templates and instructional content to get you started.  We'll begin by clicking the blank app template.

```swift
Text("Jasmine Green")
```

more teas

```swift
Text("Jasmine Green")
Text("English Breakfast")
Text("Byte's Oolong")
Text("Golden Tippy Assam")
Text("Matt P's Tea Party")
Text("Darjeeling")
Text("Genmaicha")
Text("Jasmine Green")
Text("Vanilla Rooibos")
```

Add swift collection package for `OrderedSet`.

```swift
let teas: OrderedSet<String> = ["Byte's Oolong", "Golden Tippy Assam", "English Breakfast", "Matt P's Tea Party", "Darjeeling", "Genmaicha", "Jasmine Green", "Vanilla Rooibos"]
```

```swift
ForEach(teas, id: \.self) { tea in
     Text(tea)
}
```

Supports capabilities and purpose strings.

Since we're sharing via iCloud shared folder, can drag it out of Locations on iPad.

Use previews by autocompleting `PreviewProvider` suggestion.

```swift
struct TeaWheelView_Previews: PreviewProvider {
    static let items: [String] = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    static var previews: some View {
        Text("Hello, world!")
    }
}
```

Tap on right chevron in sidebar to see this preview, as opposed to the app preview.

 Can leave the insertion point between the two brackets, and then drag on the trailing bracket to create placeholders for additional items.



```swift
struct TeaWheelView_Previews: PreviewProvider {
    static let items: [String] = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    static var previews: some View {
        TeaWheelView(items, id: \.self)
            .padding()
    }
}
```

Delete the button and replace with a `TeaWheelView` instead.  

```swift
TeaWheelView(dataSource.teas, action: { tea in
    lastPickedTea = tea
    showPickAlert = true
})
```

Let's add a print statement in our view preview to see if it's broken too

```swift
struct TeaWheelView_Previews: PreviewProvider {
    static let items: [String] = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5"]
    static var previews: some View {
        TeaWheelView(items, id: \.self) {
            print($0)
        }
            .padding()
    }
}
```

Run apps full-screen.  See "swift" icon in upper right status bar.

Upload to App Store Connect button.  Creates an app record, uploads to ASC, etc.

Now that my app is uploaded, I can go to ASC and submit for beta app review.  Go to TF and install from there, even on iPhone.


# Wrap up
* build apps from scratch
* Debug using previews and console
* Submit to TestFlight and App Store


* https://developer.apple.com/forums/tags/wwdc2022-110348
* https://developer.apple.com/forums/create/question?&tag1=237&tag2=541030
