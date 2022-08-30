#swiftui 

Basic stacks, like on apple tv, iPHone, apple watch.  To multicolumn presentation.

bring robust support for programmatic navigation and deep linking.  

# Review
Based on links.  Views are shown in other columns.
ex, list of navigation links in a list view.  It pushes its view on the stack.
This works ok for basic navigation, you can continue using this pattern.

`NavigationLink`.  Add a binding to the link.  But this means I need a separaet ibnding for each link.

With the new API, we moved the binding up to the contianer, called the `NavigationSTack`.  path represents all values.

link appends values to the path, or you can pop.


# New navigation APIs
ex cooking app.

NavigationStack.  Push/pop interface, like settings on iPhone.  

NavigationSplitView.  Multicolumn apps like Mail/Notes on mac and iPad.  Adapts to a single column stack, iPhone, slideover on iPad, watch, tv. 
It has two initializers.  Two columns or 3 columns.

[[SwiftUI on iPad Organize your interface]]

Navigation links required a title and view to present.  These still include a title, but now include a value.  Link's behavior depeneds on the stack/list it appears in.
# Recipes for navigation
Basic stack of views, like iPhone settings.  Section => Details => More detail.  Pop to unwind.

## Pushable stack

```swift
import SwiftUI

// Pushable stack
struct PushableStack: View {
    @State private var path: [Recipe] = []
    @StateObject private var dataModel = DataModel()

    var body: some View {
        NavigationStack(path: $path) {
            List(Category.allCases) { category in
                Section(category.localizedName) {
                    ForEach(dataModel.recipes(in: category)) { recipe in
                        NavigationLink(recipe.name, value: recipe)
                    }
                }
            }
            .navigationTitle("Categories")
            .navigationDestination(for: Recipe.self) { recipe in
                RecipeDetail(recipe: recipe)
            }
        }
        .environmentObject(dataModel)
    }
}

// Helpers for code example
struct RecipeDetail: View {
    @EnvironmentObject private var dataModel: DataModel
    var recipe: Recipe

    var body: some View {
        Text("Recipe details go here")
            .navigationTitle(recipe.name)
        ForEach(recipe.related.compactMap { dataModel[$0] }) { related in
            NavigationLink(related.name, value: related)
        }
    }
}

class DataModel: ObservableObject {
    @Published var recipes: [Recipe] = builtInRecipes

    func recipes(in category: Category?) -> [Recipe] {
        recipes
            .filter { $0.category == category }
            .sorted { $0.name < $1.name }
    }

    subscript(recipeId: Recipe.ID) -> Recipe? {
        // A real app would want to maintain an index from identifiers to
        // recipes.
        recipes.first { recipe in
            recipe.id == recipeId
        }
    }
}

enum Category: Int, Hashable, CaseIterable, Identifiable, Codable {
    case dessert
    case pancake
    case salad
    case sandwich

    var id: Int { rawValue }

    var localizedName: LocalizedStringKey {
        switch self {
        case .dessert:
            return "Dessert"
        case .pancake:
            return "Pancake"
        case .salad:
            return "Salad"
        case .sandwich:
            return "Sandwich"
        }
    }
}

struct Recipe: Hashable, Identifiable {
    let id = UUID()
    var name: String
    var category: Category
    var ingredients: [Ingredient]
    var related: [Recipe.ID] = []
    var imageName: String? = nil
}

struct Ingredient: Hashable, Identifiable {
    let id = UUID()
    var description: String

    static func fromLines(_ lines: String) -> [Ingredient] {
        lines.split(separator: "\n", omittingEmptySubsequences: true)
            .map { Ingredient(description: String($0)) }
    }
}

let builtInRecipes: [Recipe] = {
    var recipes = [
        "Apple Pie": Recipe(
            name: "Apple Pie", category: .dessert,
            ingredients: Ingredient.fromLines(applePie)),
        "Baklava": Recipe(
            name: "Baklava", category: .dessert,
            ingredients: []),
        "Bolo de Rolo": Recipe(
            name: "Bolo de rolo", category: .dessert,
            ingredients: []),
        "Chocolate Crackles": Recipe(
            name: "Chocolate crackles", category: .dessert,
            ingredients: []),
        "Crème Brûlée": Recipe(
            name: "Crème brûlée", category: .dessert,
            ingredients: []),
        "Fruit Pie Filling": Recipe(
            name: "Fruit Pie Filling", category: .dessert,
            ingredients: []),
        "Kanom Thong Ek": Recipe(
            name: "Kanom Thong Ek", category: .dessert,
            ingredients: []),
        "Mochi": Recipe(
            name: "Mochi", category: .dessert,
            ingredients: []),
        "Marzipan": Recipe(
            name: "Marzipan", category: .dessert,
            ingredients: []),
        "Pie Crust": Recipe(
            name: "Pie Crust", category: .dessert,
            ingredients: Ingredient.fromLines(pieCrust)),
        "Shortbread Biscuits": Recipe(
            name: "Shortbread Biscuits", category: .dessert,
            ingredients: []),
        "Tiramisu": Recipe(
            name: "Tiramisu", category: .dessert,
            ingredients: []),
        "Crêpe": Recipe(
            name: "Crêpe", category: .pancake, ingredients: []),
        "Jianbing": Recipe(
            name: "Jianbing", category: .pancake, ingredients: []),
        "American": Recipe(
            name: "American", category: .pancake, ingredients: []),
        "Dosa": Recipe(
            name: "Dosa", category: .pancake, ingredients: []),
        "Injera": Recipe(
            name: "Injera", category: .pancake, ingredients: []),
        "Acar": Recipe(
            name: "Acar", category: .salad, ingredients: []),
        "Ambrosia": Recipe(
            name: "Ambrosia", category: .salad, ingredients: []),
        "Bok l'hong": Recipe(
            name: "Bok l'hong", category: .salad, ingredients: []),
        "Caprese": Recipe(
            name: "Caprese", category: .salad, ingredients: []),
        "Ceviche": Recipe(
            name: "Ceviche", category: .salad, ingredients: []),
        "Çoban salatası": Recipe(
            name: "Çoban salatası", category: .salad, ingredients: []),
        "Fiambre": Recipe(
            name: "Fiambre", category: .salad, ingredients: []),
        "Kachumbari": Recipe(
            name: "Kachumbari", category: .salad, ingredients: []),
        "Niçoise": Recipe(
            name: "Niçoise", category: .salad, ingredients: []),
    ]

    recipes["Apple Pie"]!.related = [
        recipes["Pie Crust"]!.id,
        recipes["Fruit Pie Filling"]!.id,
    ]

    recipes["Pie Crust"]!.related = [recipes["Fruit Pie Filling"]!.id]
    recipes["Fruit Pie Filling"]!.related = [recipes["Pie Crust"]!.id]

    return Array(recipes.values)
}()

let applePie = """
    ¾ cup white sugar
    2 tablespoons all-purpose flour
    ½ teaspoon ground cinnamon
    ¼ teaspoon ground nutmeg
    ½ teaspoon lemon zest
    7 cups thinly sliced apples
    2 teaspoons lemon juice
    1 tablespoon butter
    1 recipe pastry for a 9 inch double crust pie
    4 tablespoons milk
    """

let pieCrust = """
    2 ½ cups all purpose flour
    1 Tbsp. powdered sugar
    1 tsp. sea salt
    ½ cup shortening
    ½ cup butter (Cold, Cut Into Small Pieces)
    ⅓ cup cold water (Plus More As Needed)
    """

struct PushableStack_Previews: PreviewProvider {
    static var previews: some View {
        PushableStack()
    }
}
```

