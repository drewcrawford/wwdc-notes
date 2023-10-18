#accessibility 
Learn how Assistive Access can help people with cognitive disabilities more easily use iPhone and iPad. Discover the design principles that guide Assistive Access and find out how the system experience adapts to lighten cognitive load. We'll show you how Assistive Access works and what you can do to support this experience in your app.

Cognitive disabilities present their own unique set of challenges.  Assistive access distills apps and their experiences to lighten cognitive load to people with disabilities.  

make it easy to navigate and use apps with greater independence than before.

# Overview
"Trusted supporters"

* customizable interface
* easy-to-add preferred apps

Lock screen
* customizable wallpapers
* quick look at notifications

home screen
* large icons
* large text

apps
calls
messages
music
camera
photos

# Principles
* streamlined task completion
* error prevention and recovery
* consistency

# Your app
Third-party apps will just work.  In this setup, I've added my app.

* large back button displayed prominently at the bottom
* reduced frame

# Optimized for Assistive Access
We've created a new info.plist key to allow your apps to run in full screen excluding the back ubutton

`UISupportsFullScreenInAssistiveAccess`.
Use this key to let assistive access know you've designed your app to adapt to arbitrary screen sizes.

Great apps on iPhone/iPad have consistent and adaptive layout.  Your app does not hardcode layout based on the device or screen size.

SwiftUI - hstack, vstack, Grid, GridRow.
alignmentGuide, safeAreaInset.
[[Compose custom layouts with SwiftUI]]

UIKit
* `.safeAreaInsets`
* `.safeAreaLayoutGuide`.

[[UIKit apps for every size and shape - 18]]

New info plist key ensures all users in assistive access have a great experience.

# Next steps
* turn on assistive access
* evaluate your app
* adopt fullscreen if possible

# Resources

* https://developer.apple.com/documentation/bundleresources/information_property_list/uisupportsfullscreeninassistiveaccess
