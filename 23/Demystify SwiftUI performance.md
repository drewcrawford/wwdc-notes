Learn how you can build a mental model for performance in SwiftUI and write faster, more efficient code. We'll share some of the common causes behind performance issues and help you triage hangs and hitches in SwiftUI to create more responsive views in your app.

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

### 17:35 - List Fixed
```swift
List {
	ForEach(tennisBallDogs) { dog in
		DogCell(dog)
	}
}
```

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