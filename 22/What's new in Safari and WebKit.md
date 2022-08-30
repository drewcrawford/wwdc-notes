We've been adding lots of features.

162 new features and improvements from Safari 15.1 to 16.

Best way to see what's new is in Safari Technology Preview.  Try out the latest and greatest for safari/webkit and also hepl us know what should be next.

# HTML
New dialog element.
```html
<!-- <dialog> element -->

<dialog method="dialog">
  <form id="dialogForm">
    <label for="givenName">Given name:</label>
    <input class="focus" type="text" name="givenName">
    <label for="familyName">Family name:</label>
    <input class="focus" type="text" name="familyName">
    <label>
      <input type="checkbox"> Can trade in person
   </label>
   <button>Send</button>
  </form>
</dialog>
```

new backdrop psuedo-element
```css
/* ::backdrop pseudo-element */

dialog::backdrop {
  background: linear-gradient(rgba(233, 182, 76, 0.7), rgba(103, 12, 0, 0.6));
  animation: fade-in 0.5s;
}

@keyframes fade-in {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

Person who posted it needs to be able to accept the request.  Atht ebottom of the page, there's a carousel to flip through received requests.  Don't want someone to accidentally interact with something that isn't frontmost with clicks or keyboard.

inert can fix tihs.

```js
// inert attribute

function switchToIndex(index) {
  this.items.forEach(item => item.inert = true);
  this.items[index].inert = false;
  this.currentIndex = index;
}
```

Includes disabling interactions for assistive technologies and prevents screen readers from reading.

## Lazy loading images
Tehre are some icons in the header that I need to load right away.  for clothing item images, we can utilize lazy loading.  Images only load when the user scrolls, making page feel faster and more responsive.

```html
<img src="images/shirt.jpg" loading="lazy"
     alt="a brown polo shirt"
     width="500" height="600">
```
# CSS
We're thrilled to announce container queries will ship in iOS 16.

Using container queries to handle the change in the layout, I can write the layout code for the component once, and use it in any place, in contianer of any size.
```cs
/* Container queries */

.container {
  container-type: inline-size;
  container-name: clothing-card;
}
.content {
  display: grid;
  grid-template-rows: 1fr;
  gap: 1rem;
}
@container clothing-card (width > 250px) {
  .content {
    grid-template-columns: 1fr 1fr;
  }
  /* additional layout code */
}
```

Use the container @ rule to apply conditionally based on the container size.

Cascade layers.  Powerful change to the CSS cascade.  Since the beginning of CSS, cascade has been made up of different layers. 

Can create your own custom layers where specificity is calculated inside each layer independently.

```cs
/* Author Styles - Layer A */
@layer utilities {
  div {
    background-color: red;
  }
}

/* Author Styles - Layer B */
@layer customizations {
  div {
    background-color: teal;
  }
}

/* Author Styles - Layer C */
@layer userDefaults {
  div {
    background-color: yellow;
  }
}
```

separate a design system from overrides.  etc.

has.  A pseudoclass that can act as the parent selector and more

```html
<!-- :has() pseudo-class -->

<style>
  form:has(input[type="checkbox"]:checked) {
    background: #ff927a;
  }
</style>



<form class="message">
  <textarea rows="5" cols="60" name="text" 
    placeholder="Enter text"></textarea>
  <div class="checkbox">
    <input type="checkbox" value="urgent"> 
    <label>Urgent?</label>
  </div>
  <button>Send Message</button>
</form>
```

We hope these great improvements to handling CSS make your work as a web developer that much better.

Viewport units.  Height of viewport at smallest - svh.
largest => lvh
dynamic number to match the actual height => dvh.
Basically this has to do with the retractable chrome in safari when you scroll.

width units.  block and inline, useful when writing for multiple languages when text flows different ways.
Also min/max.

It's been a challenge to animate elements on a page when following a path or moving around by an offset.

```css
/* offset-path */

