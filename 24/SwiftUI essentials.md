Join us on a tour of SwiftUI, Apple's declarative user interface framework. Learn essential concepts for building apps in SwiftUI, like views, state variables, and layout. Discover the breadth of APIs for building fully featured experiences and crafting unique custom components. Whether you're brand new to SwiftUI or an experienced developer, you'll learn how to take advantage of what SwiftUI has to offer when building great apps.

Declarative user interface frameworks, sued to build apps across apple's platforms.  Brand new apps, and incremental adoption with existing apps.  SwiftUI is the right tool to use!

* rich feature set
* Rich interactivity
* Requires less code
* Incremental adoption

# Fundamentals of views
Which pet can do the best tricks?  Over the course of this video, I'll use swiftUI to track pets, tricks, and how they stack up.

Important to everything you do in SwiftUI.  Every pixel you see is in some way defined by a view.

* declarative
* compositional
* state-driven

###  2:30 - Declarative views
```swift
Text("Whiskers")
Image(systemName: "cat.fill")
Button("Give Treat") { // Give Whiskers a treat }
```

###  2:43 - Declarative views: layout
```swift
HStack {
    Label("Whiskers", systemImage: "cat.fill")
    Spacer()
    Text("Tightrope walking")
}
```

###  2:56 - Declarative views: list
```swift
struct ContentView: View {
    @State private var pets = Pet.samplePets
    
    var body: some View {
        List(pets) { pet in
            HStack {
                Label("Whiskers", systemImage: "cat.fill")
                Spacer()
                Text("Tightrope walking")
            }
        }
    }
}

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    
    init(_ name: String, kind: Kind, trick: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
    }
    
    static let samplePets = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking"),
        Pet("Roofus", kind: .dog, trick: "Home runs"),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle"),
        Pet("Mango", kind: .bird, trick: "Basketball dunk"),
        Pet("Ziggy", kind: .lizard, trick: "Parkour"),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip"),
        Pet("Chirpy", kind: .bug, trick: "Canon in D")
    ]
}
```

###  3:07 - Declarative views: list
```swift
struct ContentView: View {
    @State private var pets = Pet.samplePets
    
    var body: some View {
        List(pets) { pet in
            HStack {
                Label(pet.name, systemImage: pet.kind.systemImage)
                Spacer()
                Text(pet.trick)
            }
        }
    }
}

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    
    init(_ name: String, kind: Kind, trick: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
    }
    
    static let samplePets = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking"),
        Pet("Roofus", kind: .dog, trick: "Home runs"),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle"),
        Pet("Mango", kind: .bird, trick: "Basketball dunk"),
        Pet("Ziggy", kind: .lizard, trick: "Parkour"),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip"),
        Pet("Chirpy", kind: .bug, trick: "Canon in D")
    ]
}
```

declarative vs imperative programming.

Not mutually exclusive.  Focus on expected result.  Imperative is great when making a change to state.  SwiftUI embraces both.   ex button action handler is imperative.

###  4:24 - Declarative and imperative programming
```swift
struct ContentView: View {
    @State private var pets = Pet.samplePets
    
    var body: some View {
        Button("Add Pet") {
            pets.append(Pet("Toby", kind: .dog, trick: "WWDC Presenter"))
        }
        
        List(pets) { pet in
            HStack {
                Label(pet.name, systemImage: pet.kind.systemImage)
                Spacer()
                Text(pet.trick)
            }
        }
    }
}

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    
    init(_ name: String, kind: Kind, trick: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
    }
    
    static let samplePets = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking"),
        Pet("Roofus", kind: .dog, trick: "Home runs"),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle"),
        Pet("Mango", kind: .bird, trick: "Basketball dunk"),
        Pet("Ziggy", kind: .lizard, trick: "Parkour"),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip"),
        Pet("Chirpy", kind: .bug, trick: "Canon in D")
    ]
}
```

views are descriptions of the current state, not long-lived object instances.

Defined with struct,s not classes.  SwiftUI takes these and generates an efficient data structure.  Used to produce different outputs.


Because views are declarative descriptions, breaking up one view doesn't hurt the performance of your app.  

