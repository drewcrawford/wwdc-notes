Explore new opportunities on Apple Watch, including bringing Double Tap support to your watchOS app, making your Smart Stack widgets even more relevant and interactive, and displaying your iOS Live Activities in the Smart Stack.

I'll be talking about how you can take advantage of these same features in your apps.  

# Live activities

A live activity provides a glanceable way to track progress of event or task from the lock screen.  In watchOS 11, live activities from your iOS app are available in the smart stack on apple watch even if you don't have an app yet.  With no changes at all, your leading and trailing views from the dynamic island will automatically appear in the smart stack.

Specify that your live activity supports small supplemental activity family.  System wil prefer your custom content view over dynamic island views.

Use the Environment to further customize layout.  small family -> smart stack.  medium family -> lock screen on iOS and iPadOS.

[[Bring your Live Activity to Apple Watch]]
[[Design Live Activities for Apple Watch]]
# Widgets

Many new features in widgets.
* relevant widgets

Provide system with context clues

`RelevantContext` API.  
* date
* location
* Sleep (including bedtime/wakeup time)
* fitness (active workout, incomplete activity rings, etc)

You can give relevance per intent.  Each intent represents someone's favorited store location.  Can provide location-relevant context for each location.

* Useful and actionable
* Relevantcontext is used to determine the priority
* System considers many suggestions

## interactive widgets

In iOS 17/macOS 14, we introduced interactive widgets, allowing people to perform actions right from a widget without having to open the app.  Bring your interactive widget to watchOS 2.  Buttons and toggles are available to make widget interactive.

Home widget - directly lock/unlock the door.  Interactivity makes it possible to perform quick actions without launching an app.

Available for all watchOS widget families.  Enables multiple interactions in the same widget.

[[Bring widgets to life]]

## AccessoryWidgetGroup

In watchOS 11, many new widgets using this layout.  Messages shows top 3 pinned contacts.
Two main components: label and content.  By default, label will have a display name of the app's widget extension bundle.  We encourage you to provide a custom label or text view.

Content - up to 3 views.  We only show the first 3 if you provide more.  Can deep link, etc.

Focus on the most important content for your widget.  Fewer than 3 pinned contact,s system will automatically insert extra empty views.  When tapped, empty views will simply launch the app.  System provides the color of the empty views.  This color is not configurable.

Can specify the masking shape.  circular or roundedSquare.  Default is circular.

containerBackground - tint.  
foregroundStyle - tint label

# Double tap

Series 9 introduced a brand new way to perform common actions such as answering the phone, pausing music, or scrolling through smartstack.  In watchOS 11, double tap is expanded to work in more places.  Lists, scrollview, tabview.  Your app gets this behavior automatically.

Identify a primary action to perform with double tap.  For example, activating a button or toggle in your app.  

`.handGestureShortcut(.primaryAction)`.  Can customize the shape of the highlight using clip shape view modifier.  Only one element can be the primary action so consider your app's interface and usecase carefully.

Control must be on screen.

Consistent and predictable.

# Health and fitness

Custom pool swimming
distanceWithTime goal type.  Both distance and time goal.

Can now customize step names.
[[Build custom swimming workouts with WorkoutKit]]


State of Mind API.  
[[Explore wellbeing APIs in HealthKit]]

# Wrap up

* customize your live activity
* Make your widgets relevant and interactive
* Take action with double tap
* Explore new health and fitness possibilities

[[What’s new in SwiftUI]]


# Resources
* https://developer.apple.com/documentation/watchOS-Apps/enabling-double-tap
* https://developer.apple.com/design/human-interface-guidelines/designing-for-watchos
* https://developer.apple.com/design/human-interface-guidelines/widgets
* https://developer.apple.com/documentation/WidgetKit