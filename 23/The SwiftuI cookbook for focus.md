The SwiftUI team is back in the coding "kitchen" with powerful tools to shape your app's focus experience. Join us and learn about the staple ingredients that support focus-driven interactions in your app. Discover focus interactions for custom views, find out about key-press handlers for keyboard input, and learn how to support movement and hierarchy with focus sections. We'll also go through some tasty recipes for common focus patterns in your app.

### Title: Focusable views - 5:05
```swift
// Focusable views

struct RecipeGrid: View {
    var body: some View {
        LazyVGrid(columns: [GridItem(), GridItem()]) {
            ForEach(0..<4) { _ in Capsule() }
        }
        .focusable(interactions: .edit)
    }
}

struct RatingPicker: View {
    var body: some View {
        HStack { Capsule() ; Capsule() }
            .focusable(interactions: .activate)
    }
}
```

### Title: Focus state - 6:12
```swift
// Focus state

struct GroceryListView: View {
    @FocusState private var isItemFocused
    @State private var itemName = ""

    var body: some View {
        TextField("Item Name", text: $itemName)
            .focused($isItemFocused)

        Button("Done") { isItemFocused = false }
            .disabled(!isItemFocused)
    }
}
```

### Title: Focused values - 7:32
```swift
// Focused values

struct SelectedRecipeKey: FocusedValueKey {
    typealias Value = Binding<Recipe>
}

extension FocusedValues {
    var selectedRecipe: Binding<Recipe>? {
        get { self[SelectedRecipeKey.self] }
        set { self[SelectedRecipeKey.self] = newValue }
    }
}

struct RecipeView: View {
    @Binding var recipe: Recipe

    var body: some View {
        VStack {
            Text(recipe.title)
        }
        .focusedSceneValue(\.selectedRecipe, $recipe)
    }
}

struct RecipeCommands: Commands {
    @FocusedBinding(\.selectedRecipe) private var selectedRecipe: Recipe?

    var body: some Commands {
        CommandMenu("Recipe") {
            Button("Add to Grocery List") {
                if let selectedRecipe {
                    addRecipe(selectedRecipe)
                }
            }
            .disabled(selectedRecipe == nil)
        }
    }

    private func addRecipe(_ recipe: Recipe) { /* ... */ }
}

struct Recipe: Hashable, Identifiable {
    let id = UUID()
    var title = ""
    var isFavorite = false
}
```

### Title: Focus sections - 10:03
```swift
// Focus sections

struct ContentView: View {
    @State private var favorites = Recipe.examples
    @State private var selection = Recipe.examples.first!

    var body: some View {
        VStack {
            HStack {
                ForEach(favorites) { recipe in
                    Button(recipe.name) { selection = recipe }
                }
            }

            Image(selection.imageName)

            HStack {
                Spacer()
                Button("Add to Grocery List") { addIngredients(selection) }
                Spacer()
            }
            .focusSection()
        }
    }

    private func addIngredients(_ recipe: Recipe) { /* ... */ }
}

struct Recipe: Hashable, Identifiable {
    static let examples: [Recipe] = [
        Recipe(name: "Apple Pie"),
        Recipe(name: "Baklava"),
        Recipe(name: "Crème Brûlée")
    ]

    let id = UUID()
    var name = ""
    var imageName = ""
}
```

### Title: Controlling focus - 11:29
```swift
struct GroceryListView: View {
    @State private var list = GroceryList.examples
    @FocusState private var focusedItem: GroceryList.Item.ID?

    var body: some View {
        NavigationStack {
            List($list.items) { $item in
                HStack {
                    Toggle("Obtained", isOn: $item.isObtained)
                    TextField("Item Name", text: $item.name)
                        .onSubmit { addEmptyItem() }
                        .focused($focusedItem, equals: item.id)
                }
            }
            .defaultFocus($focusedItem, list.items.last?.id)
            .toggleStyle(.checklist)
        }
        .toolbar {
            Button(action: addEmptyItem) {
                Label("New Item", systemImage: "plus")
            }
        }
    }

    private func addEmptyItem() {
        let newItem = list.addItem()
        focusedItem = newItem.id
    }
}

struct GroceryList: Codable {
    static let examples = GroceryList(items: [
        GroceryList.Item(name: "Apples"),
        GroceryList.Item(name: "Lasagna"),
        GroceryList.Item(name: "")
    ])

    struct Item: Codable, Hashable, Identifiable {
        var id = UUID()
        var name: String
        var isObtained: Bool = false
    }

    var items: [Item] = []

    mutating func addItem() -> Item {
        let item = GroceryList.Item(name: "")
        items.append(item)
        return item
    }
}

struct ChecklistToggleStyle: ToggleStyle {
    func makeBody(configuration: Configuration) -> some View {
        Button {
            configuration.isOn.toggle()
        } label: {
            Image(systemName: configuration.isOn ? "checkmark.circle.fill" : "circle.dashed")
                .foregroundStyle(configuration.isOn ? .green : .gray)
                .font(.system(size: 20))
                .contentTransition(.symbolEffect)
                .animation(.linear, value: configuration.isOn)
        }
        .buttonStyle(.plain)
        .contentShape(.circle)
    }
}

extension ToggleStyle where Self == ChecklistToggleStyle {
    static var checklist: ChecklistToggleStyle { .init() }
}
```