To add programmatic navigation, I need to tease apart.
* value it presents
* view that goes with that value.

1.  Pull destination view out of NavigationLink and into `.navigationDestination` modifier.
	2. Declares the type of presented ata it's responsible for.
	3. Takes a viewbuilder that describes what view to push.
4. Switch to new NavigationLink constructor which takes name/value.

Every navigation stack keeps track of a path that represents all the data.  When stack is just showing top level, path is empty.  Stack also keeps track of all the navigation destinations declared inside it.  In tgeneral, this is a set, although ehre we only have 1 type of destination.

Pushed views.  because the path is empty, so ts the list of pushed views.  When we put these together, we append to the path.  And then navigation stack maps destination over the path values to decide which views to push onto the stack.

So I can use an array of `Recipe` on my path.  For mixed data, check out `NavigationPath` collection to erase types.

Now we pass `NavigationSTack(path: $path)` to use binding.

[[Build a productivity app for Apple Watch]]

## Multicolumn presentation
Consider Mail on Mac,iPad.

```swift
import SwiftUI

// Multiple columns
struct MultipleColumns: View {
    @State private var selectedCategory: Category?
    @State private var selectedRecipe: Recipe?
    @StateObject private var dataModel = DataModel()

    var body: some View {
        NavigationSplitView {
            List(Category.allCases, selection: $selectedCategory) { category in
                NavigationLink(category.localizedName, value: category)
            }
            .navigationTitle("Categories")
        } content: {
            List(
                dataModel.recipes(in: selectedCategory),
                selection: $selectedRecipe)
            { recipe in
                NavigationLink(recipe.name, value: recipe)
            }
            .navigationTitle(selectedCategory?.localizedName ?? "Recipes")
        } detail: {
            RecipeDetail(recipe: selectedRecipe)
        }
    }
}

// Helpers for code example
struct RecipeDetail: View {
    var recipe: Recipe?

    var body: some View {
        Text("Recipe details go here")
            .navigationTitle(recipe?.name ?? "")
    }
}

class DataModel: ObservableObject {
    @Published var recipes: [Recipe] = builtInRecipes

    func recipes(in category: Category?) -> [Recipe] {
        recipes
            .filter { $0.category == category }
            .sorted { $0.name < $1.name }
    }
}

enum Category: Int, Hashable, CaseIterable, Identifiable, Codable {
    case dessert
    case pancake
    case salad
    case sandwich

    var id: Int { rawValue }

    var localizedName: LocalizedStringKey {
        switch self {
        case .dessert:
            return "Dessert"
        case .pancake:
            return "Pancake"
        case .salad:
            return "Salad"
        case .sandwich:
            return "Sandwich"
        }
    }
}

struct Recipe: Hashable, Identifiable {
    let id = UUID()
    var name: String
    var category: Category
    var ingredients: [Ingredient]
    var related: [Recipe.ID] = []
    var imageName: String? = nil
}

struct Ingredient: Hashable, Identifiable {
    let id = UUID()
    var description: String

    static func fromLines(_ lines: String) -> [Ingredient] {
        lines.split(separator: "\n", omittingEmptySubsequences: true)
            .map { Ingredient(description: String($0)) }
    }
}

let builtInRecipes: [Recipe] = {
    var recipes = [
        "Apple Pie": Recipe(
            name: "Apple Pie", category: .dessert,
            ingredients: Ingredient.fromLines(applePie)),
        "Baklava": Recipe(
            name: "Baklava", category: .dessert,
            ingredients: []),
        "Bolo de Rolo": Recipe(
            name: "Bolo de rolo", category: .dessert,
            ingredients: []),
        "Chocolate Crackles": Recipe(
            name: "Chocolate crackles", category: .dessert,
            ingredients: []),
        "Crème Brûlée": Recipe(
            name: "Crème brûlée", category: .dessert,
            ingredients: []),
        "Fruit Pie Filling": Recipe(
            name: "Fruit Pie Filling", category: .dessert,
            ingredients: []),
        "Kanom Thong Ek": Recipe(
            name: "Kanom Thong Ek", category: .dessert,
            ingredients: []),
        "Mochi": Recipe(
            name: "Mochi", category: .dessert,
            ingredients: []),
        "Marzipan": Recipe(
            name: "Marzipan", category: .dessert,
            ingredients: []),
        "Pie Crust": Recipe(
            name: "Pie Crust", category: .dessert,
            ingredients: Ingredient.fromLines(pieCrust)),
        "Shortbread Biscuits": Recipe(
            name: "Shortbread Biscuits", category: .dessert,
            ingredients: []),
        "Tiramisu": Recipe(
            name: "Tiramisu", category: .dessert,
            ingredients: []),
        "Crêpe": Recipe(
            name: "Crêpe", category: .pancake, ingredients: []),
        "Jianbing": Recipe(
            name: "Jianbing", category: .pancake, ingredients: []),
        "American": Recipe(
            name: "American", category: .pancake, ingredients: []),
        "Dosa": Recipe(
            name: "Dosa", category: .pancake, ingredients: []),
        "Injera": Recipe(
            name: "Injera", category: .pancake, ingredients: []),
        "Acar": Recipe(
            name: "Acar", category: .salad, ingredients: []),
        "Ambrosia": Recipe(
            name: "Ambrosia", category: .salad, ingredients: []),
        "Bok l'hong": Recipe(
            name: "Bok l'hong", category: .salad, ingredients: []),
        "Caprese": Recipe(
            name: "Caprese", category: .salad, ingredients: []),
        "Ceviche": Recipe(
            name: "Ceviche", category: .salad, ingredients: []),
        "Çoban salatası": Recipe(
            name: "Çoban salatası", category: .salad, ingredients: []),
        "Fiambre": Recipe(
            name: "Fiambre", category: .salad, ingredients: []),
        "Kachumbari": Recipe(
            name: "Kachumbari", category: .salad, ingredients: []),
        "Niçoise": Recipe(
            name: "Niçoise", category: .salad, ingredients: []),
    ]

    recipes["Apple Pie"]!.related = [
        recipes["Pie Crust"]!.id,
        recipes["Fruit Pie Filling"]!.id,
    ]

    recipes["Pie Crust"]!.related = [recipes["Fruit Pie Filling"]!.id]
    recipes["Fruit Pie Filling"]!.related = [recipes["Pie Crust"]!.id]

    return Array(recipes.values)
}()

let applePie = """
    ¾ cup white sugar
    2 tablespoons all-purpose flour
    ½ teaspoon ground cinnamon
    ¼ teaspoon ground nutmeg
    ½ teaspoon lemon zest
    7 cups thinly sliced apples
    2 teaspoons lemon juice
    1 tablespoon butter
    1 recipe pastry for a 9 inch double crust pie
    4 tablespoons milk
    """

let pieCrust = """
    2 ½ cups all purpose flour
    1 Tbsp. powdered sugar
    1 tsp. sea salt
    ½ cup shortening
    ½ cup butter (Cold, Cut Into Small Pieces)
    ⅓ cup cold water (Plus More As Needed)
    """

struct MultipleColumns_Previews: PreviewProvider {
    static var previews: some View {
        MultipleColumns()
    }
}
```

