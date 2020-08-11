#swiftui 
# Creating an Xcode-compatible playground
See all -> scroll right -> Xcode playground
Evidently this is a template in Swift Playgrounds app.




# Showing a SwiftUI live view
```swift
import SwiftUI
import PlaygroundSupport
```

```swift
struct ProgressView: View {
  
  var body: some View {
    Text("Hello, world!")
  }
  
}
```

```swift
PlaygroundPage.current.setLiveView(ProgressView())
```
# editing using Swift Playgrounds features
```swift
Circle()
	.stroke(lineWidth: 25)
	.foregroundColor(.blue)
```

```swift
//add some padding
ProgressView().padding(150)
```



```swift
//dark mode
.environment(\.colorScheme, .dark)
```
# Working with multiple source files
File ->New File
cmd-shift-] to switch tabs


# Previewing more than one SwiftUI view at once

```swift
struct Preview: View {
 
  var body: some View {
    // ...
  }
  
}
```

```swift
//create a VStack of progress views
VStack(spacing: 30) {
  ProgressView()
  ProgressView()
}
```

```swift
//add a system background color to a view
.background(Color(UIColor.secondarySystemBackground))
```

```swift
//use an environment modifier to preview dark mode
.environment(\.colorScheme, .dark)
```

```swift
func increment() {
  withAnimation {
    self.progress += 0.25
  }
}
```
```swift
Button(action: increment) {
  Text("Increment Progress")
}
```

# Wrap-up
* Swift Playgrounds has numerous features that make editing complex code easier
* Use an Xcode Playground if you plan on bringing your work back to Xcode
* Split your code into multiple source files for clarity




