#swiftcharts #swift 

Unprecedented resource for understanding the world.
Yet, data alone is of little use.  To make data useful, we must amke sense of it.

At apple, we spent years studying the best practices for vissualizations.  WE learnred that charts are best when they show additional useful context.  Trends, etc.

Stocks, heart rate, temperature, etc.  

**Swift Charts**
* apple-designed charts
* same syntax as swiftUI


* Important information is available without a chart.

You build charts by composition.  The main components are the bars.  Visual elements for each item in your data.  Swift charts calls these marks.  

```swift
Chart {
	   BarMark(x: .value("Description","Value itself"), y: .value("Sales",916))
}
```

> To indicate that we're not setting the height or position directly,we use `.value`

We usually want to create a chart driven bya collection.

If `ForEach` is the only content, we can also put the content directly in the `Chart` initializer.


Transpose x and y to go horizontal.

Exposes the data to voicover.  When I navigate the chart in voiceover, it has name and number of pancakes sold.

Of course, the chart supports audiographs feature we presented in 2021.

# Time series
`.value("Name", value, unit: .day)`


Move your control flow into a computed property, not swiftUI?

`.foregroundStyle(by: .value("City", series.city))`

Swift charts makes it easy to change your chart to expore different designs.
Bar chart shows total sales per day.  What if I want to compare two cities?

`.symbol(by: .value)` to differentiate without color.

* marks + Mark properties

Marks:
* bar
* point
* line
Properties:
* x position
* y position
* foreground style
* symbol

Supports much more than we've discussed today.  Add custom marks.  Combine these building blocks into great data visualizations etc.  Possibilities are endless.

dark mode, device sizes, dynamic type, voiceover, audio graphs, etc., for free.

High contrast mode.  Locales, multi-platform, etc.  

# Wrap up
* apple-designed charts
* same syntax as SwiftUI
* [[Swift Charts Raise the Bar]]
https://developer.apple.com/documentation/Charts
https://developer.apple.com/documentation/Charts/Creating-a-chart-using-Swift-Charts
https://developer.apple.com/documentation/charts/visualizing_your_app_s_data
