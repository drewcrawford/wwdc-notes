SwiftUI makes it easy to build amazing experiences that are accessible to everyone. We'll discover how assistive technologies understand and navigate your app through the rich accessibility elements provided by SwiftUI. We'll also discuss how you can further customize these experiences by providing more information about your app's content and interactions by using accessibility modifiers.

Bring those experiences to life across apple platforms.

# Fundamentals

Creates accessibility elements as one of its primary outputs.  AX elements are the fundamental building block that AX technologies like VO, VC, switch control use.

An accessibility element represents one or more views, and provides attributes to describe contents of the view, and actions which expose how the view can be interacted with, from taps to complex gestures.

VO - only interact with apps through their elements, so only content from views that are included in an element will be accessible.

When VO focuses on the toggle, element is described, double tap will perform press action.

SwiftUI combines the text and toggle into a single element.  Multiple views can be represented by a single element to simplify navigation and collect relevant info together.

When customizing look/feel, preserving the attributes and actions are important.  SwiftUI's powerful view styling system is the key to change how views visually appear without losing builtin AX support.

When changing a style using the toggle style modifier, the visuals will update, but its element keeps label and traits.  When a custom toggle is tapped, its elements and attributes reflect new state of the toggle.

View styles are supported across various controls.  Controls, grouping, indicators.  

Using styles and builtin views will provide a good ax experience in your app.

Accessibility modifiers let you customize how a view is represented by an AX element.  Attributes can be added.  Actions can be exposed.  Modifiers let you combine elements together.

Before using modifiers, the best way to understand where you can improve is by using it with e.g. VO.  

Reduces swipes to access content
Clarify relationships between content and controls
Describe decorative views

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

When its opacity is 0, SwiftUI will automatically hide the element from AX.  To simplify navigating the comments view I can make each stack a single element.

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

Actions like buttons are turned into custom actions.  

# View accessibility
While SwiftuI has done the heavy lifting, providing more info can fix missing content and provide a more enjoyable experience.

When I double-tap on a favorite, it becomes a super favorite.  use a new symbol to indicate how important they are.  New problem: before my combined actioned for favoriting was correctly labeled.  But now it announces the sparkle name for super favorites.


When using symbols in view, many provide default labels where a meaning can be inferred.  Others need an explicit label to be added if they symbolize content, and fall back to their names as the label.  Always verify labels!

You can now add an isEnabled parameter to accessibility modifiers.  When enabled, the attribute is applied, when not enabled it doesn't go into effect.

###  10:37 - Accessibility Conditional Modifiers
```swift
var body: some View {
    Button(action: favorite) {
        Image(systemName: isSuperFavorite ? "sparkles" : "star.fill")
    }
    .accessibilityLabel("Super Favorite", isEnabled: isSuperFavorite)
}
```

A great opportunity to explore these enhancements are when views dynamically appear.  Hover, key press, gesture.

I've been workign on bringing my app to macOS.  Great interaction to add: hover.  When hovering over trip imageview, attachment buttons including location, recording, beaches rating appears.  This removes clutter, etc.

When moving a mouse cursor, view appears and the interaction can be performed in seconds.  But VO has to do a lot of steps to navigate ot this view.  Hoverable view has to be focuse,d hove rhas to be invoked with command, subview navigated to, focus returns to original view.

Appending AX attributes and actions of the hoverable view as part of the main view.  Not every trip may have recording or location attached, so the exact number of actions/labels is not fixed.

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

VO can now perform each attachment without having to present the overlay.

Rating appears on hover, but it's a centerpiece of my trip.  So how to make more prominent

* append labels to simplify interactions
* Provide labels for decorative views

A regular label wipes out swiftui defaults.  Instead, we can set `.accessibilityLabel` modifier.  

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

# Enhanced integration
Remember to design great experiences for alternative forms of input.

Drag and drop.  AX technologies like VO and VC work with dnd.  

Now VO can access each drop point on my alert view.  
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

This lets us describe drop points.  

Try drag and drop experiences with accessibility technologies.  
Consider providing alternative custom actions.

Expose unique drag and drop points for each interaction.
Interactions can expand outside the bounds of your app into interactive widgets.  AX modifiers won't update in real time.

App intents allow widgets to create interactive views.  But there may be places to elevate experience through additional custom actions.


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

AX action modifiers now accept AppIntent.  When invoked will perform the intent and update your widget.  Here I've added a custom action, to set a beach as my favorite beach.

Magic tap action -> take a photo with doubletap.

# Next steps
* try your app with VO
* Investigate accessibility refinements
* Explore AX samples

# Resources
* https://developer.apple.com/documentation/Updates/Accessibility
* https://developer.apple.com/documentation/Accessibility/enhancing-the-accessibility-of-your-swiftui-app
* https://developer.apple.com/documentation/Accessibility/performing-accessibility-testing-for-your-app
