# Improved
* Service Workers
* XHR+Fetch
* WebAssembly
* etc.

# Performance
|                                         | Improvement    |
|-----------------------------------------|----------------|
| Link clicking to unvisited page         | 13% faster     |
| Link clicking to recently visited page  | 42% faster     |
| Entering URL to recently visited page   | 52% faster     |
| Instant back                            | 34% more pages |
| First-page PDF render while downloading | 60x faster     |
| Closing unresponsive tabs               | 50 ms          |

|                       | Improvement      |
|-----------------------|------------------|
| Scrolling CPU usage   | 3x less          |
| Indexed DB operations | Up to 10x faster |
| for-of loops          | Up to 5x faster  |
| JS promises           | Up to 2x faster  |
| JS delete operations  | Up to 12x faster |

# Web API
## Web animations API
* create and control animations in JS
* query animations
* seek to a specific time
* change speed/direction

```js
// Web Animations API Code Example

let needle = document.getElementById("needle");
let logo = document.getElementById("logo");
logo.addEventListener("click", () => {
    needle.animate({
        transform: [
            "rotateX(35deg) rotateZ(13deg)", 
            "rotateX(35deg) rotateZ(733deg)",
        ],
        easing: ["ease-out"],
    }, 800);
});
```

WebInspector now shows these animations for debugging purposes

