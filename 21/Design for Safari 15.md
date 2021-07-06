#safari 

# New Safari design
Formerly
1.  Tall toolbar
2.  Website stayed inside this box
3.  Browser frames thes design

What if we could gt rid of the frame?

All focus n the web content.  Blend tab bar into each website by changing its background color.

Content feels more expansive.  Browser interface yields to content.

What determines which color is used?  You can choose the color and encode that choice in HTML.  Safari will figure out a color to use.

`<meta name="theme color" content="#ecd96f"`

`<meta name content= media="(prefers-color-scheme: light)"`

Put your fallback first.  Might want to set your own color for the tabbar.  If the color you specify... safari will make adjustments in order to give users a dark mode experience.

You can define in a web manifest file.  If you specify both, HEAD is used.

There's not *yet* a way to define multiple colors for light/dark mode in a manifest file.

Safari isn't going to apply some colors, e.g. matching red with close button.

If the color that you've chosen doesn't seem to be working, try adjusting a bit.

Each page has its own head.  Or use JS to swap dynamically.

That's how to create a feeling of filling the whole window.

## tabs
Tab groups.  Put away the things that I'm not working on, knowing that I can reload them later.

Provide a great icon, with high-resolution ?

Transparent background can look especially ?

Often tooling generates code that specifies theme color along with icon.  Perhaps your frontend framework defines a color?  See if it's a good one.

Page title can be manipulated.  Quicker the title gets to the point, the better.

Some sites, but `Site Name | Conte title`.  Consider reversing this, so it's easier to tell which tab is which.

## metatag
Extend header color up into tab bar.

> For now, I'll just remove the newsletter bar.

Directly under the user's thumb which makes the tab bar easier to reach.  Switch sideways between tabs.  swipe up to see the grid of all tabs.

Tab bar minimizes when you scroll the webpage.  

The websites and webapps are sthe shining star of the show.  But what if you put something important at the bottom of the page?  This is where you want to use environment variables in CSS.  Use environment variable to ensure any UI/content scoots out of the way.  

`env(safe-area-inset-bottom)`.  e.g. "safe area".

Measurement of however many px it is, from safe area to the bottom of the viewport.

Use custom view properties to turn colors into variables.  Browser defines the variable, provide info about the environment.

* safe-area-inset-top, left, right, bottom

Defined in CSS spec and work in different browsers, devices, and OS.

Landscape: also a safe area.  By default, safari will automatically move web content in from l/r edge and put it into the safe area.  Why?  Well, if it extended to L/R, content could be wrong.  

Safari will scoot content in in this case.  Including with `width=device-width`.  Commonly used in responsive web design.

Can override safar with `viewport-fit=cover`.  Now I'm responsible to ensure that the layout works.

## PWA

# New features and the web
Often the first experience people have is when somebody tells them.

Site appears as a simple rich link in messages, etc.  

New places and ways for people to share web content with each other.  

Rich links pick up webview's title and URL.  Improve on this by adding to our metadata in HTML head through metatags in open graph.

Can provide image or video.  video will autoplay

[[Ensuring beautiful rich links]]

## Visual intelligence
Get text out of images.  Mouse pointer changes to a cursor.  Not only can you select/copy, you can look up, translate, share, etc.  

Text is injected into the shadow dom inside the image.  So not affected by JS, but it is affected by zorder.  So if you overlay the image, visual intelligence won't be able to find the text.

**Content that consists mostly of text should be put as text.**

For photos, best practice is to include alt text as an image element.  Great alt text provides the kind of meaning/context that only a human can explain.

> Not all browsers have visual intelligence

# What's new in CSS
## `Aspect-ratio: 16/9`
What does it mean a 'preferred' aspect ratio?

Because images have a natural aspect ratio, height changes with width (for `height: auto`)

Not all content has a natural aspect ratio.  e.g. `article`.

By can use `width:auto height:auto`.

Now we can define a preferred aspect ratio for elements.

`min-hight:0`
or `overflow:scroll` to handle overflow behavior.

`aspect-ratio: 1` will assume it's 1:1.  

Speaking of film and video, embedded video from third party services.  Those give you an iframe to a video, but the iframe is not an element that has an aspect ratio.  

## Flex gaps
Adding a minimum space between items.  

# Form controls
In Safari 14.1, we added date/time inputs on macOS.  Client-side validation built in.

Redesigned form controls.  

New color picker for input type=color.

Make sure you know where to find important resources

# Resources

[[Develop advanced web content]]
[[Discover Web inspector improvements]]

Technology preview: purple log.  Test aginst latest bugfixes.

Documentation for web inspector

FB assistant

http://bugs.webkit.org

@webkit on Twitter

https://developer.apple.com/safari/download/
https://developer.apple.com/documentation/safari-release-notes
https://webkit.org
https://developer.apple.com/bug-reporting/

