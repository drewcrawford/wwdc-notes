Patterns independent of UIKit, swiftUI, etc.
# Design best practices
Avoid fixed widths or frames, particularly around text.
Avoid fixed spacing when it's between 2 distant objects
Allow multiple lines of wrapped text
Do not place to omany controls in a fixed space

# Demo
We have a "Localizer Hint" field in IB, near the accessibility settings.

We want buttons to lay out horizontaly, but if there's not enough space, lay out vertically.  We accomplish this by using a horizontal stack view, and querying if it has enough space to lay out its content.  If not, we will change its orientation to vertical.

To see the details, download the sample project.

Use accessibility inspector or environment overrides to adjust DT size.

# Useful tools
* Document preview lets you preview UI immediately
* Scheme options to test in specific languages, including pseudo
* DT settings can be quickly adjusted via Control Center, Accessibility Inspector, and Environment Overrides

* Auto layout fix-its provide quick solutions to common problems
* "Embed in" options quickly convert existing UI into moern containers (stackviews, grid views)

# demo
This demo is highly visual and I will not try to explain it, but basically it involves using various IB features, like "embed in", and the fixits.

# Additional considerations
* manual testing with native speakers is crucial
* Inconsistencies with OS terminology
* Truncated and clipped text
* Out of context translations

Particularly critical for localization additions or big changes
* make users excited about being able to use your app in their language
* Happy users are advocates for your app and brand

# wrap up
International customers are the majority of your customers
Be aware of how translations impact the look and feel of your app
Use localization-friendly design patterns
Testing by native speakers is irreplacable.