[[What's New in Web Inspector]]

## Resize observer
* Reports element size changes
* Reports changes based on
	* Viewport resizing
	* Display property
	* Appended elements

```js
// Resize Observer Example

let formatPanelObserver = new ResizeObserver((entries) => {
    entries.forEach((entry) => {
        let container = entry.target;
        container.classList.toggle("small", entry.contentRect.width < 175);
   }
});

formatPanelObserver.observe(document.getElementById("format-panel"));
```

see webkit blogpost for details

## async clipboard API
* system clipboard read and write
* keeps the page usable
* No selection or focus needed to copy
* Supports images and rich text
* Requires a secure context and user interaction

```js
// Programmatic copy
copyButtonElement.addEventListener("click", (event) => {
    navigator.clipboard.writeText("Plain text to copy.").then(() => {
       // Successful copy
    }, () => {
       // Copy failed
    });
});
```
```js
// Programmatic copy
copyButtonElement.addEventListener("click", (event) => {
    navigator.clipboard.writeText("Plain text to copy.").then(() => {
       // Successful copy
    }, () => {
       // Copy failed
    });
});

// Programmatic paste
pasteButtonElement.addEventListener("click", (event) => {
    navigator.clipboard.readText().then((clipText) => {
        document.querySelector(".editor").innerText += clipText);
    });
});
```

(rich text example not shown)

Learn more on the webkit blog.

## Use native event dispatch and handling
* No DOM reliance
* Combine with CustomEvent

## Web components
```html
<template id="format-button">
    <button class="format">
        <span class="icon"></span>
        <span class="label"></span>
    </button>
</template>
```

```js
//byID value is repurposed to serve as the custom element hook
//extend generic HTML element
let template = document.getElementById("format-button");
window.customElements.define(template.id, class extends HTMLElement {
    constructor() {
        super();

        this.attachShadow({mode: "open"});
		//clone the existing item and then we will modify
        let newButtonElement = template.content.cloneNode(true);

        let parts = newButtonElement.querySelectorAll("span");
        parts[0].textContent = this.getAttribute("data-icon");
        parts[1].textContent = this.textContent;
		
        this.shadowRoot.appendChild(newButtonElement);
        this.addEventListener("click", this.handleClick.bind(this));
    }
});
```

Now our page markup can create these elements

```js
<format-button id="bold" data-icon="B">Bold</format-button>
<format-button id="italic" data-icon="I">Italic</format-button>
<format-button id="underline" data-icon="U">Underline</format-button>
<format-button id="strikethrough" data-icon="S">Strikethrough</format-button>
<format-button id="paste" data-icon="&#x1f4cb;">Paste</format-button>
```

You provide a generic component that can be used in various ways.  So you want to give the page author control over how to style.

## CSS shadow parts
* Exposes parts of a component to content authors
* Protects critical layout styles of the components
* Allows content authors to customize components


### Original markup:
```html
<template id="format-button">
    <button class="format">
        <span class="icon"></span>
        <span class="label"></span>
    </button>
</template>
```

### shadow parts addition
```html
<template id="format-button">
    <button class="format">
        <span part="icon" class="icon"></span>
        <span part="label" class="label"></span>
    </button>
</template>
```
By doing the above, these are now exposed to CSS.

```css
#bold::part(icon) {
    color: var(--formatting-button-icon-color);
    font-weight: bold;
}

#italic::part(icon) {
    color: var(--formatting-button-icon-color);
    font-style: italic;
}

#underline::part(icon) {
    color: var(--formatting-button-icon-color);
    text-decoration: underline;
}
```

## HTML enterkeyhint attribute
* Labels virtual keyboard enter key


```html
<div id="editor" contenteditable="true" enterkeyhint="send"></div>
```

## Web Authentication API
[[Meet Face ID and Touch ID for the web]]

# CSS
## system font families
```css
font-family: system-ui; /*sf*/
font-family: ui-sans-serif;
font-family: ui-serif; /*new york?*/
font-family: ui-monospace; /*SFMono*/
font-family: ui-rounded; /*SFRounded*/
```

## `line-break: anywhere`
Break before content overflows
Keep long content visible.  
Prevent layout issues

```css
code {
    line-break: anywhere;
}
```

## `:is()` pseudo-selector
* Matches a list of selectors
* specificity of the most specific selector
* Avoids repetitive selectors


```css
/* Removing margins from subsequent headings */
h1, h2, h3, h4, h5, h6 {
    margin-top: 3em;
}


/* remove the margin from headings when followed by the next level down */
h1 + h2,
h2 + h3,
h3 + h4,
h4 + h5,
h5 + h6 {
    margin-top: 0;
}
```
Problem here is that users may follow h1 by h3 for example, not necessarily h1->h2.  To do that, you need this

```css
h1, h2, h3, h4, h5, h6 {
    margin-top: 3em;
}

h1 + h2, h1 + h3, h1 + h4, h1 + h5, h1 + h6,
h2 + h3, h2 + h3, h2 + h4, h2 + h5, h2 + h6,
h3 + h4, h3 + h3, h3 + h4, h3 + h5, h3 + h6,
h4 + h5, h4 + h3, h4 + h4, h4 + h5, h4 + h6,
h5 + h6, h5 + h3, h5 + h4, h5 + h5, h5 + h6 {
    margin-top: 0;
}
```

instead, we can say

```css
h1, h2, h3, h4, h5, h6 {
    margin-top: 3em;
}

:is(h1, h2, h3, h4, h5, h6) + :is(h1, h2, h3, h4, h5, h6) {
    margin-top: 0;
}
```

## `:where()`
* Same capability as `:is()`
* Arguments always have 0 specificity

```css
/* :is() specificity prevents the override from working */
/* Idea here is to make these (h1 I guess?) following p
to be this uppercase modifier,
while h2,h3,h4 etc. + p to be non-uppercase.  However, with :is
it won't work as expected */
:is(.intro, .pullquote, #hero) + p {
    text-transform: uppercase;
}

h2 + p,
h3 + p,
h4 + p,
h5 + p,
h6 + p {
    text-transform: none;
}
```
```css
/* Now, elements matched by 'where' are set to a specificity level of 0*/
:where(.intro, .pullquote, #hero) + p {
    text-transform: uppercase;
}

/* followup rules override as expected */
h2 + p,
h3 + p,
h4 + p,
h5 + p,
h6 + p {
    text-transform: none;
}
```

# Media
## WebP image support
* smaller file sizes
* lossy and lossless formats
* Transparency and animation support

```html
<picture>
  <source srcset="example.webp" type="image/webp">
  <img src="example.jpg" alt="Example Image">
</picture>
```
Server side, use `Accept:` header.

## Default image aspect ratio
Allows you to fix "images load and fuck up my layout" problem
* Calculates from width and height attributes

```html
<img src="MexicoCity.png" width="560" height="747">
```

## Default image orientation
```css
image-orientation: from-image; /* Safari should respect EXIF orientation */
```

Can also use

```css
image-orientation: none; /* Safari should ignore EXIF orientation */
```

## Mac HDR display support
* System support for HDR video
* Media query support for `dynamic-range`

```html
<style>
@media only screen (dynamic-range: high) {
    /* HDR-only CSS rules */
}
</style>
```

```html
<style>
@media only screen (dynamic-range: high) {
    /* HDR-only CSS rules */
}
</style>

<script>
if (window.matchMedia("dynamic-range: high")) {
    // HDR-specific JavaScript
}
</script>
```

## Remote playback API
Standards-based version of AirPlay API
* Adds AirPlay support to custom web players
* Works across different remote playback devices

```html
<video id="videoElement" src="https://site.example/video.mp4"></video>
<button id="deviceButton">Send video to a remote device</button>

<script>
    let videoElement = document.getElementById("videoElement");
    let deviceButton = document.getElementById("deviceButton");
    deviceButton.addEventListener("click", (event) => {
        videoElement.remote.prompt().then(updateRemotePlaybackState);
    });
</script>
```

## Picture-in-picutre API
* Standards-based


```html
<video id="videoElement" src="https://site.example/video.mp4"></video>
<button id="pipButton">Enter picture-in-picture mode</button>

<script>
    let videoElement = document.getElementById("videoElement");
    let pipButton = document.getElementById("pipButton");
    pipButton.addEventListener("click", (event) => {
        videoElement.requestPictureInPicture().then(handlePictureInPicture);
    });
</script>
```

## Timed metadata support
* Exposed HLS `#EXT-X-DATERANGE` to DataCue
* Event message  boxes ('emsg') in fragmented mp4

## Subtitles and captions
* Render natively with TextTrackCue
* Use a variety of caption formats
* Respect user accessibility preferences
* Shown in fullscreen and pip

# JS
## BigInt support
* Unbounded integers
* Can't mix with regular number
* Not JSON serializable

BigInt will drop decimal values during division.

## Nullish coalescing
* works like other logical operators
* checks for existence

```js
class Person {
    constructor(firstName, lastName, age) {
        this.firstName = firstName ?? "Unknown";
        this.lastName = lastName ?? "Unknown";
        this.age = age ?? NaN;
   }
}

console.log(new Person());  
// { firstName: "Unknown", lastName: "Unknown", age: NaN }

console.log(new Person(false, false, true));
// { firstName: false, lastName: false, age: true }

console.log(new Person("John", "", 0));  
// { firstName: "John", lastName: "", age: 0 }

console.log(new Person("John", "Appleseed", 42));  
// { firstName: "John", lastName: "Appleseed", age: 42 }
```

## Optional chaining
* Nullish coalescing for property access
* Works for properties, indexes, and methods

```js
class Person {
    constructor(firstName, lastName, age) {
        this.firstName = firstName ?? "Unknown";
        this.lastName = lastName ?? "Unknown";
        this.age = age ?? NaN;
        this.name = { firstName: this.firstName, lastName: this.lastName };
  }
}

function register(person) {
    // Before (that is, without) optional chaining
    if (person !== undefined && person.name !== undefined)
        console.log(person.name.firstName);
}

register(new Person());
// undefined

register(new Person("John", "Appleseed"));
// "John"
```

```js
class Person {
    constructor(firstName, lastName, age) {
        this.firstName = firstName ?? "Unknown";
        this.lastName = lastName ?? "Unknown";
        this.age = age ?? NaN;
        this.name = { firstName: this.firstName, lastName: this.lastName };
  }
}

function register(person) {
    // With optional chaining
    console.log(person?.name.firstName);
}

register(new Person());
􀆊 undefined

register(new Person("John", "Appleseed"));
􀆊 "John"
```

```js
// Without optional chaining
console.log(person.children[0]);
// TypeError: undefined is not an object

// With optional chaining
console.log(person.children?.[0]);
// undefined
```

```js
// Without optional chaining
console.log(person.fullName());
􀆊 TypeError: person.fullName is not a function.

// With optional chaining
console.log(person.fullName?.());
􀆊 undefined
```
## Logical assignment operators
```js
a &&= b // and assignment operator
a ||= b // or assignment operator
a ??= b // nullish assignment operator
```
* Evaluates once
* Doesn't always reassign

Consider this example, here we always reassign

```js
// Nullish coalescing approach
element.innerHTML = element.innerHTML ?? "Hello World!"
```

Can avoid reassigning unless null or undefined with
```js
// Logical assignment operator
element.innerHTML ??= "Hello World!"
```

## public class fields
* Declare member properties
* Always available on the object

```js
class Person {
    firstName = "";
    lastName = "";
    age = NaN;
    children = []; //children will still be available even though not set in constructor.

    constructor(firstName, lastName, age) {
        this.firstName = firstName ?? "Unknown";
        this.lastName = lastName ?? "Unknown";
        this.age = age ?? NaN;
    }
}
```

## String `replaceAll`
* replaces all instances of a string

# Platform integration
## Customizing AR quick look
* customize elements of the banner
* Or create a completely custom HTML banner

[[Shop Online with AR Quick Look]]

## Apple pay
* New button types and custom rounded corners
* request Redacted billing information

[[20/What's new in Wallet and Apple Pay]]

## App clips
Offer an App Clip on your webpages
#appclips 

```html
<meta name="apple-itunes-app"
      content="app-id=myAppStoreID,
               app-clip-bundle-id=clipBundleID,
               affiliate-data=myAffiliateData,
               app-argument=myURL">
```

[[Explore App Clips]]

# Wrap up
* try these features for yourself
* File WebKit bugs at bugs.webkit.org
* File Safari bugs with Feedback Assistant

Safari Technology Preview.  
webkit.org
@webkit