###  5:33 - Layout container
```swift
HStack {
    Label("Whiskers", systemImage: "cat.fill")
    Spacer()
    Text("Tightrope walking")
}
```

###  5:41 - Container views
```swift
struct ContentView: View {
    var body: some View {
        HStack {
            Image(whiskers.profileImage)
            VStack(alignment: .leading) {
                Label("Whiskers", systemImage: "cat.fill")
                Text("Tightrope walking")
            }
            Spacer()
        }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```



###  6:23 - View modifiers
```swift
struct ContentView: View {
    var body: some View {
        Image(whiskers.profileImage)
            .clipShape(.circle)
            .shadow(radius: 3)
            .overlay {
                Circle().stroke(.green, lineWidth: 2)
            }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```




###  7:05 - Custom views: Intro
```swift
struct PetRowView: View {
    var body: some View {
        // ...
    }
}
```

###  7:14 - Custom views
```swift
struct PetRowView: View {
    var body: some View {
        Image(whiskers.profileImage)
            .clipShape(.circle)
            .shadow(radius: 3)
            .overlay {
                Circle().stroke(.green, lineWidth: 2)
            }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```

###  7:20 - Custom views: iteration
```swift
struct PetRowView: View {
    var body: some View {
        HStack {
            Image(whiskers.profileImage)
                .clipShape(.circle)
                .shadow(radius: 3)
                .overlay {
                    Circle().stroke(.green, lineWidth: 2)
                }
            Text("Whiskers")
            Spacer()
        }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```

###  7:24 - Custom views: view properties
```swift
struct PetRowView: View {
    var body: some View {
        HStack {
            profileImage
            VStack(alignment: .leading) {
                Text("Whiskers")
                Text("Tightrope walking")
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
            Spacer()
        }
    }
    
    private var profileImage: some View {
        Image(whiskers.profileImage)
            .clipShape(.circle)
            .shadow(radius: 3)
            .overlay {
                Circle().stroke(.green, lineWidth: 2)
            }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```

###  7:34 - Custom views: complete row view
```swift
struct PetRowView: View {
    var body: some View {
        HStack {
            profileImage
            VStack(alignment: .leading) {
                HStack(alignment: .firstTextBaseline) {
                    Text("Whiskers")
                    if pet.hasAward {
                        Image(systemName: "trophy.fill")
                            .foregroundStyle(.orange)
                    }
                }
                Text("Tightrope walking")
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
            Spacer()
        }
    }
    
    private var profileImage: some View {
        Image(whiskers.profileImage)
            .clipShape(.circle)
            .shadow(radius: 3)
            .overlay {
                Circle().stroke(.green, lineWidth: 2)
            }
    }
}

let whiskers = Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers")

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    var hasAward: Bool = false
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
    }
}
```

###  7:41 - Custom views: input properties
```swift
struct PetRowView: View {
    var pet: Pet
    
    var body: some View {
        HStack {
            profileImage
            VStack(alignment: .leading) {
                Text(pet.name)
                Text(pet.trick)
                    .font(.subheadline)
                    .foregroundStyle(.secondary)
            }
            Spacer()
        }
    }
    
    private var profileImage: some View {
        Image(pet.profileImage)
            .clipShape(.circle)
            .shadow(radius: 3)
            .overlay {
                Circle().stroke(pet.favoriteColor, lineWidth: 2)
            }
    }
}

struct Pet: Identifiable {
    enum Kind {
        case cat
        case dog
        case fish
        case bird
        case lizard
        case turtle
        case rabbit
        case bug
        
        var systemImage: String {
            switch self {
            case .cat: return "cat.fill"
            case .dog: return "dog.fill"
            case .fish: return "fish.fill"
            case .bird: return "bird.fill"
            case .lizard: return "lizard.fill"
            case .turtle: return "tortoise.fill"
            case .rabbit: return "rabbit.fill"
            case .bug: return "ant.fill"
            }
        }
    }
    
    let id = UUID()
    var name: String
    var kind: Kind
    var trick: String
    var profileImage: String
    var favoriteColor: Color
    
    init(_ name: String, kind: Kind, trick: String, profileImage: String, favoriteColor: Color) {
        self.name = name
        self.kind = kind
        self.trick = trick
        self.profileImage = profileImage
        self.favoriteColor = favoriteColor
    }
}
```