One cool thing about comining list and navigation split view, is that swiftui can automatically adapt this to a single stack for Iphone etc.  Changes to selection automatically translate into the appropriate pushes and pops.

Also works great on the mac.  And although apple tv/watch don't show columns, they get translation to single stack.

## Two column experience w/ grid.
```swift
import SwiftUI

// Multiple columns with a stack
struct MultipleColumnsWithStack: View {
    @State private var selectedCategory: Category?
    @State private var path: [Recipe] = []
    @StateObject private var dataModel = DataModel()

    var body: some View {
        NavigationSplitView {
            List(Category.allCases, selection: $selectedCategory) { category in
                NavigationLink(category.localizedName, value: category)
            }
            .navigationTitle("Categories")
        } detail: {
            NavigationStack(path: $path) {
                RecipeGrid(category: selectedCategory)
            }
        }
        .environmentObject(dataModel)
    }
}

struct RecipeGrid: View {
    @EnvironmentObject private var dataModel: DataModel
    var category: Category?

    var body: some View {
        if let category = category {
            ScrollView {
                LazyVGrid(columns: columns) {
                    ForEach(dataModel.recipes(in: category)) { recipe in
                        NavigationLink(value: recipe) {
                            RecipeTile(recipe: recipe)
                        }
                    }
                }
            }
            .navigationTitle(category.localizedName)
            .navigationDestination(for: Recipe.self) { recipe in
                RecipeDetail(recipe: recipe)
            }
        } else {
            Text("Select a category")
        }
    }

    var columns: [GridItem] { [GridItem(.adaptive(minimum: 240))] }
}

struct RecipeDetail: View {
    @EnvironmentObject private var dataModel: DataModel
    var recipe: Recipe

    var body: some View {
        Text("Recipe details go here")
            .navigationTitle(recipe.name)
        ForEach(recipe.related.compactMap { dataModel[$0] }) { related in
            NavigationLink(related.name, value: related)
        }
    }
}

struct RecipeTile: View {
    var recipe: Recipe

    var body: some View {
        VStack {
            Rectangle()
                .fill(Color.secondary.gradient)
                .frame(width: 240, height: 240)
            Text(recipe.name)
                .lineLimit(2, reservesSpace: true)
                .font(.headline)
        }
        .tint(.primary)
    }
}

class DataModel: ObservableObject {
    @Published var recipes: [Recipe] = builtInRecipes

    func recipes(in category: Category?) -> [Recipe] {
        recipes
            .filter { $0.category == category }
            .sorted { $0.name < $1.name }
    }

    subscript(recipeId: Recipe.ID) -> Recipe? {
        // A real app would want to maintain an index from identifiers to
        // recipes.
        recipes.first { recipe in
            recipe.id == recipeId
        }
    }
}

enum Category: Int, Hashable, CaseIterable, Identifiable, Codable {
    case dessert
    case pancake
    case salad
    case sandwich

    var id: Int { rawValue }

    var localizedName: LocalizedStringKey {
        switch self {
        case .dessert:
            return "Dessert"
        case .pancake:
            return "Pancake"
        case .salad:
            return "Salad"
        case .sandwich:
            return "Sandwich"
        }
    }
}

struct Recipe: Hashable, Identifiable {
    let id = UUID()
    var name: String
    var category: Category
    var ingredients: [Ingredient]
    var related: [Recipe.ID] = []
    var imageName: String? = nil
}

struct Ingredient: Hashable, Identifiable {
    let id = UUID()
    var description: String

    static func fromLines(_ lines: String) -> [Ingredient] {
        lines.split(separator: "\n", omittingEmptySubsequences: true)
            .map { Ingredient(description: String($0)) }
    }
}

let builtInRecipes: [Recipe] = {
    var recipes = [
        "Apple Pie": Recipe(
            name: "Apple Pie", category: .dessert,
            ingredients: Ingredient.fromLines(applePie)),
        "Baklava": Recipe(
            name: "Baklava", category: .dessert,
            ingredients: []),
        "Bolo de Rolo": Recipe(
            name: "Bolo de rolo", category: .dessert,
            ingredients: []),
        "Chocolate Crackles": Recipe(
            name: "Chocolate crackles", category: .dessert,
            ingredients: []),
        "Crème Brûlée": Recipe(
            name: "Crème brûlée", category: .dessert,
            ingredients: []),
        "Fruit Pie Filling": Recipe(
            name: "Fruit Pie Filling", category: .dessert,
            ingredients: []),
        "Kanom Thong Ek": Recipe(
            name: "Kanom Thong Ek", category: .dessert,
            ingredients: []),
        "Mochi": Recipe(
            name: "Mochi", category: .dessert,
            ingredients: []),
        "Marzipan": Recipe(
            name: "Marzipan", category: .dessert,
            ingredients: []),
        "Pie Crust": Recipe(
            name: "Pie Crust", category: .dessert,
            ingredients: Ingredient.fromLines(pieCrust)),
        "Shortbread Biscuits": Recipe(
            name: "Shortbread Biscuits", category: .dessert,
            ingredients: []),
        "Tiramisu": Recipe(
            name: "Tiramisu", category: .dessert,
            ingredients: []),
        "Crêpe": Recipe(
            name: "Crêpe", category: .pancake, ingredients: []),
        "Jianbing": Recipe(
            name: "Jianbing", category: .pancake, ingredients: []),
        "American": Recipe(
            name: "American", category: .pancake, ingredients: []),
        "Dosa": Recipe(
            name: "Dosa", category: .pancake, ingredients: []),
        "Injera": Recipe(
            name: "Injera", category: .pancake, ingredients: []),
        "Acar": Recipe(
            name: "Acar", category: .salad, ingredients: []),
        "Ambrosia": Recipe(
            name: "Ambrosia", category: .salad, ingredients: []),
        "Bok l'hong": Recipe(
            name: "Bok l'hong", category: .salad, ingredients: []),
        "Caprese": Recipe(
            name: "Caprese", category: .salad, ingredients: []),
        "Ceviche": Recipe(
            name: "Ceviche", category: .salad, ingredients: []),
        "Çoban salatası": Recipe(
            name: "Çoban salatası", category: .salad, ingredients: []),
        "Fiambre": Recipe(
            name: "Fiambre", category: .salad, ingredients: []),
        "Kachumbari": Recipe(
            name: "Kachumbari", category: .salad, ingredients: []),
        "Niçoise": Recipe(
            name: "Niçoise", category: .salad, ingredients: []),
    ]

    recipes["Apple Pie"]!.related = [
        recipes["Pie Crust"]!.id,
        recipes["Fruit Pie Filling"]!.id,
    ]

    recipes["Pie Crust"]!.related = [recipes["Fruit Pie Filling"]!.id]
    recipes["Fruit Pie Filling"]!.related = [recipes["Pie Crust"]!.id]

    return Array(recipes.values)
}()

let applePie = """
    ¾ cup white sugar
    2 tablespoons all-purpose flour
    ½ teaspoon ground cinnamon
    ¼ teaspoon ground nutmeg
    ½ teaspoon lemon zest
    7 cups thinly sliced apples
    2 teaspoons lemon juice
    1 tablespoon butter
    1 recipe pastry for a 9 inch double crust pie
    4 tablespoons milk
    """

let pieCrust = """
    2 ½ cups all purpose flour
    1 Tbsp. powdered sugar
    1 tsp. sea salt
    ½ cup shortening
    ½ cup butter (Cold, Cut Into Small Pieces)
    ⅓ cup cold water (Plus More As Needed)
    """

struct MultipleColumnsWithStack_Previews: PreviewProvider {
    static var previews: some View {
        MultipleColumnsWithStack()
    }
}
```

