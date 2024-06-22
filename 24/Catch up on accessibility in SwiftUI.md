SwiftUI makes it easy to build amazing experiences that are accessible to everyone. We'll discover how assistive technologies understand and navigate your app through the rich accessibility elements provided by SwiftUI. We'll also discuss how you can further customize these experiences by providing more information about your app's content and interactions by using accessibility modifiers.

###  7:27 - Accessibility Label Modifier & Opacity
```swift
struct UnreadIndicatorView: View {
    var isUnread: Bool
    
    var body: some View {
        Circle()
            .foregroundStyle(.blue)
            .accessibilityLabel("Unread")
            .opacity(isUnread ? 1 : 0)
    }
}
```

###  8:02 - Accessibility Element Children Combine Modifier
```swift
var body: some View {
    HStack {
        UnreadIndicatorView(isUnread: message.isUnread)
        MessageContentsView(message: message)
        Spacer()
        Button(action: favorite) {
            favoriteLabel
        }
        Button(action: reply) {
            replyLabel
        }
    }
    .accessibilityElement(children: .combine)
}
```

###  10:37 - Accessibility Conditional Modifiers
```swift
var body: some View {
    Button(action: favorite) {
        Image(systemName: isSuperFavorite ? "sparkles" : "star.fill")
    }
    .accessibilityLabel("Super Favorite", isEnabled: isSuperFavorite)
}
```

###  13:38 - Accessibility Actions Modifier
```swift
var body: some View {
    TripView(trip: trip)
        .onHover { showAttachments = $0 }
        .overlay {
            MessageAttachments(attachments: trip.attachments)
                .opacity(showAttachments ? 1 : 0)
        }
        .accessibilityActions {
            MessageAttachments(attachments: trip.attachments)
        }
}
```

###  15:16 - Accessibility Label Modifier
```swift
var body: some View {
    TripView(trip: trip)
        .accessibilityLabel { label in
            if let rating = trip.rating {
                Text(rating)
            }
            label
        }
}
```

###  17:42 - Accessibility Drop Point Modifier
```swift
var body: some View {
    CommentAlertView(contact: contact)
        .onDrop(of: [.audio], delegate: delegate)
        .accessibilityDropPoint(.leading, description: "Set Sound 1")
        .accessibilityDropPoint(.center, description: "Set Sound 2")
        .accessibilityDropPoint(.trailing, description: "Set Sound 3")
}
```

###  19:45 - Accessibility App Intent Action Modifier
```swift
var body: some View {
    ForEach(beaches) { beach in
        BeachView(beach)
            .accessibilityAction(named: "Favorite", intent: ToggleRatingIntent(beach: beach, rating: .fullStar))
            .accessibilityAction(.magicTap, intent: ComposeIntent(type: .photo))
    }
}
```
# Resources
* https://developer.apple.com/documentation/Updates/Accessibility
* https://developer.apple.com/documentation/Accessibility/enhancing-the-accessibility-of-your-swiftui-app
* https://developer.apple.com/documentation/Accessibility/performing-accessibility-testing-for-your-app
