3 new interactions across 3 platforms
# iPadOS pointer
In 13.4 we added mouse/trackpad support
allows use of mouse/trackpad with iPad
Pointer-specific behavior and interactions

## XCTest support
* New API for controlling iPadOS pointer
* Enables robust automated testing for pointer interactions
* Available for use starting with iPadOS 15

```swift
extension XCUIDevice {

  public var supportsPointerInteraction: Bool

}
```

```swift
extension XCUIElement {
  open func hover()

  open func click()

  open func rightClick()

  open func doubleClick()

  open func scroll(byDeltaX: CGFloat, deltaY: CGFloat)

  open func click(forDuration: TimeInterval, thenDragToElement: XCUIElement)

  open func click(forDuration: TimeInterval, thenDragToElement: XCUIElement,
                  withVelocity: XCUIGestureVelocity, thenHoldForDuration: TimeInterval)

  open class func perform(withKeyModifiers flags: XCUIElement.KeyModifiers,
                          block: () -> Void)
}
```

Also available on XCUICoordinate if additional precision is required.

## Demo
```swift
import XCTest

class Pointer_UI_Tests: XCTestCase {

  @available(iOS 15.0, *)
  func testHorizontalScrollRevealsSidebar() throws {
    let app = XCUIApplication()
    app.launch()

    let sidebar = app.tables["Sidebar"]

    // Make sure sidebar menu is not present initially.
    XCTAssertFalse(sidebar.exists, "Sidebar should not be present initially")

    // Swipe horizontally to reveal sidebar.
    app.staticTexts["Select a smoothie"].scroll(byDeltaX: -200, deltaY: 0)

    // Verify sidebar is now present.
    XCTAssertTrue(sidebar.waitForExistence(timeout: 5),
                  "Sidebar did not appear within 5 second timeout")
  }

}
```

Run on iPhone simulator?  Test fails with error message.  To resolve this, only execute this on devices with pointer interaction.

```swift
try XCTSkipUnless(...)
```


# watchOS Digital Crown
* Xcode 12.5 introduced UI testing for watchOS
* Synthesize touch events
* Support for crown click events
* Expanding support to include Digital Crown rotation

```swift
// New rotateDigitalCrown API

extension XCUIDevice {

  open func rotateDigitalCrown(delta: CGFloat, velocity: XCUIGestureVelocity = .default)

}
```

# macOS trackpad
## Types of scrolling
* Discrete
	* Incremental movement, like a scroll wheel
	* Non-intertial
* Continuous
	* Fluid movement, like a trackpad
	* Intertial

## XCTest support
* XCTest currently provides API for discrete scrolling

```swift
extension XCUIElement {

  // Use the existing API for discrete (mouse wheel-like) scrolling.
  open func scroll(byDeltaX: CGFloat, deltaY: CGFloat)

}
```

Now we're adding new API for continuous scrolling

```swift
extension XCUIElement {

  // Use the new API for continuous (trackpad-like) scrolling.
  open func swipeUp(velocity: XCUIGestureVelocity = .default)
  open func swipeDown(velocity: XCUIGestureVelocity = .default)
  open func swipeLeft(velocity: XCUIGestureVelocity = .default)
  open func swipeRight(velocity: XCUIGestureVelocity = .default)

}
```

Enable automation of new input methods.

# Wrap up
* Pointer interactions on iPadOS
* Digital crown rotation on watchOS
* Continuous scrolling on macOS

[[Handle trackpad and mouse input]]
[[Write tests to fail]]

* https://developer.apple.com/documentation/xctest/user_interface_tests
* 