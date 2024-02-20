Bring your widgets to watchOS with the new Smart Stack. We'll show you how to use standard design layouts, color and iconography, and signal-based relevancy to ensure your app's widgets are glanceable, distinctive and smart. When you're ready to make your own, watch this codealong: "Build widgets for the Smart Stack on watchOS"

New and dynamic way of surfacing your app's content over watch face.  How to design widgets shown to maximize usefulness while ensuring visual consistency.  Over the years, we've seen that one of the best qualities is the ability to delivery glanceable info with a simple raise of the wrist.  Useful with watch faces that can contain many complications.

wayfinder, modular, infograph.  Watch faces like these give you everything you need with a simple glance.  Sometimes, what you want from your watch face goes beyond just utility.  Just as much an object of personal expression as an object of telling the time.  Worn like a favorite article of clothing or accessory.

All it takes is a turn of the digital crown.  Glanceable widgets spring onto the screen.  Smart stack also provides people with the ability to add, remove, and pin widgets to the top.  Dynamic or as fixed as a person wants.

# Layouts

Visual consistency is key to providing a smooth and predictable reaidng experience.  Avoid a situation where every page has writing in a different place, style, or size.

To encourage smooth reading experience, we have 6 design layouts.  Using our text styling classes, you can ensure appropriate type sizes, weights, margins.

Primarily text-based information?  3-line text layouts.  ex news.

Part of a group?  Use second layout to color-code content.  ex calendar.  

graphical with supporting text?  Integrated bar gage, and integrated circular graphic.

Numerical?  Large text layout.  Calendar month widget, etc.

Data over time?  Layout for charts.  

Only show what is necessary to ensure glanceability.  People should only spend 10s max engaging with glanceable information on their watch.  Find them in apple design resources online.

Cases where grpahically unique layout makes more sense.  Ex, how many cupts of coffee I've had.  How long until caffeine wears out?  Difficult to execute in standard layouts.  Explore a unique layout that makes more sense.

Even so, we encourage the use of our text classes for regularity and readability.  

Combo widget comes default in every smart stack.  Unique layout with 3 circular combinations.  Great for app launchers, etc.  Even better when complications provide rich data.  Better yet when all 3 complications work together as a set.
# Color and iconography

widgets don't need to defer to watch face color.  Have more flexibilty to tmake widgets more recognizable and stylistically tied.  By default, every widget has a dark material background with white text on top.  Go beyond this, choose a color that assists in app recognition or convey sinformational meaning.  For example, weather widget shows a dynamic gradient to communicate climate conditions.

Stocks widget shows red/green.  Book cover.  Etc.

thoughtful iconography further assists in setting your widget apart, and setting context for the content inside.  ex we add an SFSymbol in front of many widgets so you know the kind of content being represented.  We recommend using vector icons as they lock together with text.


# Sessions
Active state in an app that has a clear start and end.  ex when a song is being played, counting down a timer, etc.  

Key feature of the smart stack is the system generated session control widgets.  Automatically appear on top of your smart stack.  ex, system will generate a music session.

If your widget is related to a session, encourage you to focus on offering helpful content ot lead into or out of a session.  Compleemnt the generated session contorl widget.  ex suggest a track to play, show today's workout, etc.

Not only reduce redundant functions, also enable your widget to continue to be useful even during a session by changing music or starting a new timer, etc.
# Relevancy

Intelligently orders widgets so that relevant one floats the top.  Here we focus on how to design for this capability.  When designing the content inside a widget, consider when your widget should be prioritized in the stack.

* time/date is imminent
	* ex calendar
* location reached
	* ex reminders
* headphones are detected
	* ex audiobooks
* wakeup/bedtime
* active workout

# Resources

[[Build widgets for the Smart Stack on Apple Watch]]

* https://developer.apple.com/design/resources/
* https://developer.apple.com/design/human-interface-guidelines/widgets/overview/introduction/
* https://developer.apple.com/documentation/WidgetKit/Keeping-a-Widget-Up-To-Date
* https://developer.apple.com/documentation/WidgetKit