Lazy contianers dont' load all their views immediately.  If I put the navigationDestination modifier on the link, the destination might not be loaded.  So the surrounding naivgation stack might not see it.

If I put the modifier here, it will be repeated for every item in the grid.  Instead, attach modifier to the scrollview.  By attaching it here, I ensure that the navigation stack can see this regardless of the scroll position.

# Persistent state
Persist the navigation state.  
1.  Codable
2. SceneStorage

1.  Move navigation state into a model type
2. Make the navigation model Codable
3. Use SceneStorage to save and restore

```swift
import SwiftUI
import Combine
import Foundation

// Use SceneStorage to save and restore
struct UseSceneStorage: View {
    @StateObject private var navModel = NavigationModel()
    @SceneStorage("navigation") private var data: Data?
    @StateObject private var dataModel = DataModel()

    var body: some View {
        NavigationSplitView {
            List(
                Category.allCases, selection: $navModel.selectedCategory
            ) { category in
                NavigationLink(category.localizedName, value: category)
            }
            .navigationTitle("Categories")
        } detail: {
            NavigationStack(path: $navModel.recipePath) {
                RecipeGrid(category: navModel.selectedCategory)
            }
        }
        .task {
            if let data = data {
                navModel.jsonData = data
            }
            for await _ in navModel.objectWillChangeSequence {
                data = navModel.jsonData
            }
        }
        .environmentObject(dataModel)
    }
}

// Make the navigation model Codable
class NavigationModel: ObservableObject, Codable {
    @Published var selectedCategory: Category?
    @Published var recipePath: [Recipe] = []

    enum CodingKeys: String, CodingKey {
        case selectedCategory
        case recipePathIds
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encodeIfPresent(selectedCategory, forKey: .selectedCategory)
        try container.encode(recipePath.map(\.id), forKey: .recipePathIds)
    }

    init() {}

    required init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.selectedCategory = try container.decodeIfPresent(
            Category.self, forKey: .selectedCategory)

        let recipePathIds = try container.decode([Recipe.ID].self, forKey: .recipePathIds)
        self.recipePath = recipePathIds.compactMap { DataModel.shared[$0] }
    }

    var jsonData: Data? {
        get {
            try? JSONEncoder().encode(self)
        }
        set {
            guard let data = newValue,
                  let model = try? JSONDecoder().decode(NavigationModel.self, from: data)
            else { return }
            self.selectedCategory = model.selectedCategory
            self.recipePath = model.recipePath

        }
    }

    var objectWillChangeSequence:
        AsyncPublisher<Publishers.Buffer<ObservableObjectPublisher>>
    {
        objectWillChange
            .buffer(size: 1, prefetch: .byRequest, whenFull: .dropOldest)
            .values
    }
}

struct RecipeGrid: View {
    var category: Category?
    @EnvironmentObject private var dataModel: DataModel

    var body: some View {
        if let category = category {
            ScrollView {
                LazyVGrid(columns: columns) {
                    ForEach(dataModel.recipes(in: category)) { recipe in
                        NavigationLink(value: recipe) {
                            RecipeTile(recipe: recipe)
                        }
                    }
                }
            }
            .navigationTitle(category.localizedName)
            .navigationDestination(for: Recipe.self) { recipe in
                RecipeDetail(recipe: recipe)
            }
        } else {
            Text("Select a category")
        }
    }

    var columns: [GridItem] { [GridItem(.adaptive(minimum: 240))] }
}

struct RecipeDetail: View {
    @EnvironmentObject private var dataModel: DataModel
    var recipe: Recipe

    var body: some View {
        Text("Recipe details go here")
            .navigationTitle(recipe.name)
        ForEach(recipe.related.compactMap { dataModel[$0] }) { related in
            NavigationLink(related.name, value: related)
        }
    }
}

struct RecipeTile: View {
    var recipe: Recipe

    var body: some View {
        VStack {
            Rectangle()
                .fill(Color.secondary.gradient)
                .frame(width: 240, height: 240)
            Text(recipe.name)
                .lineLimit(2, reservesSpace: true)
                .font(.headline)
        }
        .tint(.primary)
    }
}

class DataModel: ObservableObject {
    @Published var recipes: [Recipe] = builtInRecipes

    static var shared: DataModel {
        // Just instantiate each time for the example. A real app would need to
        // persist the data model as well.
        DataModel()
    }

    func recipes(in category: Category?) -> [Recipe] {
        recipes
            .filter { $0.category == category }
            .sorted { $0.name < $1.name }
    }

    subscript(recipeId: Recipe.ID) -> Recipe? {
        // A real app would want to maintain an index from identifiers to
        // recipes.
        recipes.first { recipe in
            recipe.id == recipeId
        }
    }
}

enum Category: Int, Hashable, CaseIterable, Identifiable, Codable {
    case dessert
    case pancake
    case salad
    case sandwich

    var id: Int { rawValue }

    var localizedName: LocalizedStringKey {
        switch self {
        case .dessert:
            return "Dessert"
        case .pancake:
            return "Pancake"
        case .salad:
            return "Salad"
        case .sandwich:
            return "Sandwich"
        }
    }
}

struct Recipe: Hashable, Identifiable {
    let id: UUID
    var name: String
    var category: Category
    var ingredients: [Ingredient]
    var related: [Recipe.ID] = []
    var imageName: String? = nil
}

struct Ingredient: Hashable, Identifiable {
    let id = UUID()
    var description: String

    static func fromLines(_ lines: String) -> [Ingredient] {
        lines.split(separator: "\n", omittingEmptySubsequences: true)
            .map { Ingredient(description: String($0)) }
    }
}

let builtInRecipes: [Recipe] = {
    var recipes = [
        "Apple Pie": Recipe(
            id: UUID(uuidString: "E35A5C9C-F1EA-4B3D-9980-E2240B363AC8")!,
            name: "Apple Pie", category: .dessert,
            ingredients: Ingredient.fromLines(applePie)),
        "Baklava": Recipe(
            id: UUID(uuidString: "B95B2D99-F45D-4B74-9EC4-526914FFC414")!,
            name: "Baklava", category: .dessert,
            ingredients: []),
        "Bolo de Rolo": Recipe(
            id: UUID(uuidString: "E17C729D-1E09-48F6-99E2-5BB959F5AE70")!,
            name: "Bolo de Rolo", category: .dessert,
            ingredients: []),
        "Chocolate Crackles": Recipe(
            id: UUID(uuidString: "89202A12-2B04-4EFE-ADC5-D1ECE7A25389")!,
            name: "Chocolate Crackles", category: .dessert,
            ingredients: []),
        "Crème Brûlée": Recipe(
            id: UUID(uuidString: "412EA92A-40B5-4CFE-9379-627A1C80FFE1")!,
            name: "Crème Brûlée", category: .dessert,
            ingredients: []),
        "Fruit Pie Filling": Recipe(
            id: UUID(uuidString: "4792C8AE-9596-4502-A9CB-806E2DFEA408")!,
            name: "Fruit Pie Filling", category: .dessert,
            ingredients: []),
        "Kanom Thong Ek": Recipe(
            id: UUID(uuidString: "331C25F6-4FED-4DA5-980E-7E619855DE92")!,
            name: "Kanom Thong Ek", category: .dessert,
            ingredients: []),
        "Mochi": Recipe(
            id: UUID(uuidString: "1EAA5288-8D2B-4969-AF97-ED591796B456")!,
            name: "Mochi", category: .dessert,
            ingredients: []),
        "Marzipan": Recipe(
            id: UUID(uuidString: "416F4F5A-A81C-40FD-87F1-060B0F57DE6D")!,
            name: "Marzipan", category: .dessert,
            ingredients: []),
        "Pie Crust": Recipe(
            id: UUID(uuidString: "D0820C1A-1AFB-4472-97DA-39A475304048")!,
            name: "Pie Crust", category: .dessert,
            ingredients: Ingredient.fromLines(pieCrust)),
        "Shortbread Biscuits": Recipe(
            id: UUID(uuidString: "3D9FEA8C-B38E-4739-8B4B-424885D76926")!,
            name: "Shortbread Biscuits", category: .dessert,
            ingredients: []),
        "Tiramisu": Recipe(
            id: UUID(uuidString: "586B9A4C-410A-40D2-AE40-BC32351A5C08")!,
            name: "Tiramisu", category: .dessert,
            ingredients: []),
        "Crêpe": Recipe(
            id: UUID(uuidString: "9BD6C3B2-30CB-425E-8D60-7F07D0BA720C")!,
            name: "Crêpe", category: .pancake,
            ingredients: []),
        "Jianbing": Recipe(
            id: UUID(uuidString: "117E5CD4-8FF9-43FB-ACAE-53C35A648F6F")!,
            name: "Jianbing", category: .pancake,
            ingredients: []),
        "American": Recipe(
            id: UUID(uuidString: "4584B877-E482-4FF2-824E-FC667BFAD271")!,
            name: "American", category: .pancake,
            ingredients: []),
        "Dosa": Recipe(
            id: UUID(uuidString: "5666FEB6-90DB-4CD2-91FA-D6F00986E90E")!,
            name: "Dosa", category: .pancake,
            ingredients: []),
        "Injera": Recipe(
            id: UUID(uuidString: "752DAEB8-123E-4C48-A190-79742AA56869")!,
            name: "Injera", category: .pancake,
            ingredients: []),
        "Acar": Recipe(
            id: UUID(uuidString: "F0D54AF2-04AD-4F08-ACE4-7886FCAE1F7B")!,
            name: "Acar", category: .salad,
            ingredients: []),
        "Ambrosia": Recipe(
            id: UUID(uuidString: "F7FD59E8-F1AE-4331-8667-D5534817F7E7")!,
            name: "Ambrosia", category: .salad,
            ingredients: []),
        "Bok L'hong": Recipe(
            id: UUID(uuidString: "3DE38C07-F985-4E05-810C-1108A777766B")!,
            name: "Bok L'hong", category: .salad,
            ingredients: []),
        "Caprese": Recipe(
            id: UUID(uuidString: "055D963C-0546-4578-AF18-6FBEE249EF35")!,
            name: "Caprese", category: .salad,
            ingredients: []),
        "Ceviche": Recipe(
            id: UUID(uuidString: "50B62AF4-89AF-4D00-9832-E200FEC01279")!,
            name: "Ceviche", category: .salad,
            ingredients: []),
        "Çoban Salatası": Recipe(
            id: UUID(uuidString: "87AD6B33-FFD2-4E5C-BC4B-59769F7AC7E3")!,
            name: "Çoban Salatası", category: .salad,
            ingredients: []),
        "Fiambre": Recipe(
            id: UUID(uuidString: "8A9BC0D5-A931-4381-BDA8-713DF6389FE7")!,
            name: "Fiambre", category: .salad,
            ingredients: []),
        "Kachumbari": Recipe(
            id: UUID(uuidString: "E9497D38-49E0-4A18-939B-63A3F2C7C0B4")!,
            name: "Kachumbari", category: .salad,
            ingredients: []),
        "Niçoise": Recipe(
            id: UUID(uuidString: "DE9F7106-4D0C-4EAC-B44C-A8D8ECD81087")!,
            name: "Niçoise", category: .salad,
            ingredients: [])
    ]

    recipes["Apple Pie"]!.related = [
        recipes["Pie Crust"]!.id,
        recipes["Fruit Pie Filling"]!.id
    ]

    recipes["Pie Crust"]!.related = [recipes["Fruit Pie Filling"]!.id]
    recipes["Fruit Pie Filling"]!.related = [recipes["Pie Crust"]!.id]

    return Array(recipes.values)
}()

let applePie = """
    ¾ cup white sugar
    2 tablespoons all-purpose flour
    ½ teaspoon ground cinnamon
    ¼ teaspoon ground nutmeg
    ½ teaspoon lemon zest
    7 cups thinly sliced apples
    2 teaspoons lemon juice
    1 tablespoon butter
    1 recipe pastry for a 9 inch double crust pie
    4 tablespoons milk
    """

let pieCrust = """
    2 ½ cups all purpose flour
    1 Tbsp. powdered sugar
    1 tsp. sea salt
    ½ cup shortening
    ½ cup butter (Cold, Cut Into Small Pieces)
    ⅓ cup cold water (Plus More As Needed)
    """

struct UseSceneStorage_Previews: PreviewProvider {
    static var previews: some View {
        UseSceneStorage()
    }
}
```

