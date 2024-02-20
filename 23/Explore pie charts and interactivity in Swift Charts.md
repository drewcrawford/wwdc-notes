Swift Charts has come full circle: Get ready to bake up pie and donut charts in your app with the latest improvements to the framework. Learn how to make your charts scrollable, explore the chart selection API for revealing additional details in your data, and find out how enabling additional interactivity can make your charts even more delightful.
###  Stacked bar chart - 2:06
```swift
Chart(data, id: \.name) { element in
  BarMark(
    x: .value("Sales", element.sales),
    stacking: .normalized
  )
  .foregroundStyle(by: .value("Name", element.name))
}
.chartXAxis(.hidden)
```

###  Pie chart - 2:44
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales)
  )
  .foregroundStyle(by: .value("Name", element.name))
}
```

###  Pie chart with angular inset - 3:05
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales),
    angularInset: 1.5
  )
  .foregroundStyle(by: .value("Name", element.name))
}
```

###  Pie chart with corner radius - 3:06
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales),
    angularInset: 1.5
  )
  .cornerRadius(5)
  .foregroundStyle(by: .value("Name", element.name))
}
```

###  Donut chart - 3:33
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales),
    innerRadius: .ratio(0.618),
    angularInset: 1.5
  )
  .cornerRadius(5)
  .foregroundStyle(by: .value("Name", element.name))
}
```

###  Donut chart with text in the center - 4:02
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales),
    innerRadius: .ratio(0.618),
    angularInset: 1.5
  )
  .cornerRadius(5)
  .foregroundStyle(by: .value("Name", element.name))
}
.chartBackground { chartProxy in
  GeometryReader { geometry in
    let frame = geometry[chartProxy.plotAreaFrame]
    VStack {
      Text("Most Sold Style")
        .font(.callout)
        .foregroundStyle(.secondary)
      Text(mostSold)
        .font(.title2.bold())
        .foregroundColor(.primary)
    }
    .position(x: frame.midX, y: frame.midY)
  }
}
```

###  Chart visualizing average sales by city - 5:14
```swift
struct LocationDetailsChart: View {
  ...

  var body: some View {
    Chart {
      ForEach(data) { series in
        ForEach(series.sales, id: \.day) { element in
          LineMark(
            x: .value("Day", element.day, unit: .day),
            y: .value("Sales", element.sales)
          )
        }
        .foregroundStyle(by: .value("City", series.city))
        .symbol(by: .value("City", series.city))
        .interpolationMethod(.catmullRom)
      }
    }
    ...
  }
}
```

###  Chart selection modifier - 5:39
```swift
struct LocationDetailsChart: View {
  @Binding var rawSelectedDate: Date?

  var body: some View {
    Chart {
      ForEach(data) { series in
        ForEach(series.sales, id: \.day) { element in
          LineMark(
            x: .value("Day", element.day, unit: .day),
            y: .value("Sales", element.sales)
          )
        }
        .foregroundStyle(by: .value("City", series.city))
        .symbol(by: .value("City", series.city))
        .interpolationMethod(.catmullRom)
      }
    }
    .chartXSelection(value: $rawSelectedDate)
  }
}
```

###  Processing raw selected date from chart selection binding - 5:47
```swift
struct LocationDetailsChart: View {
  @Binding var rawSelectedDate: Date?

  var selectedDate: Date? {
    guard let rawSelectedDate else { return nil }
    return data.first?.sales.first(where: {
      let endOfDay = endOfDay(for: $0.day)
      return ($0.day ... endOfDay).contains(rawSelectedDate)
    })?.day
  }

