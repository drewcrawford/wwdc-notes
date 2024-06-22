Learn how to provide semantic search results in your app using Core Spotlight. Understand how to make your app's content available in the user's private, on-device index so people can search for items using natural language. We'll also share how to optimize your app's performance by scheduling indexing activities. To get the most out of this session, we recommend first checking out Core Spotlight documentation on the Apple Developer website.

### Creating CSSearchableItem - 2:14
```swift
// Creating searchable items for donation
let item = CSSearchableItem(uniqueIdentifier: uniqueIdentifier, domainIdentifier: domainIdentifier, attributeSet: attributeSet)
```

### Creating CSSearchableAttributeSet - 2:28
```swift
// Creating searchable content for donation
let attributeSet = CSSearchableItemAttributeSet(contentType: UTType.text)
attributeSet.contentType = UTType.text.identifier
```

### Searchable items with type - 2:40
```swift
// Searchable items with text
attributeSet.title
attributeSet.textContent

// Searchable items with media
attributeSet.contentType
attributeSet.contentURL

// Searchable items with links
attributeSet.contentURL
attributeSet.relatedUniqueIdentifier
```

### Batch indexing with client state - 3:31
```swift
// Batch indexing with client state
let index = CSSearchableIndex(name: "SpotlightSearchSample")
index.fetchLastClientState { state, error in
    if state == nil {
        index.beginBatch()
        index.indexSearchableItems(items)
        index.endIndexBatch(expectedClientState: state, newClientState: newState) { error in
        }
    }
}
```

### Avoid overwriting existing attributes - 3:56
```swift
// Make it an update to avoid overwriting existing attributes
item.isUpdate = true
```

### Configure a query - 7:19
```swift
// Configure a query
let queryContext = CSUserQueryContext()
queryContext.fetchAttributes = ["title", "contentDescription"]
```

### Ranked results - 7:33
```swift
// Ranked results
queryContext.enableRankedResults = true
queryContext.maxRankedResultCount = 2
```

### Suggestions - 7:47
```swift
// Suggestions
queryContext.maxSuggestionCount = 4
```

### Filter queries - 7:55
```swift
// Filter queries
queryContext.filterQueries = ["contentTypeTree=\"public.image\""]
```

### Query for searchable items and suggestions - 8:23
```swift
// Query for searchable items and suggestions
let query = CSUserQuery(userQueryString: "windsurfing carmel", userQueryContext: queryContext)
for try await element in query.responses {
    switch(element) {
    case .item(let item):
        self.items.append(item)
        break
    case .suggestion(let suggestion):
        self.suggestions.append(suggestion)
        break
    }
}
```

### Suggestions - 8:40
```swift
// Suggestions
suggestion.localizedAttributedSuggestion
```

### Preparing for queries - 8:56
```swift
// Preparing for queries
CSUserQuery.prepare
CSUserQuery.prepareWithProtectionClasses
```

### Set the lastUsedDate - 9:50
```swift
// Set the lastUsedDate when the user interacts with the item
item.attributeSet.lastUsedDate = Date.now
item.isUpdate = true
```

### Interactions with items and suggestions from a query - 10:00
```swift
// Interactions with items and suggestions from a query
query.userEngaged(item, visibleItems: visibleItems, interaction: CSUserQuery.UserInteractionKind.select)
query.userEngaged(suggestion, visibleSuggestions: visibleSuggestions, interaction: CSUserQuery.UserInteractionKind.select)
```

# Resources
* https://developer.apple.com/documentation/CoreSpotlight
* 