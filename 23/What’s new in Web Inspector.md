Web Inspector provides a powerful set of tools to debug and inspect web pages, web extensions, and WKWebViews on macOS, iOS and iPadOS. We'll share the latest updates, including improved typography inspection, editing tools for variable fonts, controls to emulate people's preferences, element badges in the DOM node tree, and Symbolic breakpoints.
#safari 

Inspect all the resources and activities of a website, webapp, etc.

settings-.advanced->show features for web developers
develop menu is now available.  You can open web inspector with cmd-option-I.

[[Rediscover safari developer features]]

# Typography inspection
Web developers use custom fonts to create beautiful content.

Getting typography right is hard.  ex,
1.  right font file, such as woff2,
2. right features
3. right glyphs, etc.
4. many CSS properties that influence the way a font renders.  Most of them are inheritable through the cascade, creating confusion.

web inspector can help with a host of typography inspection tools.  Find them in the details sidebar of the elements tab, font panel.

The font panel shows a summary of the basic font properties.  Font size, style, weight, stretch, etc.  A secition that shows supported font feature properties and their values.  Special spects of the typeface such as ligatures, smallcaps, oldstyles, etc.

New this year, the font panel shows warnings for synthetic warning or weak styles.

Italic style are often supplied in their own font files, spearate from the ones for upright styles.  When missing, wk generates its own.  Some algorithm that skews the individual glyphs.  This is called a synthetic oblique.

Specially designed to convey a particular aesthetic.  Somethign simlar happens for bold.  Keep in mind that not all fonts support this.  Web sinspector now displays a warning when synthetic bold or oblique is used.  Shows up next to synthesized weight/style in the font panel.  This warning can be a hint that the expected font file is not loaded.

Variable fonts.

A font format that can include in a single file, all the info needed to generate multiple style variations.  weight, width, slant, etc.  For each supported style, variable fonts provide a spectrum of values.  This gives you a lot more flexibility to choose the exact style for your content.


almost every aspect can be ocnfigurable.  From the thickness of the stroke to the curves of the widths, etc.  The possibilities really are vast.  All aspects of a avariable fonts that can be configured are expressed as variation axes.  You have to know the fonts' documentation, etc.

web inspector can help.  When inspecting an element that uses a variable font, the panel shows you a list of the supported variation axes.  Axis tag, axis label.  Minimum/max values.  Current (or default) value.

This year, we added interactive controls to edit these values and see results live.  Great way to tweak the font styles until they're just right.  Here's a webpage I'm building for a travel photography blog.  Photo gallery, navigation labels, etc.  Right click to inspect the heading element.

I can see this is using a variable font.  Demo.

Accessibility settings.  One of the most popular things people do is to browse the web, so it makes sense to build websites/webapps to welcome everyone.  Test your website with reduced motion.  You could go to accessibilty settings on macOS, but that will influence the entire system.  What you often want is to set it just to the page you're testing.

User preference overrides popover.  Override preferences just for the inspected page, while web inspector is open.  map to css media features.


# User preference overrides

Ideally you want to build content. 
Use this media feature to adapt your styles.


# Element badges
Provide a quick way to annotate at a glance, nodes of interest.

See grid/flex nodes.  The highlight of the badge matches the page overlay.

[[Discover Web Inspector improvements]]
[[What's new in Safari and WebKit]]
A container that scrolls horizontally because the content inside it doesn't fit the available width.  Lies undetected for a long time when scrollviews are not visible by default.

New element badge that provides a visual hint when content overflows bounds and the scrollbar is applied.

reveal the matching CSS rules in the styles panel.  The labels themselves, etc.

To fix this, let's comment out the min-width declaration.  Now each label takes up as much space as it needs.

I'll add a flex property with a value of 1.  This distributes the unused space to each label.  

Next, let's talk about

## event badge
JS event listeners attached.  Both for builtin events, and also custom JS events

name, type, source location, etc.
bubbling: yes (dom tree)
once: yes (automatically rmoved after 1 event)
enabled/disabled
breakpoint



# Breakpoint enhancements
When debugging JS, you may be used to adding console log statements.  Powerful way to debug and pause through JS without making changes to your source.

If you happen to use breakpoitns before, the easiest way to start is by clicking a line number.  This sets a JS breakpoint on that line.  Next time that line is about to run, WI will pause JS execution.

While paused, you can observe the callstack, inspect the state of objects/variables in scope, and even make changes to them through the console.

set a condition for breakpoints.  Evaluate as JS in the same scope your breakpoint is set.
sometimes you may find it easier to skip the breakpoint some number of times.
run actions, such as evaluating JS.

can log messages to the console. 

of course, these are useful.  But sometimes you just want to run the options, use 'automatically continue after evaluating' option.  Makes the log messages action behave like a console log statement.

breakpoints that trigger when microtasks, animation frames, timeouts, etc.  new this year, we've added symbolic breakpoints.  Helpful to debug calls to built-in JS functions, or to pause in multiple functions with the same name.

Be as specific or generic as you want.  Either match the name exactly, including case-sensitivity, or use a regex.

navigator.shared browser API that shows the system popover.  I'll click away to set my symbolic breakpoint, and now I'll hit the sahre button on a photo in my page.  WI has paused beofre navigate.share is invoked.

So many more workflows, try them out in your projects!

# Wrap up
* web inspector reference at webkit.org
* file issues on bugs.webkit.org
* try out safari 

[[Rediscover safari developer features]]
[[What's new in CSS]]

# resources
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/adding_a_web_development_tool_to_safari_web_inspector
* https://developer.apple.com/documentation/safari-release-notes
* https://developer.apple.com/safari/download/
* http://feedbackassistant.apple.com/
* https://webkit.org/web-inspector/
* https://webkit.org/