###  7:53 - Custom views: reuse
```swift
PetRowView(pet: model.pet(named: "Whiskers"))
PetRowView(pet: model.pet(named: "Roofus"))
PetRowView(pet: model.pet(named: "Bubbles"))
```
###  7:59 - List composition
```swift
struct ContentView: View {
    var model: PetStore
    
    var body: some View {
        List(model.allPets) { pet in
            PetRowView(pet: pet)
        }
    }
}

@Observable class PetStore {
    var allPets: [Pet] = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers", favoriteColor: .green),
        Pet("Roofus", kind: .dog, trick: "Home runs", profileImage: "Roofus", favoriteColor: .blue),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle", profileImage: "Bubbles", favoriteColor: .orange),
        Pet("Mango", kind: .bird, trick: "Basketball dunk", profileImage: "Mango", favoriteColor: .green),
        Pet("Ziggy", kind: .lizard, trick: "Parkour", profileImage: "Ziggy", favoriteColor: .purple),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip", profileImage: "Sheldon", favoriteColor: .brown),
        Pet("Chirpy", kind: .bug, trick: "Canon in D", profileImage: "Chirpy", favoriteColor: .orange)
    ]
}
```

###  8:14 - List composition: ForEach
```swift
struct ContentView: View {
    var model: PetStore
    
    var body: some View {
        List {
            ForEach(model.allPets) { pet in
                PetRowView(pet: pet)
            }
        }
    }
}

@Observable class PetStore {
    var allPets: [Pet] = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers", favoriteColor: .green),
        Pet("Roofus", kind: .dog, trick: "Home runs", profileImage: "Roofus", favoriteColor: .blue),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle", profileImage: "Bubbles", favoriteColor: .orange),
        Pet("Mango", kind: .bird, trick: "Basketball dunk", profileImage: "Mango", favoriteColor: .green),
        Pet("Ziggy", kind: .lizard, trick: "Parkour", profileImage: "Ziggy", favoriteColor: .purple),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip", profileImage: "Sheldon", favoriteColor: .brown),
        Pet("Chirpy", kind: .bug, trick: "Canon in D", profileImage: "Chirpy", favoriteColor: .orange)
    ]
}
```

###  8:27 - List composition: sections
```swift
struct ContentView: View {
    var model: PetStore
    
    var body: some View {
        List {
            Section("My Pets") {
                ForEach(model.myPets) { pet in
                    PetRowView(pet: pet)
                }
            }
            
            Section("Other Pets") {
                ForEach(model.otherPets) { pet in
                    PetRowView(pet: pet)
                }
            }
        }
    }
}

@Observable class PetStore {
    var myPets: [Pet] = [
        Pet("Roofus", kind: .dog, trick: "Home runs", profileImage: "Roofus", favoriteColor: .blue),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip", profileImage: "Sheldon", favoriteColor: .brown),
    ]
    
    var otherPets: [Pet] = [
        Pet("Whiskers", kind: .cat, trick: "Tightrope walking", profileImage: "Whiskers", favoriteColor: .green),
        Pet("Bubbles", kind: .fish, trick: "100m freestyle", profileImage: "Bubbles", favoriteColor: .orange),
        Pet("Mango", kind: .bird, trick: "Basketball dunk", profileImage: "Mango", favoriteColor: .green),
        Pet("Ziggy", kind: .lizard, trick: "Parkour", profileImage: "Ziggy", favoriteColor: .purple),
        Pet("Chirpy", kind: .bug, trick: "Canon in D", profileImage: "Chirpy", favoriteColor: .orange)
    ]
}
```

###  8:36 - List composition: section actions
```swift
PetRowView(pet: pet)
    .swipeActions(edge: .leading) {
        Button("Award", systemImage: "trophy") {
            // Give pet award
        }
        .tint(.orange)
        ShareLink(item: pet, preview: SharePreview("Pet", image: Image(pet.name)))
    }
```
## state-driven

When your view's state changes, we keep views up to date.

