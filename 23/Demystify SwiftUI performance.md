Learn how you can build a mental model for performance in SwiftUI and write faster, more efficient code. We'll share some of the common causes behind performance issues and help you triage hangs and hitches in SwiftUI to create more responsive views in your app.

as your app's complexity grows, performance gets worse.

Symptom, measure, identify cause, optimize.  Remeasure, reverify.  Break loop.

As your pap gets more complex, you end up with performance bugs.  When you encounter performance issue, have as many tools as you can to triage/fix.

prerequisites
* understand identity, implicit/explicit
* view lifetime and value

[[Demystify SwiftUI]] session to cover these.

# Dependencies

### 3:59 - DogView
```swift
struct DogView: View {
  @Environment(\.isPlayTime) private var isPlayTime
  var dog: Dog
  var body: some View {
  	Text(dog.name)
  		.font(nameFont)
  	Text(dog.breed)
  		.font(breedFont)
  		.foregroundStyle(.secondary)
  		
  	ScalableDogImage(dog)
  	
  	DogDetailView(dog)
  	
  	LetsPlayButton()
  		.disabled(dog.isTired)
  }
 }
```

Dog and play time variables are dependencies of the view.

SwiftUI updates the view.  Return to the graph and look at what happens when the change occurs.

* dynamic properties

If we visualize time on the x axis, first step is to produce a new value for the view.  Encompasses all stored values of the view.

Updates all dynamic properties.  Finally with the updated value, body runs to produce children.

Ends up producing a new image.  Images are leaf views, so the rest of the work is done from SwiftUI.  New rendering is produced.  

Use `Self._printChanges`.  Print out why graph evaluator called you.

If we set a breakpoint in view's body, we can use `expression` to print.  



### 4:00 - ScalableDogImage
```swift
struct ScalableDogImage: View {
	@State private var scaleToFill = false
	var dog: Dog
	
	var body: some View {
		dog.image
			.resizable()
			.aspectRatio(
				contentMode: scaleToFill ? .fill : .fit)
			.frame(maxHeight: scaleToFill ? 500 : nil)
			.padding(.vertical, 16)
			.onTapGesture {
				withAnimation { scaleToFill.toggle() }
			}
	}
}
```

### 4:01 - printChanges
```swift
expression Self._printChanges()
```

### 4:02 - ScalableDogImage + printChanges
```swift
struct ScalableDogImage: View {
	@State private var scaleToFill = false
	var dog: Dog
	
	var body: some View {
    let _ = Self._printChanges()
		dog.image
			.resizable()
			.aspectRatio(
				contentMode: scaleToFill ? .fill : .fit)
			.frame(maxHeight: scaleToFill ? 500 : nil)
			.padding(.vertical, 16)
			.onTapGesture {
				withAnimation { scaleToFill.toggle() }
			}
	}
}
```

basically we can tightly scope the dependency to rely on image instead of dog.
### 8:46 - ScaleableDogImage
```swift
struct ScalableDogImage: View {
	@State private var scaleToFill = false
	var dog: Dog
	
	var body: some View {
		dog.image
			.resizable()
			.aspectRatio(
				contentMode: scaleToFill ? .fill : .fit)
			.frame(maxHeight: scaleToFill ? 500 : nil)
			.padding(.vertical, 16)
			.onTapGesture {
				withAnimation { scaleToFill.toggle() }
			}
	}
}
```

### 8:47 - Updated DogView
```swift
struct DogView: View {
  @Environment(\.isPlayTime) private var isPlayTime
  var dog: Dog
  var body: some View {
  	Text(dog.name)
  		.font(nameFont)
  	Text(dog.breed)
  		.font(breedFont)
  		.foregroundStyle(.secondary)
  		
  	ScalableDogImage(dog)
  	
  	DogDetailView(dog)
  	
  	LetsPlayButton()
  		.disabled(dog.isTired)
  }
 }
```

### 8:48 - Final DogView
```swift
struct DogView: View {
  @Environment(\.isPlayTime) private var isPlayTime
  var dog: Dog
  var body: some View {
  	DogHeader(name: dog.name, breed: dog.breed)
  		
  	ScalableDogImage(dog.image)
  	
  	DogDetailView(dog)
  	
  	LetsPlayButton()
  		.disabled(dog.isTired)
  }
 }
```

* eliminate unnecessary dependencies
* Extract views if needed
* Explore using `Observable`.

[[Discover observation in SwiftUI]]



# Faster view updates
symptoms of slow updates
* reduced resopnsiveness
* hangs/hitch
	* delay in responding to user interaction
	* [[Analyze hangs with Instruments]]
	* hitch: user-perceivable animation issues, skipped frames, etc.  Similar to hangs
	* [[Demistify and eliminate hitches in the render phase - 21]]