### Title: Custom focusable control - 15:25
```swift
struct RatingPicker: View {
    @Environment(\.layoutDirection) private var layoutDirection
    @Binding var rating: Rating?

    #if os(watchOS)
    @State private var digitalCrownRotation = 0.0
    #endif

    var body: some View {
        EmojiContainer { ratingOptions }
            .contentShape(.capsule)
            .focusable(interactions: .activate)
            #if os(macOS)
            .onMoveCommand { direction in
                selectRating(direction, layoutDirection: layoutDirection)
            }
            #endif
            #if os(watchOS)
            .digitalCrownRotation($digitalCrownRotation, from: 0, through: Double(Rating.allCases.count - 1), by: 1, sensitivity: .low)
            .onChange(of: digitalCrownRotation) { oldValue, newValue in
                if let rating = Rating(rawValue: Int(round(digitalCrownRotation))) {
                    self.rating = rating
                }
            }
            #endif
    }

    private var ratingOptions: some View {
        ForEach(Rating.allCases) { rating in
            EmojiView(rating: rating, isSelected: self.rating == rating) {
                self.rating = rating
            }
        }
    }

    #if os(macOS)
    private func selectRating(
        _ direction: MoveCommandDirection, layoutDirection: LayoutDirection
    ) {
        var direction = direction

        if layoutDirection == .rightToLeft {
            switch direction {
            case .left: direction = .right
            case .right: direction = .left
            default: break
            }
        }

        if let rating {
            switch direction {
            case .left:
                guard let previousRating = rating.previous else { return }
                self.rating = previousRating
            case .right:
                guard let nextRating = rating.next else { return }
                self.rating = nextRating
            default:
                break
            }
        }
    }
    #endif
}

private struct EmojiContainer<Content: View>: View {
    @Environment(\.isFocused) private var isFocused
    private var content: Content

    #if os(watchOS)
    private var strokeColor: Color {
        isFocused ? .green : .clear
    }
    #endif

    init(@ViewBuilder content: @escaping () -> Content) {
        self.content = content()
    }

    var body: some View {
        HStack(spacing: 2) {
            content
        }
            .frame(height: 32)
            .font(.system(size: 24))
            .padding(.horizontal, 8)
            .padding(.vertical, 6)
            .background(.quaternary)
            .clipShape(.capsule)
            #if os(watchOS)
            .overlay(
                Capsule()
                    .strokeBorder(strokeColor, lineWidth: 1.5)
            )
            #endif
    }
}

private struct EmojiView: View {
    var rating: Rating
    var isSelected: Bool
    var action: () -> Void

    var body: some View {
        ZStack {
            Circle()
                .fill(isSelected ? Color.accentColor : Color.clear)
            Text(verbatim: rating.emoji)
                .onTapGesture { action() }
                .accessibilityLabel(rating.localizedName)
        }
    }
}

enum Rating: Int, CaseIterable, Identifiable {
    case meh
    case yummy
    case delicious

    var id: RawValue { rawValue }

    var emoji: String {
        switch self {
        case .meh:
            return "😕"
        case .yummy:
            return "🙂"
        case .delicious:
            return "🥰"
        }
    }

    var localizedName: LocalizedStringKey {
        switch self {
        case .meh:
            return "Meh"
        case .yummy:
            return "Yummy"
        case .delicious:
            return "Delicious"
        }
    }

    var previous: Rating? {
        let ratings = Rating.allCases
        let index = ratings.firstIndex(of: self)!

        guard index != ratings.startIndex else {
            return nil
        }

        let previousIndex = ratings.index(before: index)
        return ratings[previousIndex]
    }

    var next: Rating? {
        let ratings = Rating.allCases
        let index = ratings.firstIndex(of: self)!

        let nextIndex = ratings.index(after: index)
        guard nextIndex != ratings.endIndex else {
            return nil
        }

        return ratings[nextIndex]
    }
}
```

