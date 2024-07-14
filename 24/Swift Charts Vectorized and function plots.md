The plot thickens! Learn how to render beautiful charts representing math functions and extensive datasets using function and vectorized plots in your app. Whether you're looking to display functions common in aerodynamics, magnetism, and higher order field theory, or create large interactive heat maps, Swift Charts has you covered.

Now allows you to visualize things beyond data.  Plot mathematical functions in your app.  Also has vectorized plotting APIs to support visualizing larger datasets more efficiently.

LinePlot - visualizing a single function
AreaPlot - fill in area between two functions.


### Histogram that shows distribution of capacity density - 1:43
```swift
Chart { 
	   ForEach(bins) { bin in 
		   BarMark( x: .value("Capacity density", bin.range), y: .value("Probability", bin.probability) 
		   ) 
	   } 
	}
```

### Visualize function with LinePlot​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​ - 2:18
```swift
Chart { 
	   LinePlot( x: "Capacity density", y: "Probability" ) { x in // (Double) -> Double 
	   normalDistribution( x, mean: mean, standardDeviation: standardDeviation ) } ForEach(bins) { bin in BarMark(...) } }
```

AudioGraph.  

Use modifiers to customize how functions look.


### Customize function plot with modifiers - 3:36
```swift
Chart { LinePlot( x: "Capacity density", y: "Probability" ) { x in normalDistribution(x, ...) } .foregroundStyle(.gray) ForEach(bins) { bin in BarMark(...) } }
```



### Visualize area under a curve with AreaPlot​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​ - 3:57
```swift
Chart { AreaPlot( x: "Capacity density", y: "Probability" ) { x in normalDistribution(x, ...) } .foregroundStyle(.gray) .opacity(0.2) ForEach(bins) { bin in BarMark(...) } }
```

Advanced function plot.  Not only visualize area under a cruve, but visualize area between two functions as well.

Returning a tuple of yStart and yEnd for a given input x.
### Visualize area between curves with AreaPlot - 4:21
```swift
Chart { AreaPlot( x: "x", yStart: "cos(x)", yEnd: "sin(x)" ) { x in (yStart: cos(x / 180 * .pi), yEnd: sin(x / 180 * .pi)) } }
```

By default, Swift Charts infers the domain by sampling the function.  But I can customize the bounds of the chart by setting x/y scale.



### Specify domain for function plots - 4:59
```swift
Chart { AreaPlot( x: "x", yStart: "cos(x)", yEnd: "sin(x)" ) { x in (yStart: cos(x / 180 * .pi), yEnd: sin(x / 180 * .pi)) } } .chartXScale(domain: -315...225) .chartYScale(domain: -5...5)
```

### Specify sampling domain for function plots - 5:18
```swift
Chart { AreaPlot( x: "x", yStart: "cos(x)", yEnd: "sin(x)", domain: -135...45 ) { x in (yStart: cos(x / 180 * .pi), yEnd: sin(x / 180 * .pi)) } } .chartXScale(domain: -315...225) .chartYScale(domain: -5...5)
```

Parametric functions -> x/y are defined in terms of t.

### Visualize parametric functions - 5:55
```swift
Chart { LinePlot( x: "x", y: "y", t: "t", domain: -.pi ... .pi ) { t in let x = sqrt(2) * pow(sin(t), 3) let y = cos(t) * (2 - cos(t) - pow(cos(t), 2)) return (x, y) } } .chartXScale(domain: -3...3) .chartYScale(domain: -4...2)
```

Piecewise functions.  Sometimes a piecewise funciton doesn't have an output for certain values.  Return `.nan`.
### Use Double.nan to represent no value - 6:40
```swift
Chart { LinePlot(x: "x", y: "1 / x") { x in guard x != 0 else { return .nan } return 1 / x } } .chartXScale(domain: -10...10) .chartYScale(domain: -10...10)
```


### Highly customized Chart - 7:43
```swift
Chart { ForEach(model.data) { if $0.capacityDensity > 0.0001 { RectangleMark( x: .value("Longitude", $0.x), y: .value("Latitude", $0.y) ) .foregroundStyle(by: .value("Axis type", $0.axisType)) }celse { PointMark( x: .value("Longitude", $0.x), y: .value("Latitude", $0.y) ) .opacity(0.5) } } }
```

### Homogeneously styled Chart - 8:00
```swift
Chart { ForEach(model.data) { RectangleMark( x: .value("Longitude", $0.x), y: .value("Latitude", $0.y) ) .foregroundStyle(by: .value("Axis type", $0.panelAxisType)) .opacity($0.capacityDensity) } }
```

In contrast, vector API like RectanglePlot allows Swift to handle larger slices of data more efficiently.

Stored properties allow swift charts to access values with a constant memory offset instead of calling the getter.  
### Vectorized plot for homogeneously styled chart - 8:23
```swift
Chart { RectanglePlot( model.data, x: .value("Longitude", \.x), y: .value("Latitude", \.y) ) .foregroundStyle(by: .value("Axis type", \.panelAxisType)) .opacity(\.capacityDensity) }
```

PointPlot takes an entire collection of data to plot.  Key path to stored properties x/y.


### Vectorized point plot API - 9:42
```swift
Chart { contiguousUSMap PointPlot( model.data, x: .value("Longitude", \.x), y: .value("Latitude", \.y) ) }
```

Modifiers for vectorized plots take keyPaths as well.  With symbolSize, I make the size of each point represent its solar panel capacity.  All other modifiers that are often used for homogenous customization support a keyPath parameter.

### Vectorized plot modifiers - 10:26
```swift
Chart { contiguousUSMap PointPlot( model.data, x: .value("Longitude", \.x), y: .value("Latitude", \.y) ) .symbolSize(by: .value("Capacity", \.capacity)) .foregroundStyle( by: .value("Axis type", \.panelAxisType) ) }
```


Use vectorized plots for large datasets where entire plot has same properties
Use mark API with fewer datapoints where you need to customize each element.  Or complex layering with zIndex.

vectorized plots:
* group data by style
* Avoid computed properties
* Specify scale domains if known

# Resources
* https://developer.apple.com/documentation/Charts/creating-a-data-visualization-dashboard-with-swift-charts
* https://developer.apple.com/documentation/Updates/SwiftCharts
* 