Common sources of slow updates
* expensive dynamic properties
* Expensive view body
* Slow identification


### 12:22 - DogRootView and FetchModel
```swift
struct DogRootView: View {
	@State private var model = FetchModel()
	
	var body: some View {
		DogList(model.dogs)
	}
}

@Observable class FetchModel {
	var dogs: [Dog]
	
	init() {
		fetchDogs()
	}
	
	func fetchDogs() {
		// Takes a long time
	}
}
```

### 12:23 - Updated DogRootView and FetchModel
```swift
struct DogRootView: View {
	@State private var model = FetchModel()
	
	var body: some View {
		DogList(model.dogs)
			.task { await model.fetchDogs() }
	}
}

@Observable class FetchModel {
	var dogs: [Dog]
	
	init() {}
	
	func fetchDogs() async {
		// Takes a long time
	}
}
```

Other hidden work
* string interpolation
* Bundle lookup
* Heap allocations (reference types)


# Identity in List and Table

Built-in improvements
* faster filtering
* Reduced time to show large lists
* Smoother scrolling

Construction affects performance!  For consistency, all the IDs are gathered *eagerly*

Being able to quickly generate IDs translates directly to speed!!

Animations; incremental updates to the same view vs a new view
Performance: identifiers are gathered often

[[Explore SwiftUI animation]]

### 15:12 - List
```swift
List {
	ForEach(dogs) {
		DogCell(dog: $0)
	}
}
```

### 16:08 - List Again
```swift
List {
	ForEach(dogs) {
		DogCell(dog: $0)
	}
}
```

`ForEach<Data, ID, Content>`.  List needs to know how many rows to display and what each row  is.

Content closure called to produce each view.  List uses a composite of identity and content.  Rows created on-demand correlate to visible region, plus a system buffer for prefetching.

Suppose we want to refactor our list to only show certain dogs.  so we put an if statement in the list body.  Number of views can be 0 or 1.  This is bad because we need to build all views to retrieve row identifiers because variable number.  Also true if AnyView, number of views is unknown.  All rows must be created.

Instead we can do

```swift
List {
	  ForEAch(dogs.filter(...)) { dog in
		  DogCell(dog)
	  }
}
```

however the inline filter is linear over the collection.  When collection scales, this operation can be expensive.  Better to move it out to the model.

### 17:35 - List Fixed
```swift
List {
	ForEach(tennisBallDogs) { dog in
		DogCell(dog)
	}
}
```

Tips for ensuring constant counts
* avoid using `AnyView` in `ForEach` or lopsided conditions
* Use explicit stacks when appropriate
* Try to flatten nested `ForEach`.

### 18:25 - Sectioned List
```swift
// Sectioned example
struct DogsByToy: View {
	var model: DogModel
	var body: some View {
		List {
			ForEach(model.dogToys) { toy in
				Section(toy.name) {
					ForEach(model.dogs(toy: toy)) { dog in
						DogCell(dog)
					}
				}
			}
		}
	}
}
```

By using Section, SwiftUI understands what we're doing here.  Nested ForEAch is recommended here.

row count = element count x views per element.

Ensure the number of views per element is a constant, or swiftUI has to build the views in addition to the identifiers in order to figure this out.

Table identification
* tableRow always resolves to a single row


### 19:21 - DogTable
```swift
struct DogTable: View {
	var dogs: [Dog]
	var body: some View {
		Table(of: Dog.self) {
			// Columns
		} rows: {
			ForEach(dogs) { dog in
				TableRow(dog)
			}
		}
	}
}
```

New on iOS 17 we have a streamlined initializer 
### 19:22 - DogTable Brief
```swift
struct DogTable: View {
	var dogs: [Dog]
	var body: some View {
		Table(of: Dog.self) {
			// Columns
		} rows: {
			ForEach(dogs)
		}
	}
}
```

This backdeploys to previous OS.  Enforces a constant number of rows which helps with identification performance.

Change this year.  Previously, we used the id of the best friend.  But this requires peering into the loop?  Instead we use the id of the dog itself now, so we dont' need to peer in?



### 20:06 - DogTable Different IDs
```swift
struct DogTable: View {
	var dogs: [Dog]
	var body: some View {
		Table(of: Dog.self) {
			// Columns
		} rows: {
			ForEach(dogs) { dog in
				TableRow(dog.bestFriend)
			}
		}
	}
}
```

row count = element count x views per element.

In tables, it's TableRow per element.

Ensure identifiers are inexpensive
Consider the number of rows in ForEach body (make constant)

* explore dependencies
* Ensure fast identification of list and table rows
* Consider performance early

