#swiftui 

# Search

```swift
ContentView()
    .searchable(text: $text)
```

Mark view content as being searchable.  Content defiens what that means.

When viewing cities, start typing into the search bar.

```swift
NavigationView {
    WeatherList(text: $text) {
        ForEach(data) { item in
            WeatherCell(item)
        }
    }
}
.searchable(text: $text)
```

Takes the configured search field and passes down through the environment for other views to use.

Here, navigation view understands and renders as search bar.  If no views use the field, searchable modifier provides a default implementation.

Whenever weather sees a non-empty search query it displays another list.  

How can it use information to achieve ?

```swift
struct WeatherList: View {
    @Binding var text: String
    
    @Environment(\.isSearching)
    private var isSearching: Bool
    
    var body: some View {
        WeatherCitiesList()
            .overlay {
                if isSearching && !text.isEmpty {
                    WeatherSearchResults()
                }
            }
    }
}
```

When rendering your own results, consider using an overlay.  
# NavigationView
As people started using the app, I began to notice that people love colors.

How to use searchable to  implement a feature?

```swift
struct ColorsContentView: View {
    @State var text = ""
    
    var body: some View {
        NavigationView {
            Sidebar()
            DetailView()
        }
        .searchable(text: $text)
    }
}
```

Provide it with a binding to the state.  This will be rendered as as a search bar.

Search bar will be associated with sidebar.  If you want another column, place modifier on the desired column.

Display search results based on `isSearching`.

Notice that in each case, `.searchable()` placed the search field into the right place.  SwiftUI understands the structure and handles implementing them.

tvOS typically renders search as a tab in a tabview.  So I can add one.

```swift
struct ColorsContentView: View {
    @State var text = ""
    
    var body: some View {
        NavigationView {
            #if os(tvOS)
            TabView {
                Sidebar()
                ColorsSearch()
                    .searchable(text: $text)
            }
            #else
            Sidebar()
            DetailView()
            #endif
        }
        #if !os(tvOS)
        .searchable(text: $text)
        #endif
    }
}
```

Rely on SwiftUI and let the implementation of the searchable modifier pick the appropriate interface.  On tvOS, I took what I leanred about the modifier and applied it to a different structure.


# Suggestions

After using search, several users reported they want suggestions.

SwiftUI adds search suggestions.

```swift
struct ColorsContentView: View {
    @State var text = ""
    
    var body: some View {
        NavigationView {
            Sidebar()
            DetailView()
        }
        .searchable(text: $text) {
           ForEach(suggestions) { suggestion in
                ColorsSuggestionLabel(suggestion)
                    .searchCompletion(suggestion.text)
            }
        }
    }
}
```

Optional parameter called "suggestions".  Inside the lcosure, we supply a view.  e.g. a `ForEach`.  SwiftUI will present this based on if there are any suggestions.  

Can use searchCompletion modifier

```swift
struct ColorsContentView: View {
    @State var text = ""
    
    var body: some View {
        NavigationView {
            Sidebar()
            DetailView()
        }
        .searchable(text: $text) {
           ForEach(suggestions) { suggestion in
                ColorsSuggestionLabel(suggestion)
                    .searchCompletion(suggestion.text)
            }
        }
    }
}
```

Will convert the view into a button that dismisses the search and uses the text.

Consider using `.onSubmit` to know when to fetch results.  Pass in `search`

```swift
ContentView()
    .searchable(text: $text) {
        MySearchSuggestions()
    }
    .onSubmit(of: .search) {
        fetchResults()
    }
```

`onsubmit` can be used with textfield or secure fields.  Suggestions parameter provides an easy way to add powerful search functionality.

# Wrap up
* Searchable modifier
* Integration with NavigationView
* Adjust with `isSearching` from environment
* Search suggestions
