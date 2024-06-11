Join us on a tour of SwiftUI, Apple's declarative user interface framework. Learn essential concepts for building apps in SwiftUI, like views, state variables, and layout. Discover the breadth of APIs for building fully featured experiences and crafting unique custom components. Whether you're brand new to SwiftUI or an experienced developer, you'll learn how to take advantage of what SwiftUI has to offer when building great apps.

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

###  14:16 - Adaptive buttons
```swift
Button("Reward", systemImage: "trophy")
    .buttonStyle(.borderless)
//    .buttonStyle(.bordered)
//    .buttonStyle(.borderedProminent)
```

###  14:53 - Adaptive toggles
```swift
Toggle("Nocturnal Mode", systemImage: "moon", isOn: $pet.isNocturnal)
//    .toggleStyle(.switch)
//    .toggleStyle(.checkbox)
//    .toggleStyle(.button)
```

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

###  19:37 - Digital Crown rotation
```swift
ScoreCardStack(rating: $rating)
    .focusable()
    #if os(watchOS)
    .digitalCrownRotation($rating, from: 0, through: 10)
    #endif
```



# Resources
* https://developer.apple.com/tutorials/swiftui-concepts
* https://developer.apple.com/documentation/SwiftUI
* https://developer.apple.com/pathways/swiftui/
