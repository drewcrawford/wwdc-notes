#accessibility 

# Accessibility audits
testing is a fundamental component of app development.  We're able to patch and fix bugs before we ship code.

Ensure quality of the product.  Accessible product is  high-quality.

1 in 7 people have a disability that affects the way they interact with the world and their devices.

voiceover to interact with their apps in ways that are best for them.  Delivering a high-quality product means delivering an application that is best for everyone.

Provides me with motivational quotes, writes down my thoughts, etc. 
# automation elements


```swift
func testAccessibility() throws {
    let app = XCUIApplication()
    app.launch()
    
    try app.performAccessibilityAudit()
}
```

this line audits the thing for example programamtically.  Just like the inspector does.

If your UI tests can find the elements, so can assistive technologies.  Great that i get some accessibility testing, but I'd like to add some audits to catch all kinds of issues.


* fix issues
* filter issues

[[Deliver an exceptional accessibility experience - 18]]

Use the identifier if you need to programmatiaclly refer to something in tests but don't want voiceover to read it.


```swift
view.accessibilityElements = [quoteTextView, newQuoteButton]
```


> by default, issues should not be ignored
```swift
try app.performAccessibilityAudit(for: [.dynamicType, .contrast]) { issue in
    var shouldIgnore = false
          
    // ignore contrast issue on "My Label"
    if let element = issue.element, 
       element.label == "My Label",
       issue.auditType == .contrast {
           shouldIgnore = true
    }
    return shouldIgnore
}
```

consider the following
* audit all views
* override teardown
* use test plans
* test with assistive technologies yourself!
# automation elements
expose elements specifically for automation.  Now in UIKit you'll be able to use this api to expose exactly the elements you need while still being able to customize the accessibility at the same time.

'automationElements' lets me provide things only to tests, not to vo.

```swift
view.automationElements = [imageView, quoteTextView, newQuoteButton]
```

* https://developer.apple.com/documentation/xctest/xcuiapplication
* 