As data changes, new view values are created and given to swiftUI.  My app now has a list of pets and their tricks.  The most important part of a pet competition is rewarding the ones with the best tricks.  



###  9:31 - View updates
```swift
struct ContentView: View {
    var model: PetStore
    
    var body: some View {
        Button("Add Pet") {
            model.addPet()
        }
        
        List(model.pets) { pet in
            PetRowView(pet: model.pet)
                .swipeActions(edge: .leading) {
                    Button("Award", systemImage: "trophy") {
                        pet.giveAward()
                    }
                    .tint(.orange)
                    ShareLink(item: pet, preview: SharePreview("Pet", image: Image(pet.name)))
                }
        }
    }
}

@Observable class PetStore {
    var pets: [Pet] = [
        Pet("Roofus", kind: .dog, trick: "Home runs"),
        Pet("Sheldon", kind: .turtle, trick: "Kickflip"),
    ]
    
    func addPet() {
        pets.append(Pet("Toby", kind: .dog, trick: "WWDC Presenter"))
    }
}

struct Pet {
    var name: String
    var kind: String
    var trick: String
    var hasAward: Bool
    
    mutating func giveAward() {
        hasAward = true
    }
}
```

IN my app I created an `@Observable` Pet class.  SwiftUI creates dependencies to specific properties used.  Two other important ones are `@State` and `@Binding`.

state - new internal source of data for a view.
binding - two-way reference between state and some other view.


###  10:57 - State changes
```swift
struct RatingView: View {
    @State var rating: Int = 5
    
    var body: some View {
        HStack {
            Button("Decrease", systemImage: "minus.circle") {
                rating -= 1
            }
            .disabled(rating == 0)
            .labelStyle(.iconOnly)
            
            Text(rating, format: .number.precision(.integerLength(2)))
                .font(.title.bold())
            
            Button("Increase", systemImage: "plus.circle") {
                rating += 1
            }
            .disabled(rating == 10)
            .labelStyle(.iconOnly)
        }
    }
}
```


Button's action is called when tapped.  Increments internal state of the view.  SwiftUI notices this change and calls body on rating view, returning a new text value.  Result is updated onscreen.  

Customize transition.  In this case, usin ga numeric text content transition works perfectly.



###  11:51 - State changes: animation
```swift
struct RatingView: View {
    @State var rating: Int = 5
    
    var body: some View {
        HStack {
            Button("Decrease", systemImage: "minus.circle") {
                withAnimation {
                    rating -= 1
                }
            }
            .disabled(rating == 0)
            .labelStyle(.iconOnly)
            
            Text(rating, format: .number.precision(.integerLength(2)))
                .font(.title.bold())
            
            Button("Increase", systemImage: "plus.circle") {
                withAnimation {
                    rating += 1
                }
            }
            .disabled(rating == 10)
            .labelStyle(.iconOnly)
        }
    }
}
```

###  12:05 - State changes: text content transition
```swift
struct RatingView: View {
    @State var rating: Int = 5
    
    var body: some View {
        HStack {
            Button("Decrease", systemImage: "minus.circle") {
                withAnimation {
                    rating -= 1
                }
            }
            .disabled(rating == 0)
            .labelStyle(.iconOnly)
            
            Text(rating, format: .number.precision(.integerLength(2)))
                .contentTransition(.numericText(value: Double(rating)))
                .font(.title.bold())
            
            Button("Increase", systemImage: "plus.circle") {
                withAnimation {
                    rating += 1
                }
            }
            .disabled(rating == 10)
            .labelStyle(.iconOnly)
        }
    }
}
```

Here I combine multiple components.  Currently they each have their own state, with their own separate sources of truth..  But when rating view increments its own state, the other doesn't change.