  var body: some View {
    Chart {
      ForEach(data) { series in
        ForEach(series.sales, id: \.day) { element in
          LineMark(
            x: .value("Day", element.day, unit: .day),
            y: .value("Sales", element.sales)
          )
        }
        .foregroundStyle(by: .value("City", series.city))
        .symbol(by: .value("City", series.city))
        .interpolationMethod(.catmullRom)
      }
    }
    .chartXSelection(value: $rawSelectedDate)
  }
}
```

###  Rule mark as selection indicator - 6:06
```swift
Chart {
  ForEach(data) { series in
    ForEach(series.sales, id: \.day) { element in
      LineMark(
        x: .value("Day", element.day, unit: .day),
        y: .value("Sales", element.sales)
      )
    }
  }
  if let selectedDate {
    RuleMark(
      x: .value("Selected", selectedDate, unit: .day)
    )
    .foregroundStyle(Color.gray.opacity(0.3))
    .offset(yStart: -10)
    .zIndex(-1)
  }
}
.chartXSelection(value: $rawSelectedDate)
```

###  Selection popover - 6:20
```swift
Chart {
  ForEach(data) { series in
    ForEach(series.sales, id: \.day) { element in
      LineMark(
        x: .value("Day", element.day, unit: .day),
        y: .value("Sales", element.sales)
      )
    }
  }
  if let selectedDate {
    RuleMark(
      x: .value("Selected", selectedDate, unit: .day)
    )
    .foregroundStyle(Color.gray.opacity(0.3))
    .offset(yStart: -10)
    .zIndex(-1)
    .annotation(
      position: .top, spacing: 0,
      overflowResolution: .init(
        x: .fit(to: .chart),
        y: .disabled
      )
    ) {
      valueSelectionPopover
    }
  }
}
.chartXSelection(value: $rawSelectedDate)
```

###  Range selection - 7:07
```swift
Chart(data) { series in
  ForEach(series.sales, id: \.day) { element in
    LineMark(
      x: .value("Day", element.day, unit: .day),
      y: .value("Sales", element.sales)
    )
  }
  ...
}
.chartXSelection(value: $rawSelectedDate)
.chartXSelection(range: $rawSelectedRange)
```

###  Overriding default selection gesture - 7:22
```swift
Chart(data) { series in
  ForEach(series.sales, id: \.day) { element in
    LineMark(
      x: .value("Day", element.day, unit: .day),
      y: .value("Sales", element.sales)
    )
  }
  ...
}
.chartXSelection(value: $rawSelectedDate)
.chartGesture { proxy in
  DragGesture(minimumDistance: 0)
    .onChanged { proxy.selectXValue(at: $0.location.x) }
    .onEnded { _ in selectedDate = nil }
}
```

###  Selection in pie charts and donut charts - 7:31
```swift
Chart(data, id: \.name) { element in
  SectorMark(
    angle: .value("Sales", element.sales),
    innerRadius: .ratio(0.618),
    angularInset: 1.5
  )
  .cornerRadius(5)
  .foregroundStyle(by: .value("Name", element.name))
  .opacity(element.name == selectedName ? 1.0 : 0.3)
}
.chartAngleSelection(value: $selectedAngle)
```

###  Daily sales chart - 7:54
```swift
Chart {
  ForEach(SalesData.last365Days, id: \.day) {
    BarMark(
      x: .value("Day", $0.day, unit: .day),
      y: .value("Sales", $0.sales)
    )
  }
  .foregroundStyle(.blue)
}
```

###  Daily sales chart with a scrollable axis - 8:07
```swift
Chart {
  ForEach(SalesData.last365Days, id: \.day) {
    BarMark(
      x: .value("Day", $0.day, unit: .day),
      y: .value("Sales", $0.sales)
    )
  }
  .foregroundStyle(.blue)
}
.chartScrollableAxes(.horizontal)
```

###  Setting the visible domain for a scrollable chart - 8:11
```swift
Chart {
  ForEach(SalesData.last365Days, id: \.day) {
    BarMark(
      x: .value("Day", $0.day, unit: .day),
      y: .value("Sales", $0.sales)
    )
  }
  .foregroundStyle(.blue)
}
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 3600 * 24 * 30)
```

###  Chart scroll position - 8:18
```swift
Chart {
  ForEach(SalesData.last365Days, id: \.day) {
    BarMark(
      x: .value("Day", $0.day, unit: .day),
      y: .value("Sales", $0.sales)
    )
  }
  .foregroundStyle(.blue)
}
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 3600 * 24 * 30)
.chartScrollPosition(x: $scrollPosition)
```

###  Snapping in a scrolling chart - 8:50
```swift
Chart {
  ForEach(SalesData.last365Days, id: \.day) {
    BarMark(
      x: .value("Day", $0.day, unit: .day),
      y: .value("Sales", $0.sales)
    )
  }
  .foregroundStyle(.blue)
}
.chartScrollableAxes(.horizontal)
.chartXVisibleDomain(length: 3600 * 24 * 30)
.chartScrollPosition(x: $scrollPosition)
.chartScrollTargetBehavior(
  .valueAligned(
    matching: DateComponents(hour: 0),
    majorAlignment: .matching(DateComponents(day: 1))))
```


# Resources
* https://developer.apple.com/documentation/charts/visualizing_your_app_s_data
