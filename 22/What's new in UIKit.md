#uikit 

iOS 16.  

# Productivity improvements
* improved navigation bars
* title menu
* find and replace
* editing interactions

## Navigation bars
* navigator -> general
* browser -> history, folders, etc.  
* editor -> interfaces centered around editing documents.

## title menu
Add a variety of bar button items to your app.  Displayed in centered navigation bar.  Customize toolbar.  Rearrange items to/from items popup.

Persists across app launch.
To accommodate size change, system automatically provides an overflow menu to access any items that don't fit in the navigation bar.

Title menu that works with new navigation styles supporting a few standard functions, duplicate, move, rename, export, and print.  Displayed when delegate is implemented.  Can also add custom items.

Mac catalyst: take advantage by seamlessly integrating with no code required.

#catalyst 

## Find and replace
* system find panel
* find interaction
	* UITextView, WKWebView, PDFView
## Edit menu
* alternate presentations depending on input
* customizable, bridges with mac #catalyst 
* new `UIEditMenuInteraction` API
* `UIMenuController` is deprecated
* [[adopt desktop class editing interactions]]

## Materials in sidebar
* on by default in overlay sidebar mode
* UIKit manages a set of private views

## summary
* naivgation bars
* title menu
* find and replace
* editing interactiosn
[[Meet desktop class iPad]]
[[Build a desktop class iPad app]]


# Control enhancements
* UICalendarView
* ocnfigure with different types of selection behaviors
* disable individual dates from selection
* annotate dates with decorations

* better, more correct representation of the data model
* `NASCalendar.currentCalendar` may not be gregorian
* be explicit about which calendar to use.
Date components are a better, more correct representation of the date
Explicitly specify gregorian if desired.

```swift
// Configuring a calendar view with multi-date selection

let calendarView = UICalendarView()
calendarView.delegate = self
calendarView.calendar = Calendar(identifier: .gregorian)
view.addSubview(calendarView)

let multiDateSelection = UICalendarSelectionMultiDate(delegate: self)
multiDateSelection.selectedDates = myDatabase.selectedDates()
calendarView.selectionBehavior = multiDateSelection

func multiDateSelection(
    _ selection: UICalendarSelectionMultiDate,
    canSelectDate dateComponents: DateComponents
) -> Bool {
    return myDatabase.hasAvailabilities(for: dateComponents)
}
```

Dates that cannot be selected will appear greyed out in the calendar view.

To annotate with decorations,

```swift
// Configuring Decorations
func calendarView(
    _ calendarView: UICalendarView, 
    decorationFor dateComponents: DateComponents
) -> UICalendarView.Decoration? {
    switch myDatabase.eventType(on: dateComponents) {
    case .none:
        return nil
    case .busy:
        return .default()
    case .travel:
        return .image(airplaneImage, color: .systemOrange)
    case .party:
        return .customView {
            MyPartyEmojiLabel()
        }
    }
}
```

can decorate with custom views.  Note that these do no not allow interaction and are clipped to available space.

## UIPageControl
* different images depending on selection state
* control the layout direction

```swift
// Vertical page control with custom indicators

pageControl.direction = .topToBottom
pageControl.preferredIndicatorImage = UIImage(systemNamed: "square")
pageControl.preferredCurrentIndicatorImage = UIImage(systemNamed: "square.fill")
```

Comitted to protecting user privacy and security.  In iOS 15 when app accesses pasteboard, a banner would appear to indicate pasteboard was accessed.  New to iOS 16, system behavior has changed.  Now we will display an alert that asks for permission to use the pasteboard.  

## UIPasteControl
* based ont he filled `UIButton` style
* Enabled when the pasetboard is content compatible
* avoids the paste notification

# API refinements
detents were added to sheets in iOS 15. In iOS 16, we added support for custom detents.  make sheets any size.

```swift
// Create a custom detent
sheet.detents = [
    .large(),
    .custom { _ in
        200.0
    }
]
```

identifier to refer to it from other apis.  Can disable dimming etc.
```swift
// Create a custom detent
sheet.detents = [
    .large(),
    .custom { context in
        0.3 * context.maximumDetentValue
    }
]
```
```swift
// Define a custom identifier
extension UISheetPresentationController.Detent.Identifier {
    static let small = UISheetPresentationController.Detent.Identifier("small")
}

// Assign identifier to custom detent
sheet.detents = [
    .large(),
    .custom (identifier: .small) { context in
        0.3 * context.maximumDetentValue
    }
]

// Disable dimming above the custom detent
sheet.largestUndimmedDetentIdentifier = .small
```

Note that the value returned does not account for custom area insets.  Same calculation for floating and edge-attached detents.

* System detents
* other appearance options
* updated sample code
[[Customize and resize sheets in UIKit]]

Sample code was updated for that video.

## Symbol images
* inherent rendering mode for each symbol
	* monochrome, multicolor, hierarchical, palette
	* Generally a symbol's defualt rendering mode is the preferred way to display the symbol.
	* monochrome can be explicitly requested `.preferringMonochrome`
* Variable symbols
	* reflecting a value from 0 to 1
	* e.g., current volume
	* `variableValue:`
* [[Adopt variable color in SF Symbols]]
* [[What's new in SF Symbols 4]]

 ## Swift concurrency and sendable
 * UIImage
 * UIColor
 * UIFont
 * UITraitCollection

[[Eliminate data races using Swift Concurrency]]
[[Visualize and optimize Swift concurrency]]

## Stage manager
* no code required
* deprecations:
	* `UIScreen.main`
	* `UIScreen` lifecycle notifications
## Self-resizing cells
 * enabled by default for UICollectionView and UITableView
 * cells automatically resized when their content changes
 * `selfSizingInvalidation`

* automatic invalidation when using UIListContentConfiguration
* Use `invalidateIntrinsicContentSize()` to manually invalidate
* Wrap in `UIView.performWithoutAnimation` to resize without animation

`selfSizingInvalidation = .enabledIncludingConstraints` for autolayout
automatic invalidation for autolayout changes in `contentView`

# UIKit and SwiftUI
Build ucstom cells using SwiftUI
Works with UICollectionView and UITableView.

[[Use SwiftUI with UIKit]]

```swift
cell.contentConfiguration = UIHostingConfiguration {
    VStack {
        Image(systemName: "wand.and.stars")
            .font(.title)
        Text("Like magic!")
            .font(.title2).bold()
    }
    .foregroundStyle(Color.purple)
}
```



## UIDevice deprecations
* UIDevice.name now reports model name
* (setting) UIDevice.orientation is not supported
	* Use `PreferredInterfaceOrientation` instead

# next steps
* compile your apps for iOS 16
* check ourt new system features
* adopt new UIKit APIs

