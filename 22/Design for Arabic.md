660M script usage
third most written language in the world after latin and chinese

Not only for the langauge but also the directionality.  Arabic is a language from right to left.

Entire layout flows from top to bottom, right to left.  

Pages app.  Navigation, order from right to left.  Icons flow int he same direction.  Menus, controls, graphical elements, tables, designed to match the natural flow and behavior of the language.

A lot of this has been taken care of my apple.  Focu son content and a few other UI details that could be specific to your app or game.

# UI Directionality
Titles, button,s navigation bar, change order and position.

Paragraphs are aligned to the right.  

Changing the layuot directionality is only the beginning of your journey.  Entire flow of the app is now structured differently, and the user thinks about the navigation in reverse order.  

Content like image, videos, and backgrounds, remain the same.  
Carousel interaction and animation are inverted to match the UI direction.

For arabic, lowest temperature is on right and highest on left.
Pagination is reversed.

Toggles, segmented controllers, etc., are mirrored in the arabic layout.  Charts can be impacted as well.  Especially if they include a time component.


# Typography
characters are connected.
isolated, initial, media, final.
arabic is more concise than latin.  But also slightly taller, especially with the use of dots, vocalization, and diacritic marks.

SF Arabic.  Consistent with SF Pro etc.  Provides all weights you need in your app.  Using bold in the title, regular for the different cities, and light for the clock.  Explore more apps in our ecosystem to see how different way s can be used.  Health app => bold, medium, regular, etc.  

Made with scalability in mind.  Form changes slightly based on the point size (optical size).  
Smaller point sizes aer designed to prioritize legibility over style.  Adding angularity to terminals, etc.

Introducing SF Arabic Rounded this year.  
MOre practical, active, or softer look depending on the context.

[[Meet the expanded San Francisco font family]]

Non-case-sensitive script.  When uppercase is used, it gives latin mroe volume,a nd arabic feels smaller in comparison.

You may want to increase arabic font size by 10%.  This helps with legibility, epsecially when uppercase is used in small font sizes.
Letter spacing.  Some typefaces are not really optimized for having any spacing.  Misplaced links, breaking letters apart, etc.  If the typespace you're using isn't optimized for spacing, use 0% tracking.  Linkage is called Kashida.  System adds kashida of various lengths in our system fonts.

Sometimes you can see visible joins between layers if there is transparency in the font.  For system font, you don't have to worry about this.  
# Iconography
* English => lines aligned to right
* Arabic => aligned to right.
magnifying glass => align to right regardless because right handers I guess
Directionality might impact in various ways.
* writing from right to left while maintaining angularity
* speaker direction might change while slash does not
* direction of calendar dots while keeping clock hands as is
* arabic signature, etc.

SF Symbols.


# Numerals
The numerals you're familiar with are called arabic numerals.  Replaced roman numerals.

this is why mathematical calculations happen from right to left.  *Western* arabic numerals.

*Eastern* arabic numerals.  Both forms are invented in the arabic worlds.
* western => morocco, algeria, tunisia
* eastern => egypt, saudi arabia, etc.

choosing them happens automatically.

Make sure to account for both forms, or check which country you are targeting.

Refer to our RTL HIG.  

# Wrap up
* typography
* iconography
* numerals

[[Get it right (to left)]]


* https://developer.apple.com/forums/tags/wwdc2022-10034
* https://developer.apple.com/forums/create/question?&tag1=289&tag2=486030
* https://developer.apple.com/design/human-interface-guidelines/right-to-left/overview/introduction/
* https://developer.apple.com/design/resources/