###  12:22 - State changes: multiple state
```swift
struct RatingContainerView: View {
    @State private var rating: Int = 5
    
    var body: some View {
        Gauge(value: Double(rating), in: 0...10) {
            Text("Rating")
        }
        
        RatingView(rating: $rating)
    }
}

struct RatingView: View {
    @State var rating: Int = 5
    
    var body: some View {
        HStack {
            Button("Decrease", systemImage: "minus.circle") {
                withAnimation {
                    rating -= 1
                }
            }
            .disabled(rating == 0)
            .labelStyle(.iconOnly)
            
            Text(rating, format: .number.precision(.integerLength(2)))
                .contentTransition(.numericText(value: Double(rating)))
                .font(.title.bold())
            
            Button("Increase", systemImage: "plus.circle") {
                withAnimation {
                    rating += 1
                }
            }
            .disabled(rating == 10)
            .labelStyle(.iconOnly)
        }
    }
}
```

###  12:45 - State changes: state and binding
```swift
struct RatingContainerView: View {
    @State private var rating: Int = 5
    
    var body: some View {
        Gauge(value: Double(rating), in: 0...10) {
            Text("Rating")
        }
        
        RatingView(rating: $rating)
    }
}

struct RatingView: View {
    @Binding var rating: Int
    
    var body: some View {
        HStack {
            Button("Decrease", systemImage: "minus.circle") {
                withAnimation {
                    rating -= 1
                }
            }
            .disabled(rating == 0)
            .labelStyle(.iconOnly)
            
            Text(rating, format: .number.precision(.integerLength(2)))
                .font(.title.bold())
            
            Button("Increase", systemImage: "plus.circle") {
                withAnimation {
                    rating += 1
                }
            }
            .disabled(rating == 10)
            .labelStyle(.iconOnly)
        }
    }
}
```


# Built-in capability

Even higher starting point for building your app.  I've just begun on the app tracking pets/tricks.  SwiftUI automatically provides adaptivity:
* dark mode
* ax features
* localization

Exactly how a feature you're working on will feel.  

Adaptivity.  Describe the purpose of the functionality as opposed to visual construction.  

Buttons are an adaptive view.  action, label.


###  14:16 - Adaptive buttons
```swift
Button("Reward", systemImage: "trophy")
    .buttonStyle(.borderless)
//    .buttonStyle(.bordered)
//    .buttonStyle(.borderedProminent)
```

Swipe actions, menus, forms, etc.  


###  14:53 - Adaptive toggles
```swift
Toggle("Nocturnal Mode", systemImage: "moon", isOn: $pet.isNocturnal)
//    .toggleStyle(.switch)
//    .toggleStyle(.checkbox)
//    .toggleStyle(.button)
```

In different contexts, they look different.  Taking advantage of composition to affect behavior and enable customization.  




###  15:19 - Searchable
```swift
struct PetListView: View {
    @Bindable var viewModel: PetStoreViewModel
    
    var body: some View {
        List {
            Section("My Pets") {
                ForEach(viewModel.myPets) { pet in
                    row(pet: pet)
                }
            }
            
            Section("Other Pets") {
                ForEach(viewModel.otherPets) { pet in
                    row(pet: pet)
                }
            }
        }
        .searchable(text: $viewModel.searchText)
    }
    
    private func row(pet: Pet) -> some View {
        PetRowView(pet: pet)
            .swipeActions(edge: .leading) {
                Button("Reward", systemImage: "trophy") {
                    pet.giveAward()
                }
                .tint(.orange)
                ShareLink(item: pet, preview: SharePreview("Pet", image: Image(pet.name)))
            }
    }
}

@Observable class PetStoreViewModel {
    var petStore: PetStore
    var searchText: String = ""
    
    init(petStore: PetStore) {
        self.petStore = petStore
    }
    
    var myPets: [Pet] {
        petStore.myPets.filter { pet in
            searchText.isEmpty || pet.name.contains(searchText)
        }
    }
    
    var otherPets: [Pet] {
        petStore.otherPets.filter { pet in
            searchText.isEmpty || pet.name.contains(searchText)
        }
    }
}
```

customize with suggestions, scopes, tokens.



