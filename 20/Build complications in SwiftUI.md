#swiftUI #watchOS 

# New API
`CLKComplicationTEmplateGraphicCornerCircularView`
...
good lord there are a ton of them

Consider the ones that take a single view.

Consider `Test(, style:)` to get clockkit to keep your timer up to date

## Progress View and Gauge

### ProgressView
```swift
ProgressView(value: 0.7).progressViewStyle(CircularProgressViewStyle(tint: .red))
```

Also a linear style

### Gauge
```swift
Gauge(value: acidity, in: 3...10) {
	Image(systemName: "drop.fill")
	.foregroundColor(.green)
}.gaurgeStyle(CircularGaugeStyle())
```
Minimum value label, maximum value label, currentValueLabel, tint with a color
Gradient tint

Also a linear style

# Watch face tinting
## Desaturated tinting
Default tinting mode for complications
When the watch face is tinted, it creates a greyscale version of your view
Some faces like the extra large face, may apply a single color to the greyscale version

```swift
vary body: some View {
	ZStack {
		Circle().fill(Color.blue)
		Image("apple").foregroundColor(.yellow)
	}
}
```

If I had chosen colors with a similar brightness, the detail might disappear when desaturated.

## Color opacity tinting
Alternative tinting style taht we can opt into.  Works by creating layers within our complication, and external color is applied ot each layer by renderer

WatchKit considers the opacity of each view.
Tints each layer separately
Merges together

The color chosen seems very dial-specific.

```swift
vary body: some View {
	ZStack {
		Circle().fill(Color.blue) //background is default
		Image("apple").foregroundColor(.yellow)
		.complicationForeground() //promote apple to foreground image
	}
}
```
## Advanced customization
For example, you may want to change the background color to use a graident.  Or remove the background color.

```swift
enum ComplicationRenderingMode {
	case fullColor //displaying with its own color
	case tinted //displaying with renderer's opinion
}
@Environment(\.complicationRenderingMode) var renderingMode
```
## Takeaways
By default, switchUI views become destruated
`.complicationForeground()` groups pieces of a view together for opacity tinting
`ComplicationRenderingMode` allows you to customize full color and opacity tinting independently

## Demo
`CLKComplciationTEmplateGraphicRectangularFullView(YourView()).previewContext()`
...When both SwiftUI and ClockKit are available together.


# Best practices

* Tapping on a complication will always launch your app
* Use text, image, and drawing primitives such as shapes, paths, and paints
* Runtime warning to xcode in case you try to use a view that is not compatible with complications
* SwiftUI animations are not supported – only a timeline of static views

* Performance is measured every time a vew is shown on the watch face
* Prefer appropriately sized images
* Limit expensive drawing, such as blurs and formatted text
* Poorly performing views may penalize your complication's runtime

Treat these runtime warnings as if they're a build error

* Use default font sizes as a guide for your complication layout
* Circular and rectangular complication families mask the view for each size
* Rectangular full view features a safe area for layout
* Safe area provides a space that's safe for all complication placements.
* Use `.edgesIgnoringSafeArea()` to get space.  Be mindful of the layout of your content to prevent being clipped.

[[Create complications for apple watch]]
[[Keep your complications up to date]]
