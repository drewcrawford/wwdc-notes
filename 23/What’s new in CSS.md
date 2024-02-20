#css #safari 
Explore the latest advancements in CSS. Learn techniques and best practices for working with wide-gamut color, creating gorgeous typography, and writing simple and robust code. We'll also peer into the future and preview upcoming layout and typography features.

# Powerful layouts

masonry layout.  Pack content of different sizes.  CSS multicolumn.  If it's ok that the content order starts in the first column, flows down it, next column, etc.

But maybe you want content to flow across the page, placing each item close to the top.  Content loaded at the bottom as user scrolls.  had to use JS.  JS is slower at layout than css and more fragile.  Harder to code, layout really belongs in CSS.
## 2:49 - Masonry layout, example 1
```css
main {
	display: grid;
  grid-template-columns: repeat(auto-fill, minmax(14rem, 1fr));
	grid-template-rows: masonry;
}
```

## 3:20 - Masonry layout, example 2
```css
main {
	display: grid;
  grid-template-columns: 1fr 2fr 3fr;
	grid-template-rows: masonry;
}
```

## 3:24 - Masonry layout, example 3
```css
main {
	display: grid;
  grid-template-columns: 10rem 1fr minmax(100px, 300px);
	grid-template-rows: masonry;
}
```


More discussion is needed.  We look forward to shipping later.  What is ready today is margin trim.

## margin trim
Easy to remove margins from elements that push up against container.

ex let's say we have container with padding, margin.  These are added together.  



## 5:28 - Margin trim
```css
.card {
  background-color: #fcf5e7;
  padding: 2rlh;
  margin-trim: block;
}
h2, p {
  margin: 1rlh 0;
}
```


Instead we can ask for `margin-trim: block;` to the container.  safari 16.4.  Use inline to trim in inline direction.

# Dynamic color
Another radical leap forward that's gone unnoticed.  A leap in color.  The world is full of color.

50% more colors than srgb in display p3.  Past time for web design.

## 7:25 - Color gamut media query
```css
.card {
  background-color: #fcf5e7;
  padding: 2rlh;
  margin-trim: block;
}
h2, p {
  margin: 1rlh 0;
}
```

We shipped this in 2016.  

Many way to pick colors.  names, hex, rgb function, hsl, hwb, etc.  These are all limited to srgb.  4 new ones now defined in CSS.

LCH
OKLCH
LAB
OKLAB

Can represent models in any gamut including display p3, or any other gamut.  They're defined with 3 values each.  All 4, the L represents Lightness.

LCH,OKLCH also ahve values for chroma and hue.

LAB/OKLAB have A axis (green-red) and B axis (blue-yellow).

We shipped support for this in safari 15.4.  Chrome/edge firefox are adding support this year.

Now `color` function that specifies a space (either `srgb` or `display-p3`.)  CSS Color Module Level 5.

Relative Color Syntax.

`rgb(from #87cefa rgb / 0.7)`.

Gradient-interpolation color spaces.

For years, these were always calculated in srgb.  Now you can say which colorspace to use.  None are the best, depends on what you want.

Can also animage changes in color.  Same impact applies when mixing colors together.  Now mix colors in CSS.  Starting in 16.2, there's a new color-mix function.  

Less than 100%?  Translucent.  Can also use `currentcolor`.  

Not only does the browser need to support p3, and the OS.  But the browser needs to support rendering in p3 for each part of the webage in different parts of the DOM.  Check into the details.

In safari, 
* images with p3 color - safari 10
* overall support - 10.0
* canvas element - 15.2
* webGL canvas with drawingBufferColorSpace - 16.4

* 13.1 - color picker for p3
* 15.2 - graphics tab

still needed
* webgl canvas with unpackColorSpace
* SVG filters

# Productive selectors

`user-valid` and `user-invalid`.  For years, valid and invalid seemed like they would be helpful.  But this works as-you-type on each keystroke.

Now user-valid and user-invalid use a more cmplex algorithm to determine when it's valid or invalid.  We shipped in safari 16.5.  Warnings appear after you leave the field.

Additional `:has()` support.  Even more pseudoclasses.

`:has(:lang)`

playing, paused, seeking, buffering, etc.

dir - LTR vs RTL languages.  ex margin-inline-start and margin-inline-end.

# Typography advancements

Line-height units.  In CSS we have many units.
svh, lvh -r elative to viewport size
cqb, cqi  -> relative to container size.
1ex is ex-height of font
ch -> inline size of the 0 character in a font.
ic -> inline size of an ideographic character in CJK. (width in horizontal writing mode, etc.)

New unit.  lh for line height.  rlh for root-line-height.

Connect anything in our layout to the amount of space between lines.

Font size adjust.

Font you want might not download or be available on OS.  Best practice to declare a stack of fonts in font-family, to provide a fallback plan to the browser.  First font that's found is the one that gets used.

but fonts are not really the same size.  They may have different x-heights.

That's what we need `font-size-adjust`. 

For any latin font, there's a ratio between font size and x-height.  Usually around 50%.  By applying `font-size-adjust 0.47` I'm tellign the browser to resize every font inside the article so the x-height is always 47% of specified font size.

Shipped support in safari 16.4.  In 17, we're adding support for advanced capabilities.  wouldn't it be nice not to track down the magic number?

`font-size-adjust from-font` to figure it all out.

CAn also specify the value.  By default it's ex-height, but you can use cpa-height, ch-width, ic-width, ic-height, etc.

Also size-adjust.  Make a similar adjustment for `@font-face` rule.  

Textbox trim.  (formerly leading-trim).  Have you ever struggled to get soemthing to line up vertically?  Textbox can be vertically centered, but glyphs are smaller.

Space above/below are the same, etc.  Extra space is incredibly important.  Reserved for accent marks, vowel marks, etc.  Throws off typogrpahic layout on the web.

We expect this to keep changing.  Try it out in STP!

Counter styles.  You're probably familiar with ordered lists in HTML.  CSS provides an easy mechanism for changing which numbering system is used.  ex `list-style devanagri`.  There are dozens of these.

Define a custom style.  Until everybody supprts everything, copy/paste codesnippets from the standard into your code.

Can also use this to do things like number headings like your basic h1/h2.

Much more CSS coming this year.  In 16.2, we added support for last baseline alignment, etc.

16.4 also added supprot for media queries range syntax, much more.

Thanks for pinging us on twitter, etc.

visit bugs.webkit.org.  Issues about safari can go through FB assistant.

Check canIuse, stop bugging us about things we already did.

webkit.org

Download STP

[[Rediscover safari developer features]]
[[What's new in web apps]]

[[Explore media formats for the web]]


# Resources
* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://webkit.org
* https://webkit.org/web-inspector/
* http://feedbackassistant.apple.com
* https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API