###  15:20 - Searchable: customization
```swift
struct PetListView: View {
    @Bindable var viewModel: PetStoreViewModel
    
    var body: some View {
        List {
            Section("My Pets") {
                ForEach(viewModel.myPets) { pet in
                    row(pet: pet)
                }
            }
            
            Section("Other Pets") {
                ForEach(viewModel.otherPets) { pet in
                    row(pet: pet)
                }
            }
        }
        .searchable(text: $viewModel.searchText, editableTokens: $viewModel.searchTokens) { token in
            Label(token.kind.name, systemImage: token.kind.systemImage)
        }
        .searchScopes($viewModel.searchScope) {
            Text("All Pets").tag(PetStoreViewModel.SearchScope.allPets)
            Text("My Pets").tag(PetStoreViewModel.SearchScope.myPets)
            Text("Other Pets").tag(PetStoreViewModel.SearchScope.otherPets)
        }
        .searchSuggestions {
            PetSearchSuggestions(viewModel: viewModel)
        }
    }
    
    private func row(pet: Pet) -> some View {
        PetRowView(pet: pet)
            .swipeActions(edge: .leading) {
                Button("Reward", systemImage: "trophy") {
                    pet.giveAward()
                }
                .tint(.orange)
                ShareLink(item: pet, preview: SharePreview("Pet", image: Image(pet.name)))
            }
    }
}
```


Custom metal shaders directly to swiftUI views.  

WindowGroup.  Content view to show onscreen.  Scenes can also be composed together.  On multi-windowed platforms, scenes provide different ways to adapt with your app's capabilities.

###  16:58 - App definition
```swift
@main struct SwiftUIEssentialsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

###  17:15 - App definition: multiple scenes
```swift
@main struct SwiftUIEssentialsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        
        WindowGroup("Training History", id: "history", for: TrainingHistory.ID.self) {
            $id in TrainingHistoryView(historyID: id)
        }
        
        WindowGroup("Pet Detail", id: "detail", for: Pet.ID.self) {
            $id in PetDetailView(petID: id)
        }
    }
}
```

###  17:23 - Widgets
```swift
struct ScoreboardWidget: Widget {
    var body: some WidgetConfiguration {
        // ...
    }
}

struct ScoreboardWidgetView: View {
    var petTrick: PetTrick
    
    var body: some View {
        ScoreCard(rating: petTrick.rating)
            .overlay(alignment: .bottom) {
                Text(petTrick.pet.name)
                    .padding()
            }
            .widgetURL(petTrick.pet.url)
    }
}
```

Widgets - homescreen and desktop, composed out of views.  

# Across all platforms

Our capabilities extend to every platform.  Any apple platform.  Multiplier on your efforts.  Once you have a user interface with swiftuI on one platform, you have an excellent start to building your app on other platforms.

Idiomatic look and feel for any platforms.  Keyboard navigation, multiple windows, etc.  Same use of search suggestions results in a standard dropdown on macOS, and an overlay list on iOS.

It is not write once and run everywhere.  it is **learn once, use anywhere**.  

Hig describes components and platform considerations.  Navigation splitview is different on watchOS, etc.

###  19:37 - Digital Crown rotation
```swift
ScoreCardStack(rating: $rating)
    .focusable()
    #if os(watchOS)
    .digitalCrownRotation($rating, from: 0, through: 10)
    #endif
```

Bring my app to visionOS, taking advantage of views from other platforms, volumetric content, etc.  Incremental in nature, doesn't require you to support multiple platforms, but a headstart for when you're ready.

# SDK interoperability

Comes with each platform's SDK.  Many other frameworks in the SDK.  As easy as dropping in another view or property into your app.  UIKit/AppKit are imperative, object oriented user interface frameworks.  Use different patterns for creating and updating views.  Longstanding, rich capabilities that swiftui builds on.

Create a UIViewRepresentable/ NSViewRepresentable.  

Or to embed in UIView, use HostingViewController.  Add to UIKit/AppKit view hierarchy.  

Tools for building great apps.  No expectation that an app needs to be entirely swiftUI.  Every framework brings its own unique capabilities
* swiftdata - persistent models.  Connect, query models from swiftui views.
* swift charts - customizable charting framework.  Create gorgeous information visualizations.


* less code
* rich feature set
* Incremental adoption

# Next steps
* get started in xcode
* watch an introduction to swiftUI
* follow along with tutorials
* see docs







# Resources
* https://developer.apple.com/tutorials/swiftui-concepts
* https://developer.apple.com/documentation/SwiftUI
* https://developer.apple.com/pathways/swiftui/
