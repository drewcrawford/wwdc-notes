Get ready to explore Safari's rich set of tools for web developers and designers. Learn how you can inspect web content, find out about Responsive Design Mode and WebDriver, and get started with simulators and devices. We'll also show you how to pair with Vision Pro, make content inspectable in your apps, and use Open with Simulator in Responsive Design Mode to help you test your websites on any device.

# Inspect web pages

Safari->Settings.  Advanced.  Show features for web developers.

HOw can I use web inspector... on this page.  Two ways to quickly open web inspector.  

Develop menu.  Show web inspector.  

Control click and choose 'inspect element'.  Not only shows WI but also selects and shows style.  Using the inspect elemen titem also works even after WI is visible.  Choose another element and use the element selection tools to see info about each element you highlight.

When clicking an element's selection mode, it's selected in web inspector.  I want to look at the bg gradient of my page.

Radial gradient.  Use the color picker to grab a color from somewhere on screen.  

[[23/What's new in Web Inspector|What's new in Web Inspector]]
[[Discover Web Inspector improvements]]

# Preview responsive layouts

Enter responsive design mode.

Target a specific size by typing in your own width/height.  Also scale factor, % of actual size being rendered.  (ex your mac window is too small).

Pixel ratio.  
### HTML image source set - 6:20
```html
<img
  src="astronaut_1x.jpg"
  srcset="astronaut_2x.jpg 2x astronaut_3x.jpg 3x"
/>
```

### CSS image set - 6:27
```css
.starfield {
  background-image: image-set("stars_1x.jpg" 1x, "stars_2x.jpg" 2x);
}
```

### CSS resolution media query - 6:32
```css
@media (min-resolution: 2dppx) {
  .divider-line {
    border: 0.5px solid grey;
  }
}
```

New this year, quickly jump into simulator from safari.  Simulators are a great way to test web content on iOS, iPadOS, and xrOS without needing a physical device.  you need xcode.

Notice that even on roughly the same width, safari on iOS displays the screen as if more screen real estate is available.  

Test behaviors that users expect to work on iOS, like double tap zoom, smooth scrolling, etc.

Once you've opened a webpage in a simulator, inspect with web inspector in develop menu.  

Take advantage of xrOS simulator test your content on that platform.  Choose 'add simulators' to open the simulator menu and learn how to add it.

# Debug beyond the Mac

For iOS, iPadOS, go to iOS settings, safari.  Advanced.  Toggle web inspector.  Finally, connect to mac with cable.  

On mac, develop_> device -> webpages, etc.  

Now you can 'connect via network' so long as your mac and device are on the same network.  New this year, inspect xrOS.  However, getting started is a bit different.

To allow inspecting the device without a wire, we support pairing over the network.  

xrOS settings -> apps -> safari.  advanced.  Web inspector.  Now, you need to pair the device with your mac.  Ensure your mac and device are on the same network.  General, remote devices.  While this screen is visible, your device can pair with macOS.

Mac, Develop menu.  'Use for development'.  6-digit pairing code is displayed.  A window will appear in which to enter the pairing code.  Once you've entered the code, pairing will complete automatically.  Once paired, you can open safari and inspect webpages and content the same wa yyou can on iOS devices.  

To learn more,

[[Meet Safari for spatial computing]]

webpages and JS are used by over 1 million apps across apple platforms.  new this year is to make that content inspectable in released version of your apps.

May use we b content for UX or to control your app, etc.
### Inspectable WKWebViews and JSContexts - 13:41
```swift
let webConfiguration = WKWebViewConfiguration()
let webView = WKWebView(frame: .zero, configuration: webConfiguration)

if #available(macOS 13.3, iOS 16.4, *) {
  webView.isInspectable = true
}

let jsContext = JSContext()
jsContext?.name = "Context name"

if #available(macOS 13.3, iOS 16.4, tvOS 16.4, *) {
  jsContext?.isInspectable = true
}
```

We recommend taht you name JS context when making it inspectable.  Help you distinguish between multiple JS contexts.  


# Automate tests

accept automation commands from a wide variety of test setups.  Finding an element, accessibility role, executing JS, taking screenshots, etc.  

Most of the time, you'll interact with WebDriver using a 3rd party library such as Selenium.  
### WebDriver test - 15:32
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.safari.options import Options as SafariOptions

options = SafariOptions()
driver = webdriver.Safari(options=options)

driver.get("https://webkit.org/web-inspector/")

search_element = driver.find_element(by=By.ID, value="search")
search_element.send_keys("device")

assert(driver.find_element(by=By.LINK_TEXT, value="Device Settings"))

driver.quit()
```

tests can also be run on iOS/iPadOS simulators, etc.


# Explore the future web

web is constantly evolving.  More exciting to begin experimenting before shfting web browsers.

FEature flags settings.  Open from the develop menu.  Feature flags.  Prevously called experimental features in safari.

Organized by topic, etc.

[[What's new in CSS]]

categorized into one of 4 statuses
* stable.  On by default.  You can toggle these to test absence, because it's not shipping in all browsers, etc.  Eventually we will remove these from the list.
* Testable -> aren't quite ready but that may be ready for early feedback.  may not be fully complete, but can help inform the standard the feature is based on.  Disabled by default.
* Preview -> Ready for developers to begin testing.  More complete than testable, but may still have bugs.  disabled by default in safari, but enabled in STP.
* Developer -> adjust WK behavior for development.  

Keep in mind, default setting is how customers will experience your content.

STP -> every two weeks.  Latest updates, etc.

Features set to the default state when you update safari.  Now we're at the end of our tools/features.  Only brushed the surface.  In addition to everything we've talked about today, safari has even more features/enhancements.  New documentation.

web inspector reference.  

please file feedbacks.

# Resources
* https://developer.apple.com/documentation/safariservices/safari_web_extensions/adding_a_web_development_tool_to_safari_web_inspector
* https://developer.apple.com/safari/download/
* https://webkit.org/web-inspector/
* https://webkit.org/

