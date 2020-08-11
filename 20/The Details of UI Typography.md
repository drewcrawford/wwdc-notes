# Optical sizes
* SF Text for small sizes, below 20 points
* SF Display for >=20

These are wahatcha call "optical sizes".

Optical sizes have been around since typography was invented.  In some ways, typography is more crude than it used to be.

Optical sizes are entangled with resolution.  which meant designing against the challenges of rasterization.  

Because you can scale vector sizes infinitely, basically all digital fonts contain only a single glyph across sizes.  So type designers have to pick an ideal size to design for.  So most fonts were really meant to perform at a certain size.

Of course designers can create multiple fonts for multiple sizes, but it's not a decision we take lightly, because it's a lot more work, and less convenient to use.

Wouldn't it be great if a single font could switch?  

# Variable fonts
Changes the way glyphs are stored in a font.  A glyph can describe the way each point moves to produce a related glyph.  

This year, we're moving away from optical sizes as separate fonts.  From now on, system fonts will be downloadable as single variable fonts.  

In design tools, there will be more sliders to display the various axes of a font.There are still predefined instances.

Note that for some apps, changing the point size may not adjust optical size.  So in that case you may need to set the slider yourself.

It's ok if the point size you use is out of range of the optical size.  Just set optical size to the closest value.

On previous OS, it's possible optical sizes won't work as expected.

Note that various APIs handle optical sizes autoamtically
* `preferredFont(forTextStyle:)`
* `systemFont(ofSize:)`
* `systemFont(ofSize:weight:)`
* `CTFontCreateUIFontForLanguage()`


# Tracking and leading
leading is pronounced like the bullets.

## tracking

When you increase the space between letters, we call that "tracking".  Tracking refers to the action of adding space...

I thought this was kerning?  Kerning is a microcorrection in spacing that is only applied between certain pairs.  Kerning is created by type designers, and you shouldn't modify it at all.

Because we have variable fonts now, there is no hard break in tracking where we roll over between point sizes anymore.  So we have to update the tracking tables.  You'll need to apply new tracking values.

Fonts can contain multiple tracking tables, such as "tight".  

Note that if you use an API for "kerning" you may get effects like combingin the "fl" in "waffle", whereas this will not occur if you use "tracking" API.

However the best and preferred solution to make a string fit, is to tighten automatically using the "tight' tracking table.

```swift
// UIKit: UILabel
label.allowsDefaultTighteningForTruncation = true

// AppKit: NSTextField
textField.allowsDefaultTighteningForTruncation = true

// SwiftUI
Text("hamburgefonstiv").allowsTightening(true)
```

As with our System UI fronts that have built-in `track` tables
* This year, our platforms will apply tracking to any (3rd-party) font containing both a `track` and a `STAT` table
* For such fonts, tracking can be enabled on a previous OS by applying `kCTFontSizeOptical...`

## leading

Spacing between lines.  "Line height" is the height of a font's vertical limits.  Can also be measured as the distance between 2 baselines.  Regardless of the approach, the distance remains the same.

When the space between them increases, the space between them is referred to as "leading".  The name ceomes from the days of metal type, when this was a piece of lead between 2 lines of text.

When leading between lines, the line height inlcudes the leading.  

Various cases where we tighten or loosen leading.

By using system APIs, you benefit from the APIs we improve each year.


# Text styles and Dynamic Type
## text styles
Framework that enables flexible and consistent typography with a clear hierarchy and stylistic range.

Getting emphasized text styles:

```swift
// Getting emphasized text styles

let label = UILabel()
label.text = "Ready. Set. Code."

if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .title1)
    .withSymbolicTraits(.traitBold) {
    // 28 pt Bold on iOS
    label.font = .init(descriptor: descriptor, size: 0)
}
```

How to emphasize an arbitrary style

```swift
// Getting emphasized text styles

// AppKit
let descriptor = NSFontDescriptor
    .preferredFontDescriptor(forTextStyle: .body)
    .withSymbolicTraits(.bold)
// 13 pt Semibold on macOS
let emphasizedBodyFont = NSFont(descriptor: descriptor, size: 0)

// UIKit/Catalyst
if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .body)
    .withSymbolicTraits(.traitBold) {
    // 17 pt Semibold on iOS
    let emphasizedBodyFont = UIFont(descriptor: descriptor, size: 0)
}

// SwiftUI
let emphasizedFootnoteFont = Font.footnote.bold() // 13 pt Semibold on iOS
```

weight of the variant depends on the text style.  Can be medium, semibold, bold, or heavy.

Text styles come with line heights that we think are appropriate.  But in a more constrained space.  Can use tight/loose variants.

|         | Standard leading | Tight leading | Loose leading |
|---------|------------------|---------------|---------------|
| iOS     | 0                | -2pt          | +2pt          |
| macOS   | 0                | -2pt          | +2pt          |
| watchOS | 0                | -1pt          | +1pt          |

```swift
// Getting tight leading variant
import UIKit

let label = UILabel()
label.text = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat."

if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .body)
    .withSymbolicTraits(.traitTightLeading)
    // 20 pt line height
    label.font = UIFont(descriptor: descriptor, size: 0)
}
```

or loose

