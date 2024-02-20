Dive into the details of watchOS design principles and learn how to apply them in your app using SwiftUI. We'll show you how to build an app for the redesigned user interface to surface timely information, communicate focused content at a glance, and make navigation consistent and predictable.

For watchOS 10, we redesigned the UX to surface timely information, communicate focused content at a glance, celebrate the shape and fidelity of the display, and make navigation consistent and predictable.

SwiftUI.

# Design principles

Timely and relevant.
Weather achieves an ‘apple watch moment’.  Try to narrow the focus of experience to what’s relevant at the time you raise your wrist

Focused and highly specialized
If you had 10 seconds, what would you surface?  
Example: news has 5 top stories.
Example: heart rate displays finite data first.
Example: activity has single focused views.  Keep it brief and focused.

Unique to Apple Watch - crown.
While iteractions should be anchored to the crown, back them up by touch.
Shape your app’s navigation structure and experience.

Consider the journey
Smart Stack. Glanceable widgets swing onto the screen in an intelligently ordered stack.  When you design your app, thinks bout which information will make the best Smart Stack widget and design around those experiences.
[[Design widgets for the Smart Stack on Apple Watch]]



# Navigation

`NavigationSplitView` provides detailed content

`TabView`

`NavigationStack`

Each has its strengths.

## NavigationSplitView

Borrowed from the two column layout on iPadOS.  On watchOS< the source list and the detail are separated, and then recombined, with the source list tucked beneath the detail view.

Perfect option if you have a source list, like cities or stocks.  And detailed views of the items in the source list.  Land people directly in the detail view when they launch your app.  Use context clues to pick the right item.

Make your detail view so unmistakable that it doesn’t need a title.

Tap list icon to go to source list, or just swipe left.

No title, etc.  Always initialize the selection to a value.

## TabView
Fantastic updates.  My new favorite way to navigate apps.

Scroll between views, and in watchOS10, a single tab can expand and resize as needed.

Activity has a tab for each of the ring detail views. Vertical page style.  Also has a new transition between each tab.

TabView will automatically detect scrolling content, expanding to accommodate it.

Animation.  Tie interactions to the digital crown while also keeping rings in view.

watchOS 10, drive animations based on the selection value of the tab view.  Hook this up to a matched geometry effect.

## NavigationStack
If your app can’t do what it needs to do by pivtoting between detail/source, or in vertically paginated tabs, use a navigation stack.

Workouts, calendar, music.

We changed animations.

Clearest way to lead people into a do ut of a hierarchy of views.  Use a large title on the first view but not on any sub view where a back button is present.

Choose a navigation structure that accomplishes your Apple Watch moment in as few interactions as possible.

# Layout

Display has gotten consistently larger and more rounded.  When designing for a rounded display, we’ve developed a flexible grid system that defines the size and placement of controls, label,s and content.

To accommodate different types of content, we designed 3 foundational layouts
* dial
* Infographic
* List

These layouts are designed to adapt automatically to the different Apple Watch sizes supported by watchOS 10.

## dial
Dense information delivered at a glance. Use of full-screen color and imagery, weather conditions,e tc., help add additional context

Up to 4 different corner controls without obscuring content.

Based on shape of active display area.  Curvature of display informs size/position of elements.  Available in apple design resources.

I hope you find them useful in designing your app.

When you use ex topBarTrailing, time moves away, etc.

NowPlayingView adds control buttons ot bottom bar.  No need to add any additional padding.

Make the experience of Apple Watch more predictable across apps.  Inspire you to design great apps for many years to come.

### Dial Based View - 13:26
```swift
// This is an example of using scene padding to position a Circle according
// to the Dial layout grid
struct DialBasedView: View {
    var body: some View {
        ZStack {
            // Add a view to make the ZStack fill the available width, allowing the
            // Circle to position correctly. As an example, we use a rectangle.
            Rectangle()
                .foregroundStyle(Color.clear)

            // Use .scenePadding(.horizontal) on the dial to get the correct
            // width. In a ZStack with centered alignment, it is correctly
            // positioned.
            Circle()
                .foregroundStyle(Color.red)
                .scenePadding(.horizontal)
        }
        // Ignore vertical safe areas to allow the view to draw into the bottom
        // safe areas. This achieves the centering for the dial.
        .edgesIgnoringSafeArea(.vertical)
    }
}
```

# Color and materials

Emphasize the shape and fidelity of the display with fullscreen color and images.  Four vibrant full-screen background materials.

* ultra thin
* Thin
* Regular
* Thick
Different blur levels.  Use the same SwiftUI apis as you already do on other platforms.

Full-screen background gradient with your own color.  See, activity app for details.

Also blue background for sleep.  

Use fullscreen color to convey a state change. Bright orange, etc.

To make sure foreground elements look great, we added vibrant fill materials to controls and platter cells.  Vibrant text labels in primary,s Econ dary, tertiary, etc. Prominence levels for information hierarchy.

Presentations now use a full screen thin material.  Gives the wearer additional context.

# Wrap up
* use the watchOS 10 refresh to design or redesign your Apple Watch app for clear, focused andtimely interactions

[[Meet watchOS 10]]
[[Design widgets for the Smart Stack on Apple Watch]]
[[Update your app for watchOS 10]]

Only possible because of a deep collaboration between design and engineering. Never been a more vibrant platform and never been a better time to design and built your app.



# Resources
* https://developer.apple.com/design/resources/
* https://developer.apple.com/design/Human-Interface-Guidelines/designing-for-watchos
