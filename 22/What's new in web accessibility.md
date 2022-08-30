#uikit #accessibility 

# Assistive technologies
1 in 7 people have a disability that affects the way they interact with the world, their devices, and the web.  Any age, any duration, etc.

* VoiceOver
* Switch Control
* Voice Control
* Full Keyboard Access

[[Support Full Keyboard Access in your iOS app]]

## Demo
You have many tools for making pages accessible.  For example, safari has builtin accessibility support
* semantic HTML
	* Guarantees a consistent, accessible experience across browsers.
* Custom components with JavaScript and ARIA.

# Custom controls
We could replace radio buttons with custom control.

```js
class PizzaControl {
  constructor(id) {
    this.control = document.getElementById(id);
    this.sliceCount = 4;
    
    this.control.addEventListener("click", (event) => {
      const newSliceCount = this.computeSliceCount(event);
      this.update(newSliceCount);
    });
  }
}
```

What about our users with visual disabilities?  When building custom components, we need to consider how users will interact.

Give it a role with an attribute of slider.
```html
<div id="pizza-input" 
     role="slider" tabindex="0"
     aria-valuemin="0" aria-valuemax="8"
     aria-valuenow="4" aria-valuetext="4 slices">
</div>
```

minimum, maximum, etc.  Index of 0 to ensure component is focusable.  
valuemin, valuemax.  
aria-valuenow => current value of control.
add value text as well.

Handle updates to the control.  On iOS, VO facilitates the adjustment of slider of single finger swipe up or down.  Safari simulates arrowkey right/left.

```js
class PizzaControl {
  constructor(id) {
    this.control = document.getElementById(id);
    this.sliceCount = 4;

    // …click event listener…
    
    this.control.addEventListener("keydown", (event) => {
      const key = event.key;
      if (key === "ArrowRight" || key === "ArrowUp")
        this.update(this.sliceCount + 1);
      else if (key === "ArrowLeft" || key === "ArrowDown")
        this.update(this.sliceCount - 1);
    });
  }
}
```

Not only benefits VO users but also FKA.  

Now we want to define our update function.  First, we clamp value to ensure its within bounds

```js
class PizzaControl {
  // …constructor…
  
  update(newSliceCount) {
    this.sliceCount = Math.max(0, Math.min(newSliceCount, 8));

    // Visually re-render `this.sliceCount` slices
    // …

    // Update the ARIA representation of the control
    this.control.setAttribute("aria-valuenow", this.sliceCount);
    const sliceModifier = this.sliceCount === 1 ? "slice" : "slices";
    this.control.setAttribute("aria-valuetext", `${this.sliceCount} ${sliceModifier}`);
  }
}
```

If you're updating the visual representation, think about how to update the aria representation.

Let's go back our webapp and see what the experience is like.

# Web Speech and SSML
Build richer experiences for all users.

* SpeechRecognition (audio input)
* SpeechSynthesis (audio output)

provide a voice assistant or voice-only interface.  Motor disabilities, etc.  

New to speech synthesis is the ability to use SSML to manipulate the way your text is spoken.  Tons of capabilities.
```html
<speak>
  Breathe in <break time="3s"/> and breathe out.
</speak>

<speak>
  <phoneme alphabet="ipa" ph="təˈmeɪtoʊ">tomato</phoneme>
  <phoneme alphabet="ipa" ph="təˈmɑːtəʊ">tomato</phoneme>
</speak>

<speak>
  <prosody pitch="-2st" rate="slow" volume="loud">
    Hello world!
  </prosody>
</speak>
```

To learn more, see the spec from w3.org

```html
<button id="read-question-btn">
  Read question<span aria-hidden="true">🔊</span>
</button>
```

```html
function wrapWithSSML(phrase, locale) {
  return `
    <break time=“100ms"/>
    <prosody rate=“80%">
      <lang xml:lang="${locale}">
        ${phrase}
      </lang>
    </prosody>
  `;
}
```

```js
const readQuestionButton =
  document.getElementById("read-question-btn");

readQuestionButton.addEventListener("click", () => {
  const ssml = `
    <speak>
      How do you say
        ${wrapWithSSML("the water", "en-US")}
      in Spanish?
      ${wrapWithSSML("El agua", "es-MX")}
      ${wrapWithSSML("La abuela", "es-MX")}
      ${wrapWithSSML("La abeja", "es-MX")}
      ${wrapWithSSML("El árbol", "es-MX")}
    </speak>
  `;
  const utterance = new SpeechSynthesisUtterance(ssml);
  window.speechSynthesis.speak(utterance);
});
```

## Demo


# `<dialog>`

Another common design pattern is the modal. sign in, sign out form.  Provide an accessible modal experience with the aria-modal attribute.

 Makes providing an accessibility-friendly experience easier.  Escape key, scrub gesture, etc.  To see this in action, let's see the shell score button on our quiz assessment web app.
Create our dialog element.  

```html
<dialog id="show-score-modal">
  <form method="dialog">
    You got all six questions correct. Great work!
    <button type="submit">Close</button>
  </form>
</dialog>
```

By using dialog, any submit-type controls will cause it to close.

LEt's use `showModal`.

```js
const showScoreButton =
  document.getElementById("show-score-btn");

showScoreButton.addEventListener("click", () => {
  document
    .getElementById("show-score-modal")
    .showModal();
});
```

The dialog element handles iOS scrub gesture out of the box.  Two-fingers right, left, right in a swipe gesture.

When we opened the modal, VO only read the close button.  Maybe use aria label to provide more information.

```html
<dialog id="show-score-modal" aria-labelledby="modal-content">
  <form method="dialog">
    <span id="modal-content">
      You got all six questions correct. Great work!
    </span>
    <button type="submit" autofocus>Close</button>
  </form>
</dialog>
```

autoclose was default but maybe it didn't make sense if we were to have more controls.

# Wrap up
* assistive tehcnologies
* custom controls
* web speech and SSML
* `<dialog>`

* https://developer.apple.com/forums/tags/wwdc2022-10153
* https://developer.apple.com/forums/create/question?&tag1=2&tag2=316&tag3=372030
* https://support.apple.com/guide/iphone/learn-voiceover-gestures-iph3e2e2281/ios
* https://www.w3.org/TR/speech-synthesis/
* https://www.w3.org/WAI/ARIA/apg/patterns/dialogmodal/