:is(.blue, .teal, .yellow, .red)  {
  offset-path: circle(9vw at 5vw 50%);
}

@keyframes move {
  100% { 
    offset-distance: 100%;
  }
}

/* Animation */
.clothing-header.clicked :is(.blue, .teal, .red, .yellow) {
  animation: move 1100ms ease-in-out;
}
```

Scroll behavior.

```css
html {
  scroll-behavior: auto;
}
```

By default, this happens as a jump.  By specifying smooth, you can ask the browser to scroll smoothly.
```css
html {
  scroll-behavior: smooth;
}
```

also JS techniques.  

pitfalls of losing browser-based focus herusitics.  I'd love to use  a custom color.  

with focus-visible pseudoclass,

```css
/* :focus-visible & accent-color */

:focus-visible {
  outline: 4px solid var(--green);
  box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.3);
}

:root {
  accent-color: var(--green);
}
```

We've been replacing more and mroe of the webkit prefixes.  Now we're able to move towards standards-defined properties to make CSS easier to write and more interoperable.

* backface-visibility
* pritn-color-adjust
* text-align: match-parent
* mask
* text-combine-upright
* appearance

font palette.

```css
/* Dark mode */
font-palette: dark;

/* Light mode */
font-palette: light;
```
```css
/* Dark mode */
font-palette: dark;

/* Light mode */
font-palette: light;

/* Custom colors */
@font-palette-values --MyPalette {
  override-colors: 1 yellow;
}

#logo {
  font-palette: --MyPalette;
}
```

text-decoration-skip-ink.  Specifies what happens when underlien or overline interacts with a letter/character
ic unit => cjk character lineup.

subgrid.  For years, layout was hard.  CSS grid was revolutionary.  But it only affects children of grid container.

```css
/* Grid to layout cards */
main {
  display: grid;
  grid-template-columns: 
    repeat(auto-fit, minmax(225px, 1fr));
  gap: 1rem;
}

/* Grid to layout each card’s content */
article {
  display: grid;
  grid-row: span 5;
}
```

I'd like alonger title on one card to affect layout on the other cards.  Now, we can accomplish this with subgrid.  
```css
/* Grid to layout cards */
main {
  display: grid;
  grid-template-columns: 
    repeat(auto-fit, minmax(225px, 1fr));
  gap: 1rem;
}

/* Grid to layout each card’s content */
article {
  display: grid;
  grid-row: span 5;
/* Adding subgrid, tying them together */
  grid-template-rows: subgrid; 
}
```


# Web Inspector
Layout is easier to write when you can see what's going on.  Flexbox inspector.

layout tab in right.  Flexbox heading.
In style inspector I can toggle different alignment icons.

CSS fuzzy autocompletion for CSS editing.

Developer tool extensions support.  Can now port them to safari with the same underlying APIs that they use in other browsers.

* create new extensions
* using browser.devtools APIs
* Getting set up
[[Create Safari Web Inspector Extensions]]


# Web API
Web push.  Available on macOS.  Coming to iOS and iPadOS next year.
Remotely send notifications to your users.
Fully interoperable with other browsers.

[[Meet Web Push for Safari]]

You'll probably be excited about web app manifest improvements too.
```json
// Manifest file 

"icons": [
 {
   "src": "orange-icon.png",
    "sizes": "120x120",
    "type": "image/png"
  }
]
```

define the icon on the home screen.  Ensure there's no apple touch icon in the html header.

deliver one for apple devices, another for other platforms

```html
<!-- HTML head -->

<link rel="apple-touch-icon" href="blue-icon.png" />
```

If you don't declare any icon, they'll simply get a screenshot of your site.  

You can use the manifest file to define characteristics of your website.

Broadcast channel.  Suppose you have a site open in two windows.  We can post a message to the other open tab/window.

```js
// State change
broadcastChannel.postMessage("Item is unavailable");
```

File system access API.  Incremental updates across multiple releases.
* origin private file system
* every origin owns an independent directory

```js
// Accessing the origin private file system
const root = await navigator.storage.getDirectory();