### Title: Grid view - 18:50
```swift
struct ContentView: View {
    @State private var recipes = Recipe.examples
    @State private var selection: Recipe.ID = Recipe.examples.first!.id
    @Environment(\.layoutDirection) private var layoutDirection

    var body: some View {
        LazyVGrid(columns: columns) {
            ForEach(recipes) { recipe in
                RecipeTile(recipe: recipe, isSelected: recipe.id == selection)
                    .id(recipe.id)
                    #if os(macOS)
                    .onTapGesture { selection = recipe.id }
                    .simultaneousGesture(TapGesture(count: 2).onEnded {
                        navigateToRecipe(id: recipe.id)
                    })
                    #else
                    .onTapGesture { navigateToRecipe(id: recipe.id) }
                    #endif
            }
        }
        .focusable()
        .focusEffectDisabled()
        .focusedValue(\.selectedRecipe, $selection)
        .onMoveCommand { direction in
            selectRecipe(direction, layoutDirection: layoutDirection)
        }
        .onKeyPress(.return) {
            navigateToRecipe(id: selection)
            return .handled
        }
        .onKeyPress(characters: .alphanumerics, phases: .down) { keyPress in
            selectRecipe(matching: keyPress.characters)
        }
    }

    private var columns: [GridItem] {
        [ GridItem(.adaptive(minimum: RecipeTile.size), spacing: 0) ]
    }

    private func navigateToRecipe(id: Recipe.ID) {
        // ...
    }

    private func selectRecipe(
        _ direction: MoveCommandDirection, layoutDirection: LayoutDirection
    ) {
        // ...
    }

    private func selectRecipe(matching characters: String) -> KeyPress.Result {
        // ...
        return .handled
    }
}

struct RecipeTile: View {
    static let size = 240.0
    static let selectionStrokeWidth = 4.0

    var recipe: Recipe
    var isSelected: Bool

    private var strokeStyle: AnyShapeStyle {
        isSelected
            ? AnyShapeStyle(.selection)
            : AnyShapeStyle(.clear)
    }

    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: 20)
                .fill(.background)
                .strokeBorder(
                    strokeStyle,
                    lineWidth: Self.selectionStrokeWidth)
                .frame(width: Self.size, height: Self.size)
            Text(recipe.name)
        }
    }
}

struct SelectedRecipeKey: FocusedValueKey {
    typealias Value = Binding<Recipe.ID>
}

extension FocusedValues {
    var selectedRecipe: Binding<Recipe.ID>? {
        get { self[SelectedRecipeKey.self] }
        set { self[SelectedRecipeKey.self] = newValue }
    }
}

struct RecipeCommands: Commands {
    @FocusedBinding(\.selectedRecipe) private var selectedRecipe: Recipe.ID?

    var body: some Commands {
        CommandMenu("Recipe") {
            Button("Add to Grocery List") {
                if let selectedRecipe {
                    addRecipe(selectedRecipe)
                }
            }
            .disabled(selectedRecipe == nil)
        }
    }

    private func addRecipe(_ recipe: Recipe.ID) { /* ... */ }
}

struct Recipe: Hashable, Identifiable {
    static let examples: [Recipe] = [
        Recipe(name: "Apple Pie"),
        Recipe(name: "Baklava"),
        Recipe(name: "Crème Brûlée")
    ]

    let id = UUID()
    var name = ""
    var imageName = ""
}
```

### Title: Focusable grid on tvOS - 21:28
```swift
struct ContentView: View {
    var body: some View {
        HStack {
            VStack {
                List(["Dessert", "Pancake", "Salad", "Sandwich"], id: \.self) {
                    NavigationLink($0, destination: Color.gray)
                }
                Spacer()
            }
            .focusSection()

            ScrollView {
                LazyVGrid(columns: [GridItem(), GridItem()]) {
                    RoundedRectangle(cornerRadius: 5.0)
                        .focusable()
                }
            }
            .focusSection()
        }
    }
}
```
# Resources
* 