```swift
// Getting tight leading variant
import UIKit

let label = UILabel()
label.text = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat."

if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .body)
    .withSymbolicTraits(.traitLooseLeading)
    // 24 pt line height
    label.font = UIFont(descriptor: descriptor, size: 0)
}
```
per-api examples
```swift
// Getting tight/loose leading variant

// AppKit
let descriptor = NSFontDescriptor.preferredFontDescriptor(forTextStyle: .headline)
    .withSymbolicTraits(.tightLeading) // Use .looseLeading for loose leading font
let tightLeadingFont = NSFont(descriptor: descriptor, size: 0) // 14 pt line height

// UIKit/Catalyst
if let descriptor = UIFontDescriptor.preferredFontDescriptor(withTextStyle: .title1)
    .withSymbolicTraits(.traitTightLeading) { // Use .traitLooseLeading for loose leading
    let tightLeadingFont = UIFont(descriptor: descriptor, size: 0) // 36 pt line height
}

// SwiftUI
// Use .loose for loose leading font
let tightLeadingFootnoteFont = Font.footnote.leading(.tight) // 16 pt line height on iOS
```

```swift
// Access rounded system font design
import UIKit

let label = UILabel()
label.text = "Today"

if let descriptor = UIFontDescriptor
    .preferredFontDescriptor(withTextStyle: .largeTitle)
    .withSymbolicTraits(.traitBold)?
    .withDesign(.rounded) {
    // SF Pro Rounded Bold
    label.font = UIFont(descriptor: descriptor, size: 0)
}
```

rounded examples across apis
```swift
/ Access system font designs

// Use .serif for New York, .monospaced for SF Mono

// AppKit
let descriptor = NSFontDescriptor.preferredFontDescriptor(forTextStyle: .body)
    .withDesign(.rounded)
let roundedBodyFont = NSFont(descriptor: descriptor, size: 0) // SF Pro Rounded

// UIKit/Catalyst
if let descriptor = UIFontDescriptor.preferredFontDescriptor(withTextStyle: .body)
    .withDesign(.rounded) {
    let roundedBodyFont = UIFont(descriptor: descriptor, size: 0) // SF Pro Rounded
}

// SwiftUI - in this case we don't apply to an existing font
let roundedBodyFont = Font.system(.body, design: .rounded) // SF Pro Rounded
```

## webkit
```css
font-family: -apple-system; /*old syntax*/
```
new syntax:
```css
font-family: system-ui; /* SF Pro */
font-family: ui-rounded; /* SF Pro Rounded */
font-family: ui-serif; /* New York */
font-family: ui-monospace; /* SF Mono */
```
## text style
* support in macOS now
* AppKit API defines a full range of text styles as in iOS
* Font sizes are optimized to match recommended macOS app text size
* No Dynamic Type support

Consider support DT in your apps.
Using text styles, you get this behavior automatically.  But can also support DT using custom fonts.

Chart of DT point sizes.  Note that different text styles may have different scaling behaviors.

Typography is critical for brand identity.

```swift
// Support Dynamic Type with custom font in UIKit

if let customFont = UIFont(name: "Charter-Roman", size: 17) {
    let bodyMetrics = UIFontMetrics(forTextStyle: .body)
    
    // Charter-Roman scaled relative to body text style
    // in different content size categories.
    let customFontScaledLikeBody = bodyMetrics.scaledFont(for: customFont)
    label.font = customFontScaledLikeBody
    label.adjustsFontForContentSizeCategory = true

    // Scaling constant 10 relative to body text style.
    let scaledValue = bodyMetrics.scaledValue(for: 10)
}
```

Can now do this in SwiftUI.

```swift
struct ContentView: View {
    let prose = "Apple provides two type families you can use in your iOS apps. San Francisco (SF). San Francisco is a sans serif type family that includes SF Pro, SF Pro Rounded, SF Mono, SF Compact, and SF Compact Rounded."
    @ScaledMetric(relativeTo: .body) var padding: CGFloat = 20

    var body: some View {
        VStack {
            Text("Typography")
				//i think relativeTo is not needed in iOS 14
                .font(.custom("Avenir-Medium", size: 34, relativeTo: .title))
            Text(prose)
                .font(.custom("Charter-Roman", size: 17))
                .padding(padding)
        }
    }
}
```

```swift
// Support Dynamic Type with custom fonts in SwiftUI

// Text with font Avenir-Roman, scaling relative to title text style.
Text("Typography").font(.custom("Avenir-Roman", size: 34, relativeTo: .title)) 

// Text with font Helvetica, scaling relative to body text style (by default).  Behavior change in iOS 14.
Text("Title").font(.custom("Helvetica", size: 17))

// Text with font Courier, always use fixed size, do not scale according to user setting.
Text("Fixed").font(.custom("Courier", fixedSize: 17))

// Constant 10, scaled relative to title text style.
@ScaledMetric(relativeTo: .title) private var spacing: CGFloat = 10.0
```

# Wrap up
* Try using system fonts such as SF Pro, SF Pro Rounded, SF Mono, and New York
* Using text styles helps create typographic hierarchy
* Consider supporting Dynamic Type with custom fonts
* Only override system behavior in exceptional cases
* Custom tracking should be size specific

