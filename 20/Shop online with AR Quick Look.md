# AR quick look
iOS 13 -> jump directly into AR
Share product links
Vertical plane support
Suport for viewing face accessories

[[Advances in AR Quick Look - 19]]

Home Depot, WayFair, etc.  

Bottom banner ->
* apple pay
* custom action
* custom banner

# Banner user flow
1.  Tag
2.  Customization options
3.  Banner
4.  Event Listener

# Integrating QL

```html
<a rel="ar" href="alarm-clock.usdz#canonicalWebPageURL=https://developer.apple.com/alarm-clock-product-page/&allowsContentScaling=0">
    <img src="alarm-clock-thumbnail.jpg">
</a>
```

# banner styles
## apple pay
Simplest way to add payment to AR
Fast, secure, and streamlined
Payment happens outside of AR experience
Various apple pay button styles

* plain
* pay
* buy
* check-out
* book (evidently travel, not dead trees)
* donate
* subscribe

```html
<a rel="ar" id="ar-link" href="kids-slide.usdz#callToAction=Preorder&checkoutTitle=Kids%20Slide&checkoutSubtitle=Enjoy%20the%20playground,%20right%20from%20your%20home&price=$145">
    <img src="kids-slide-thumbnail.jpg">
</a>
```
`ar-link` specifies a unique element we will use later
`applePayButtonType`
`checkoutTitle`, `checkoutSubtitle`.  Need to percent-encode.
`price`.  Needs to be localized per language.

## custom action
customizable text for desired call to action
Connects AR experience back to webpage for further engagement
Allows for custom webpage logic
Flexible retail integration, for cases besides purchasing

```html
<a rel="ar" id="ar-link" href="kids-slide.usdz#callToAction=Preorder&checkoutTitle=Kids%20Slide&checkoutSubtitle=Enjoy%20the%20playground,%20right%20from%20your%20home&price=$145">
    <img src="kids-slide-thumbnail.jpg">
</a>
```

price parameter is now optional in iOS 14

## custom banner
Customizable through HTML
Use your own style, layout, and background
Three varying height sizes
Allows for custom webpage logic
HTTPS only

```html
<a rel="ar" id="ar-link" href="solar-panels.usdz#custom=https://developer.apple.com/solar_panels_banner.html&customHeight=small">
    <img src="solar-panels-thumbnail.jpg">
</a>
```

One `custom` parameter.  Make sure to %-encode.

height is optional.  small - 81 points, medium - 121 points, large - 161 points

```html
<a rel="ar" id="ar-link" href="solar-panels.usdz#custom=https://developer.apple.com/solar_panels_banner.html&customHeight=medium">
    <img src="solar-panels-thumbnail.jpg">
</a>
```

# Business chat
Digital engagement trhough Messages
Immersive and engaging
Supports media attachments like USDZ
Private and convenient experience

# Event Listener

## Listening for tap events

```html
<a rel="ar" id="ar-link" href="alarm-clock.usdz#applePayButtonType=plain&checkoutTitle=Retro%20Alarm%20Clock&checkoutSubtitle=Charming%20old-school%20look%20with%20built-in%20FM%20tuner&price=$92.50">
    <img src="alarm-clock-thumbnail.jpg">
</a>



<script type="application/javascript">
    const linkElement = document.getElementById("ar-link");
    linkElement.addEventListener("message", function (event) {
        if (event.data == "_apple_ar_quicklook_button_tapped") {
            // handle the user tap.   
        }
    }, false);
</script>
```

We use the fragment identifier we specified earlier, `id="ar-link"`.

Check the data property, evaluate `_apple_ar_quicklook_button_tapped`.  (evidently not private?)

https://developer.apple.com/augmented-reality/quick-look

Developer Documentation (where?)
Send us feedback in Feedback Assistant
