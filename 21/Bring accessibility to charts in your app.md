Charts visualize data
Quickly understand what our data is telling us, without getting too deep into the details

No value in a visual chart if you can't see it.

Audio graphs.

Audio graph menu in voiceover rotor.  Swipe down until chart details.  Double tap to open explorer view.

Same chart as the one you see, represented audio.

Without seeing the chart, audio graph allows you to perceive its important features in just a few seconds.

More precise info.

Approximately what was the maximum birth rate?  Audio graph provides an interactive mode where VO users can tap hold, drag, to listen at their own pace.

By scrubbing through until we heard the highest pitch and pausing to hear the data value, we could tell the answer.  

Can infer that this peak corresponds to baby boom era.

Bigger automobiles tend to be less fuel efficient.  How much longer to understand each datapoint?

Can also hear outliers.  

Explorer view provides summary information with important information, e.g. shape, trend, outliers, etc.

Along with audiograph, VO reads this summary to complete understanding.


# Benefits
* Reach wider audiences
* Empower your users
* Easier than ever!


# Visual accessibility
Simple line chart that shows monthly rainfall for tropical and arid regions.  How to improve visual accessiblity?

* High contrast colors.  Accessibility inspector has a calculator.  Aim for ratios of 4.5:1
* Avoid using both red and green.
* Avoid using blue and yellow together.
* Use symbols in addition to color.

Add symbols for "differentiate without color"

Use high contrast colors when increased contrast is set.

Reduce use of transparency.

Consider disabling when reduced transparency is on.

How to make data navigable for voiceover?
# Navigating data
```swift
class ChartView: UIView {
    let model: ChartModel

    func drawChart() {
        // ...
    }
}

struct ChartModel {
    let title: String
    let dataPoints: [DataPoint]
        
    struct DataPoint {
        let name: String
        let x: Double
        let y: Double
    }
}
```
1.  Make container.  Override `accessibilityContainerType`.
2.  return `.semanticGroup`.  Important to communicating which elements belong to the chart.
3.  `accessibilityLabel`.  Tell VO what to speak.  Typically, just the chart's title.
4.  Elements.  Each datapoint.  Creates elements indie the chart.
```swift
extension ChartView {
    public override var accessibilityContainerType: UIAccessibilityContainerType { … }
    public override var accessibilityLabel: String? { … }

    public override var accessibilityElements: [Any]? {
        get {
            return model.dataPoints.map { point in
                let axElement = UIAccessibilityElement(accessibilityContainer: self)
                axElement.accessibilityValue = "\(point.x) cups, \(point.y) lines of code"
                axElement.accessibilityFrameInContainerSpace = frameRect(for: point)
                return axElement
            }
        }
        set {}
    }
  
 private func frameRect(for dataPoint: DataPoint) -> CGRect {
```

value - what VO will speak.  May want to use a strings dict to define proper localization rules.

Provide frame so that VO knows where it lives.

VO speaks title to focus element inside the chart.  made it possible to navigate through each DP to create a UIAccessilbityElement for each one.

Sometimes you might have hundreds or thousands of datapoints.  don't want to mak ean element per datapoint.  Instead, break your chart into intervals.

Provides better navigation experience.

# Support audio graphs

```swift
struct ChartModel {
    let title: String
    let summary: String
    let xAxis: Axis
    let yAxis: Axis
    let data: [DataPoint]

    struct Axis {
        let title: String
        let range: ClosedRange<Double>
    }
    
    struct DataPoint {
        let name: String
        let x: Double
        let y: Double
    }
}
```

In your code, you probably have a model object.  

First, import accessiblity.  


```swift
import Accessibility

extension ChartView: AXChart {

public var accessibilityChartDescriptor: AXChartDescriptor? {
    get {
    }
    set {}
    }
}
```

How to build descriptor?

1.  Axis descriptor.  Each axis provides info about categoryical vs numerical data, range of values, positions of gridlines, speakable values.
2.  Value description provider closure.  Description speaks "cups".  May wantt o localize for pluralization, etc.
3.  Same for axis descriptor for y axis, except we format for lines of code.
4.  Add data.  `AXDataSeriesDescriptor`

```swift
public var accessibilityChartDescriptor: AXChartDescriptor? {
    get {
        let xAxis = AXNumericDataAxisDescriptor( … ) 
        let yAxis = AXNumericDataAxisDescriptor(title: model.yAxis.title,
                                                range: model.yAxis.range,
                                                gridlinePositions:[],
                                                valueDescriptionProvider: { value in
            return "\(value) lines of code"
        })
    }
    set {}
}
```

```swift
public var accessibilityChartDescriptor: AXChartDescriptor? {
    get {
        let xAxis = AXNumericDataAxisDescriptor( … )
        let yAxis = AXNumericDataAxisDescriptor( … )
        let series = AXDataSeriesDescriptor( … )
        return AXChartDescriptor(title: model.title,
                                 summary: model.summary,
                                 xAxis: xAxis,
                                 yAxis: yAxis,
                                 additionalAxes: [],
                                 series: [series])
    }
    set {}
}
```

Use `false` for isContiguous when it's dots etc., or otherwise line.

Provide an array of datapoint objects.

`summary`: kinda like "alt text".
Very helpful to VO users.

# WRap up
* Make good visual choices, avoid bad color combos, etc.
* Make your chart navigable
* Support audio graphs

Make your charts accessible for everyone