I don't want to store the entire model value for state restoration
1.  My recipe database already contains all the details, don't want to repeat them here.
2. If my database can change independently of my navigation state, I don't want my local navigation state to contain stale data.

SceneStorage automatically save/restore the associated values.  When system restores the scene, swiftui ensures this is restored.  Take advantage of this to persist my navigation model.

`.task` runs asynchronously.  STarts when the view appears, cancelled when the view goes away.  

Tips:
Adopt the new navigation APIs.
`.navigationViewStyle(.stack)` => NavigationStack
NavigationView (multicolumn) => NavigationSplitView
programmatic navigation(...isActive:...) etc.  => NavigationLink(value:) with navigation paths and list selection.  This situation is deprecating.

See article "Migrating to new navigation types".

Compose `NavigationSplitView`, `NavigationStack`, and `List`.  
Put `navigationDestination` modifiers within easy reach.  But not in lazy contianers.
Start with `NavigationSplitView` when it makes sense.

# Wrap up
[[Bring multiple windows to your SwiftUI app]]

This seems unused:


```swift
import SwiftUI

struct Biscuits: View {
    @State private var step = 0
    @ScaledMetric private var fontSize = 18

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Spacer()
                VStack {
                    Text("Biscuits")
                        .font(.headline)
                    Text(subtitle)
                        .font(.subheadline)
                }
                .padding(16)
                Spacer()
            }
            Spacer()
            Text(LocalizedStringKey(steps[step]))
                .font(.system(
                    size: fontSize, weight: .semibold, design: .serif))
                .padding(16)
                .lineLimit(1...)
            Spacer()
            HStack {
                Button {
                    withAnimation {
                        step -= 1
                    }
                } label: {
                    Label("Previous", systemImage: "chevron.backward")
                }
                .disabled(step - 1 < 0)

                Spacer()

                Button {
                    withAnimation {
                        step += 1
                    }
                } label: {
                    Label("Next", systemImage: "chevron.forward")
                }
                .disabled(step + 1 >= steps.count)
            }
            .buttonStyle(CarouselButtonStyle())
            .padding(16)
        }
        .foregroundStyle(Color.white)
        .background(gradient)
        .ignoresSafeArea(edges: .bottom)
    }

    var subtitle: LocalizedStringKey {
        if step == 0 { return "Ingredients" }
        return "Step \(step)"
    }

    var gradient: AngularGradient {
        AngularGradient(
            colors: colors,
            center: UnitPoint(x: 0.5, y: 1.0),
            angle: .degrees(180 * Double(step) / Double(steps.count - 1)))
    }

}

struct CarouselButtonStyle: ButtonStyle {
    @Environment(\.isEnabled) private var isEnabled

    func makeBody(configuration: Configuration) -> some View {
        ZStack {
            Circle()
                .fill(.ultraThinMaterial.shadow(.inner(
                    radius: configuration.isPressed ? 3 : 0)))
                .frame(width: 44, height: 44)
            configuration.label
                .labelStyle(.iconOnly)
                .foregroundStyle(isEnabled ? .black : .secondary)
                .opacity(configuration.isPressed ? 0.3 : 0.8)
        }
    }
}

let steps = [
    """
    2 cups all-purpose flour
    ¼ teaspoons coarse salt
    1 cup (2 sticks) unsalted butter, room temperature
    ¾ cup confectioners' sugar
    """,
    "Sift flour and salt, mix into bowl and set aside.",
    "Mix butter on high speed until fluffy (3 to 5 minutes).",
    "Gradually add sugar slowly, continuing to mix until pale and fluffy.",
    "Add flour all at once and mix until combined.",
    "Butter a square pan.",
    "Pat and roll shortbread into pan no more than 1/2-inch thick.",
    "Refrigerate for at least 30 minutes.",
    "Preheat oven to 300 F.",
    "Cut chilled shortbread into squares.",
    """
    Bake until golden and make sure the middle is firm. \
    Approximately 45 to 60 minutes.
    """,
    "Cool completely. Re-slice them, if necessary, and serve.",
]

let colors = [Color.yellow, .red, .purple]

struct Biscuits_Previews: PreviewProvider {
    static var previews: some View {
        Biscuits()
    }
}
```



* https://developer.apple.com/forums/tags/wwdc2022-10054
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=521030
* https://developer.apple.com/documentation/SwiftUI/List
* https://developer.apple.com/documentation/SwiftUI/Migrating-to-New-Navigation-Types
* https://developer.apple.com/documentation/SwiftUI/NavigationSplitView
* https://developer.apple.com/documentation/SwiftUI/NavigationStack
* https://developer.apple.com/documentation/swiftui/bringing_robust_navigation_structure_to_your_swiftui_app