// Create a file named Draft.txt under root directory
const draftHandle = await root.getFileHandle("Draft.txt", { "create": true });

// Access and read an existing file
const existingHandle = await root.getFileHandle("Draft.txt");
const existingFile = await existingHandle.getFile();
```

Display -P3 colro in H TML canvas.  Here we have some xamples of the color picker.  On the left of the squiggly right line, it exists in RGB.  In the right, only available in p3.  in 2016, we added support for videos and photos.  This year we're the first to introduce the new color syntax in color level 4.

This year, we've added support for p3 color with content inside the canvas element.  Utilize the full color capabilities of devices of today.

* shadow realms
* web locks
* ResizeObserverSize

# JS and WebAssembly
New shared workers interface wil help and reduce memory usage.  Instead of spawning new workers for every task, you can have just one worker in use for each browsing context with the same origin.

```js
// Create Shared Worker
let worker = new SharedWorker("SharedWorker.js");

// Listen for messages from Shared Worker
worker.port.addEventListener("message", function(event) {
  console.log("Message received from worker: " + event);
});

// Send messages to Shared Worker
worker.port.postMessage("Send message to worker");
```

shared worker receives and resopnds to messages sent with all scripts.  Less demand on your server, making yuor webpage fast and responsive for customers.

Array indexes made easier.  findLast, findLastIndex, etc.

```js
const list = ["shirt","pants","shoes","hat","shoestring","dress"];
const hasShoeString = (string) => string.includes("shoe");

console.log(list.findLast(hasAppString));
// shoestring

console.log(list.findLastIndex(hasAppString));
// 4
```

```js
const list = ["shirt","pants","shoes","hat","shoestring","dress"];

// Instead of this:
console.log(list[list.length - 2]);

// It's as easy as:
console.log(list.at(-2));
```

sheer number of new internationalization features.

* Intl.NumberFormat
* Intl.Locale
* Intl.DisplayNames
* Intl Enumeration API

No shortage of things to try and explore.

For C/ObjC/Swift.  WebASsembly gets them running without any need to rewrite.  With these years improvements, Wasm is only getting better
* 4gb addressable memory
* zero-cost exception handling

Speaking ofwasm, we also have security and privacy enhancements that not only protect users, new potential for you as developers.
# Security and privacy

Cross origin opener policy
cross-origin embedder policy

you can opt into process isolation.  Your own dedicated web content process.  Lots of apps can benefit from running on multiple threads with wasm threading.  Able to do so securely.

content security policy level 3.  CSP provides ehnhanced security controls over loaded content
protection against cross-site scripting.
Most exciting addition has been 'strict-dynamic' source expression.
Use nonces to allow certain scripts, then extend taht trust to scripts loaded by the already loaded ones.

More to explore.
Media updates
getUserDisplay() improvements
webRTC perfect negotiation
in-band chapter tracks
requestVideoFrameCallback()

web extensions
Manifest verison 3 support
New web extensions APIs

Make sure to download safari technology preview to keep up.
Explore web technology by checking out release notes, etc.
Let us know what you think in bugzilla.  Or feedback assistant for safari.





```js
// strict-dynamic source expression

// Without strict-dynamic
Content-Security-Policy: script-src desired-script.com dependent-script-1.com
  dependent-script-2.com dependent-script-3.com; default-src "self";

// With strict-dynamic
Content-Security-Policy: default-src "self"; script-src "nonce-desired" "strict-dynamic";
```


* https://developer.apple.com/forums/tags/wwdc2022-10048
* https://developer.apple.com/forums/create/question?&tag1=205&tag2=261&tag3=492030
* https://developer.apple.com/safari/download/
* https://developer.apple.com/documentation/safari-release-notes
* https://webkit.org
* https://developer.apple.com/bug-reporting/
* https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API
