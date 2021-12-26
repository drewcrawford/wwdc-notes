#watchOS 

41mm 176x215pt
45mm 198x242pt
Larger active content areas than ever before.

Larger corner radius.  

# Design Principles
Enlarged UI components.  
Clearer - hierarchy
More glanceable - Offer a better sense of place in a quick glance.
## UI Elements
Layout margins.  Because corners are curved, content needs to come in and down so it fits well in the corners.
Status bars get taller, matched by a taller scroll clearance margin.

Layout margins are generous to give the content plenty of room to breathe.

Issues with example.  Text too close to display, issue with the wraparound display.  Text is clipped in corners.  Lost alignment to leading edge of back button.

`.scenePadding()` applies system-defined padding for the layout margin.  This resolves the layout for the current price view.  This resolves our layout issues.

Note that we didn't apply this do the day chart.  Some content is fine to go to edges.  Preserve only text content that would distort or be clipped by the edges of the display.

Large titles.  Scrolling tableviews get this large title, similar to titles on iOS.  Transitions into status bar on scroll.  Root levels and subviews get the large title.  Settings is a great example.  

Looked for opportunities like timer, to bring title of view into content area and out of the status bar.  Or remove it entirely, divorcing title of view from back butotn navigation.

By default on watchOS 8, all navigation titles are large and tinte by application accent color.  If you don't want a large title, use the navigation bar title display mode modifier.  After aplying, any subsequent view in your hierarchy will inherit the display mode of the view above.  

We use system teal as the app key color and push back the system background color to create a more muted effect.

Use of blue background enforces that you're in the mail app and not some other app.
Bright yellow visually differentiates tips from other carousel navigation apps like workouts.  
Set this in asset catalog.

Buttons used in scrolling vs buttons used in fixed views.  Fixed are pinned to the bottom of the screen and show the shape of the watch.

In series 7, we simplified this.  Now we use pill shapes.  We spent a long time on these buttons.  Deriving the shape of the button from the shape of the display helps this harmonize with the hardware of the watch.

Updated buttons go back to pre series 7 devices (????????)
All buttons in swift UI have the new look.  No changes are necessary.  

Note that swiftUI applies various defualt modifies on your behalf.
`.buttonBorderShape(.automatic)`  (scrollview vs nonscrollview) and `.buttonStyle(.bordered)`
`.borderedProminent` -> applies application accent color to the button.  More pronounced look.

## Typography
Same default type sizes.
Large for 40-41
XL for 44-45mm

3 large type sizes for accessibilitiy.

We recommend supplementing text labels with SFSymbols.  Consider using outline symbols in application's accent color.  Helps with cross-platform consistency and accessibility.

[[What's new in SF Symbols]]
[[SF Symbols in SwiftUI]]

## Keyboard
We don't draw borders around keys.  Encourages swiping to type.  To maximize space, we pull the delete key from the keyboard into the text field.
Customize autofill type for certain use cases like passwords or 2fa.  
Improved functionality for text input.  L/R accessories can be customized using SF Symbols.  We recommend using your app's accent color for both symbols to emphasize tappability.
Use describive words for input fields' placeholder text and suggested list title.
[[Craft search experiences in SwiftUI]]
[[21/What's new in SwiftUI]]

# Wrap up
* Scene padding
* Automatic button shapes
* Higherarchical navigation

[[What's new in watchOS 8]]
[[21/What's new in SwiftUI]]