We want our app to commuincate data
and be accessible
localization, dark mode, layout, dynamic type, screen sizes, animation, many more!

Declarative syntax that uses a smalla mount of code.
Rich customization options

# marks and composition
Creating charts by composition.

Chart showing the number of pankcakes delivered over 30 days.  Bar chart.

Ins wift charts, we consider each of these a mark.  a mark is a grpahical element that represents data.  
Bar mark

In a real app, you generate marks programmatically.  `ForEach`.  If `ForEach` is the only content in the chart, you can just put it in the chart directly.
```swift
import SwiftUI
import Charts

struct TopStyleChart: View {
    let data = [
        (name: "Cachapa", sales: 916),
        (name: "Injera", sales: 850),
        (name: "Crêpe", sales: 802),
        (name: "Jian Bing", sales: 753),
        (name: "Dosa", sales: 654),
        (name: "American", sales: 618)
    ]

    var body: some View {
        Chart(data, id: \.name) {
            BarMark(
                x: .value("Sales", $0.sales),
                y: .value("Name", $0.name)
            )
            // Set the foreground style of the bars.
            .foregroundStyle(.pink)
            // Customize the accessibility label and value.
            .accessibilityLabel($0.name)
            .accessibilityValue("\($0.sales) sold")
        }
    }
}
```

We set the label to the name of pancake, and the value to cells as sold.  Custom experience for VO users.

Also tracks deliveries over days.  Gives a detailed view of each individual day.

```swift
struct DailySalesChart: View {
    var body: some View {
        Chart {
            ForEach(dailySales, id: \.day) {
                // Try change to LineMark.
                BarMark( 
                    x: .value("Day", $0.day, unit: .day),
                    y: .value("Sales", $0.sales)
                )
            }
        }
    }
  
    let dailySales: [(day: Date, sales: Int)] = [
        (day: date(year: 2022, month: 5, day: 8), sales: 168),
        (day: date(year: 2022, month: 5, day: 9), sales: 117),
        (day: date(year: 2022, month: 5, day: 10), sales: 106),
        (day: date(year: 2022, month: 5, day: 11), sales: 119),
        (day: date(year: 2022, month: 5, day: 12), sales: 109),
        (day: date(year: 2022, month: 5, day: 13), sales: 104),
        (day: date(year: 2022, month: 5, day: 14), sales: 196),
        (day: date(year: 2022, month: 5, day: 15), sales: 172),
        (day: date(year: 2022, month: 5, day: 16), sales: 122),
        (day: date(year: 2022, month: 5, day: 17), sales: 115),
        (day: date(year: 2022, month: 5, day: 18), sales: 138),
        (day: date(year: 2022, month: 5, day: 19), sales: 110),
        (day: date(year: 2022, month: 5, day: 20), sales: 106),
        (day: date(year: 2022, month: 5, day: 21), sales: 187),
        (day: date(year: 2022, month: 5, day: 22), sales: 187),
        (day: date(year: 2022, month: 5, day: 23), sales: 119),
        (day: date(year: 2022, month: 5, day: 24), sales: 160),
        (day: date(year: 2022, month: 5, day: 25), sales: 144),
        (day: date(year: 2022, month: 5, day: 26), sales: 152),
        (day: date(year: 2022, month: 5, day: 27), sales: 148),
        (day: date(year: 2022, month: 5, day: 28), sales: 240),
        (day: date(year: 2022, month: 5, day: 29), sales: 242),
        (day: date(year: 2022, month: 5, day: 30), sales: 173),
        (day: date(year: 2022, month: 5, day: 31), sales: 143),
        (day: date(year: 2022, month: 6, day: 1), sales: 137),
        (day: date(year: 2022, month: 6, day: 2), sales: 123),
        (day: date(year: 2022, month: 6, day: 3), sales: 146),
        (day: date(year: 2022, month: 6, day: 4), sales: 214),
        (day: date(year: 2022, month: 6, day: 5), sales: 250),
        (day: date(year: 2022, month: 6, day: 6), sales: 146)
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: .init(year: year, month: month, day: day)) ?? Date()
}
```

Very easy to switch between chart types in swift charts.

We showed total sales over time, but now we operate in two cities.  Compare sales between two cities.

```swift
struct LocationsChart: View {
    var body: some View {
        Chart {
            ForEach(seriesData, id: \.city) { series in
                ForEach(series.data, id: \.weekday) {
                    LineMark(
                        x: .value("Weekday", $0.weekday, unit: .day),
                        y: .value("Sales", $0.sales)
                    )
                }
                .foregroundStyle(by: .value("City", series.city))
                .symbol(by: .value("City", series.city))
                .interpolationMethod(.catmullRom)
            }
        }
    }

    let seriesData = [
        (
            city: "Cupertino", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 54),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 88),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 49),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 125),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 67)
            ]
        ),
        (
            city: "San Francisco", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 81),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 90),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 52),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 72),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 137)
            ]
        )
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

To make lines look smoother, we can use a curve as the interpolation method for the lines.  Even with the two series line chart, go back to bar chart, and see how it looks like.  

Bars are automaticaly stacked because we have 2 bars.  To copmare between cities, we might use `.position(by:)`.

```swift
struct LocationsChart: View {
    var body: some View {
        Chart {
            ForEach(seriesData, id: \.city) { series in
                ForEach(series.data, id: \.weekday) {
                    BarMark(
                        x: .value("Weekday", $0.weekday, unit: .day),
                        y: .value("Sales", $0.sales)
                    )
                }
                .foregroundStyle(by: .value("City", series.city))
                .position(by: .value("City", series.city))
            }
        }
    }

    let seriesData = [
        (
            city: "Cupertino", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 54),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 88),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 49),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 125),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 67)
            ]
        ),
        (
            city: "San Francisco", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 81),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 90),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 52),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 72),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 137)
            ]
        )
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

So far we've seen bar marks and line marks.
* point
* area
* rule
* rectangle

ex

```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart {
            ForEach(data, id: \.month) {
                AreaMark(
                    x: .value("Month", $0.month, unit: .month),
                    yStart: .value("Daily Min", $0.dailyMin),
                    yEnd: .value("Daily Max", $0.dailyMax)
                )
                .opacity(0.3)
                LineMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Daily Average", $0.dailyAverage)
                )
            }
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952, dailyAverage: 127, dailyMin: 95, dailyMax: 194),
        (month: date(year: 2021, month: 8), sales: 4044, dailyAverage: 130, dailyMin: 96, dailyMax: 189),
        (month: date(year: 2021, month: 9), sales: 3930, dailyAverage: 131, dailyMin: 101, dailyMax: 184),
        (month: date(year: 2021, month: 10), sales: 4217, dailyAverage: 136, dailyMin: 96, dailyMax: 193),
        (month: date(year: 2021, month: 11), sales: 4006, dailyAverage: 134, dailyMin: 104, dailyMax: 202),
        (month: date(year: 2021, month: 12), sales: 3994, dailyAverage: 129, dailyMin: 96, dailyMax: 190),
        (month: date(year: 2022, month: 1), sales: 4202, dailyAverage: 136, dailyMin: 96, dailyMax: 203),
        (month: date(year: 2022, month: 2), sales: 3749, dailyAverage: 134, dailyMin: 98, dailyMax: 200),
        (month: date(year: 2022, month: 3), sales: 4329, dailyAverage: 140, dailyMin: 104, dailyMax: 218),
        (month: date(year: 2022, month: 4), sales: 4084, dailyAverage: 136, dailyMin: 93, dailyMax: 221),
        (month: date(year: 2022, month: 5), sales: 4559, dailyAverage: 147, dailyMin: 104, dailyMax: 242),
        (month: date(year: 2022, month: 6), sales: 1023, dailyAverage: 170, dailyMin: 120, dailyMax: 250)
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

Switch to bar mark

```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart {
            ForEach(data, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    yStart: .value("Daily Min", $0.dailyMin),
                    yEnd: .value("Daily Max", $0.dailyMax),
                    width: .ratio(0.6)
                )
                .opacity(0.3)
                RectangleMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Daily Average", $0.dailyAverage),
                    width: .ratio(0.6),
                    height: 2
                )
            }
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952, dailyAverage: 127, dailyMin: 95, dailyMax: 194),
        (month: date(year: 2021, month: 8), sales: 4044, dailyAverage: 130, dailyMin: 96, dailyMax: 189),
        (month: date(year: 2021, month: 9), sales: 3930, dailyAverage: 131, dailyMin: 101, dailyMax: 184),
        (month: date(year: 2021, month: 10), sales: 4217, dailyAverage: 136, dailyMin: 96, dailyMax: 193),
        (month: date(year: 2021, month: 11), sales: 4006, dailyAverage: 134, dailyMin: 104, dailyMax: 202),
        (month: date(year: 2021, month: 12), sales: 3994, dailyAverage: 129, dailyMin: 96, dailyMax: 190),
        (month: date(year: 2022, month: 1), sales: 4202, dailyAverage: 136, dailyMin: 96, dailyMax: 203),
        (month: date(year: 2022, month: 2), sales: 3749, dailyAverage: 134, dailyMin: 98, dailyMax: 200),
        (month: date(year: 2022, month: 3), sales: 4329, dailyAverage: 140, dailyMin: 104, dailyMax: 218),
        (month: date(year: 2022, month: 4), sales: 4084, dailyAverage: 136, dailyMin: 93, dailyMax: 221),
        (month: date(year: 2022, month: 5), sales: 4559, dailyAverage: 147, dailyMin: 104, dailyMax: 242),
        (month: date(year: 2022, month: 6), sales: 1023, dailyAverage: 170, dailyMin: 120, dailyMax: 250)
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

Option tos how average daily sales among all months.  Mute the existing items.  `RuleMark` to show average value. `.annotation` can show label modifier.



```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart {
            ForEach(data, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    yStart: .value("Daily Min", $0.dailyMin),
                    yEnd: .value("Daily Max", $0.dailyMax),
                    width: .ratio(0.6)
                )
                .opacity(0.3)
                RectangleMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Daily Average", $0.dailyAverage),
                    width: .ratio(0.6),
                    height: 2
                )
            }
            .foregroundStyle(.gray.opacity(0.5))

            RuleMark(
                y: .value("Average", averageValue)
            )
            .lineStyle(StrokeStyle(lineWidth: 3))
            .annotation(position: .top, alignment: .leading) {
                Text("Average: \(averageValue, format: .number)")
                    .font(.headline)
                    .foregroundStyle(.blue)
            }
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952, dailyAverage: 127, dailyMin: 95, dailyMax: 194),
        (month: date(year: 2021, month: 8), sales: 4044, dailyAverage: 130, dailyMin: 96, dailyMax: 189),
        (month: date(year: 2021, month: 9), sales: 3930, dailyAverage: 131, dailyMin: 101, dailyMax: 184),
        (month: date(year: 2021, month: 10), sales: 4217, dailyAverage: 136, dailyMin: 96, dailyMax: 193),
        (month: date(year: 2021, month: 11), sales: 4006, dailyAverage: 134, dailyMin: 104, dailyMax: 202),
        (month: date(year: 2021, month: 12), sales: 3994, dailyAverage: 129, dailyMin: 96, dailyMax: 190),
        (month: date(year: 2022, month: 1), sales: 4202, dailyAverage: 136, dailyMin: 96, dailyMax: 203),
        (month: date(year: 2022, month: 2), sales: 3749, dailyAverage: 134, dailyMin: 98, dailyMax: 200),
        (month: date(year: 2022, month: 3), sales: 4329, dailyAverage: 140, dailyMin: 104, dailyMax: 218),
        (month: date(year: 2022, month: 4), sales: 4084, dailyAverage: 136, dailyMin: 93, dailyMax: 221),
        (month: date(year: 2022, month: 5), sales: 4559, dailyAverage: 147, dailyMin: 104, dailyMax: 242),
        (month: date(year: 2022, month: 6), sales: 1023, dailyAverage: 170, dailyMin: 120, dailyMax: 250)
    ]

    let averageValue = 137
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

many ways to use and ocmbine marks.  Just some examples of the wide variety of charts you can build.

# Plotting data with mark properties

* quantitative
	* numerical value,
	* number of products, price, etc.  
	* Int,float.double
* Nominal
	* Discrete types or groups
	* String or enum
	* name of person, continent, product type
* Temporal
	* day: Date
	* time: Date

For horizontal/vertical bar charts we can swap y/x.

6 marks, 6 mark properties.

* "scale" this takes an abstract value and returns a property value.  for example, sales to y position. 
* A scale is created to transform the data into its corresopnding mark property.  e.g. weekday=>x, sales=>y,style=>city.
* By default, we infer automatically from data.
* Use scale modifiers to configure scale in the hcart.

ex, let's configure so y scale starts at 0 and ends at 200


```swift
struct LocationsChart: View {
    var body: some View {
        Chart {
            ForEach(seriesData, id: \.city) { series in
                ForEach(series.data, id: \.weekday) {
                    LineMark(
                        x: .value("Weekday", $0.weekday, unit: .day),
                        y: .value("Sales", $0.sales)
                    )
                }
                .foregroundStyle(by: .value("City", series.city))
                .symbol(by: .value("City", series.city))
                .interpolationMethod(.catmullRom)
            }
        }
        .chartYScale(domain: 0 ... 200)
        .chartForegroundStyleScale([
            "San Francisco": .orange,
            "Cupertino": .pink
        ])
    }
  
    let seriesData = [
        (
            city: "Cupertino", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 54),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 88),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 49),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 42),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 125),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 67)
            ]
        ),
        (
            city: "San Francisco", data: [
                (weekday: date(year: 2022, month: 5, day: 2), sales: 81),
                (weekday: date(year: 2022, month: 5, day: 3), sales: 90),
                (weekday: date(year: 2022, month: 5, day: 4), sales: 52),
                (weekday: date(year: 2022, month: 5, day: 5), sales: 72),
                (weekday: date(year: 2022, month: 5, day: 6), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 7), sales: 84),
                (weekday: date(year: 2022, month: 5, day: 8), sales: 137)
            ]
        )
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

# Customization
Dive into more customization options taht swift char provides.
axis, legends, etc.




```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart(data, id: \.month) {
            BarMark(
                x: .value("Month", $0.month, unit: .month),
                y: .value("Sales", $0.sales)
            )
        }
        .chartXAxis {
            AxisMarks( values: .stride(by: .month) ) { value in
                AxisGridLine()
                AxisTick()
                AxisValueLabel(
                    format: .dateTime.month(.narrow)
                )
            }
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952),
        (month: date(year: 2021, month: 8), sales: 4044),
        (month: date(year: 2021, month: 9), sales: 3930),
        (month: date(year: 2021, month: 10), sales: 4217),
        (month: date(year: 2021, month: 11), sales: 4006),
        (month: date(year: 2021, month: 12), sales: 3994),
        (month: date(year: 2022, month: 1), sales: 4202),
        (month: date(year: 2022, month: 2), sales: 3749),
        (month: date(year: 2022, month: 3), sales: 4329),
        (month: date(year: 2022, month: 4), sales: 4084),
        (month: date(year: 2022, month: 5), sales: 4559),
        (month: date(year: 2022, month: 6), sales: 1023)
    ]

    let averageValue = 137
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

Conditionally determine the existence of axis marks.

```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart(data, id: \.month) {
            BarMark(
                x: .value("Month", $0.month, unit: .month),
                y: .value("Sales", $0.sales)
            )
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month)) { value in
                if value.as(Date.self)!.isFirstMonthOfQuarter {
                    AxisGridLine().foregroundStyle(.black)
                    AxisTick().foregroundStyle(.black)
                    AxisValueLabel(
                        format: .dateTime.month(.narrow)
                    )
                } else {
                    AxisGridLine()
                }
            }
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952),
        (month: date(year: 2021, month: 8), sales: 4044),
        (month: date(year: 2021, month: 9), sales: 3930),
        (month: date(year: 2021, month: 10), sales: 4217),
        (month: date(year: 2021, month: 11), sales: 4006),
        (month: date(year: 2021, month: 12), sales: 3994),
        (month: date(year: 2022, month: 1), sales: 4202),
        (month: date(year: 2022, month: 2), sales: 3749),
        (month: date(year: 2022, month: 3), sales: 4329),
        (month: date(year: 2022, month: 4), sales: 4084),
        (month: date(year: 2022, month: 5), sales: 4559),
        (month: date(year: 2022, month: 6), sales: 1023)
    ]

    let averageValue = 137
}

extension Date {
    var isFirstMonthOfQuarter: Bool {
        Calendar.current.component(.month, from: self) % 3 == 1
    }
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

Make axis leading instead of trailing:

```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart(data, id: \.month) {
            BarMark(
                x: .value("Month", $0.month, unit: .month),
                y: .value("Sales", $0.sales)
            )
        }
        .chartYAxis {
            AxisMarks( preset: .extended, position: .leading)
        }
    }

    let data = [
        (month: date(year: 2021, month: 7), sales: 3952),
        (month: date(year: 2021, month: 8), sales: 4044),
        (month: date(year: 2021, month: 9), sales: 3930),
        (month: date(year: 2021, month: 10), sales: 4217),
        (month: date(year: 2021, month: 11), sales: 4006),
        (month: date(year: 2021, month: 12), sales: 3994),
        (month: date(year: 2022, month: 1), sales: 4202),
        (month: date(year: 2022, month: 2), sales: 3749),
        (month: date(year: 2022, month: 3), sales: 4329),
        (month: date(year: 2022, month: 4), sales: 4084),
        (month: date(year: 2022, month: 5), sales: 4559),
        (month: date(year: 2022, month: 6), sales: 1023)
    ]

    let averageValue = 137
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

## Plot area

Configure plot area of a achart.  Here we take in original area and return modified area.

ex height is driven by number of categories in the chart.  
```swift
struct TopStyleChart: View {
    var body: some View {
        Chart(data, id: \.name) {
            BarMark(
                x: .value("Sales", $0.sales),
                y: .value("Name", $0.name)
            )
            // Set the foreground style of the bars.
            .foregroundStyle(.pink)
            // Customize the accessibility label and value.
            .accessibilityLabel($0.name)
            .accessibilityValue("\($0.sales) sold")
        }
        .chartPlotStyle { plotArea in
            plotArea.frame(height: 60 * 6)
                    .background(.pink.opacity(0.2))
                    .border(.pink, width: 1)
        }
    }

    let data = [
        (name: "Cachapa", sales: 916),
        (name: "Injera", sales: 850),
        (name: "Crêpe", sales: 802),
        (name: "Jian Bing", sales: 753),
        (name: "Dosa", sales: 654),
        (name: "American", sales: 618)
    ]
}
```

achieve a special visual effect, e.g. set a background color.

Swift charts provides you a `ChartProxy` to access the x/y scale.  Use the `.position` and `.value` to get data value at a given position.

Allows you to coordinate other views with the chart.

`.chartOverlay` and `.chartBackground` provides you the proxy.


```swift
struct InteractiveBrushingChart: View {
    @State var range: (Date, Date)? = nil

    var body: some View {
        Chart {
            ForEach(data, id: \.day) {
                LineMark(
                    x: .value("Month", $0.day, unit: .day),
                    y: .value("Sales", $0.sales)
                )
                .interpolationMethod(.catmullRom)
                .symbol(Circle().strokeBorder(lineWidth: 2))
            }
            if let (start, end) = range {
                RectangleMark(
                    xStart: .value("Selection Start", start),
                    xEnd: .value("Selection End", end)
                )
                .foregroundStyle(.gray.opacity(0.2))
            }
        }
        .chartOverlay { proxy in
            GeometryReader { nthGeoItem in
                Rectangle().fill(.clear).contentShape(Rectangle())
                    .gesture(DragGesture()
                        .onChanged { value in
                            // Find the x-coordinates in the chart’s plot area.
                            let xStart = value.startLocation.x - nthGeoItem[proxy.plotAreaFrame].origin.x
                            let xCurrent = value.location.x - nthGeoItem[proxy.plotAreaFrame].origin.x
                            // Find the date values at the x-coordinates.
                            if let dateStart: Date = proxy.value(atX: xStart),
                               let dateCurrent: Date = proxy.value(atX: xCurrent) {
                                range = (dateStart, dateCurrent)
                            }
                        }
                        .onEnded { _ in range = nil } // Clear the state on gesture end.
                    )
            }
        }
    }

    let data: [(day: Date, sales: Int)] = [
        (day: date(year: 2022, month: 5, day: 8), sales: 168),
        (day: date(year: 2022, month: 5, day: 9), sales: 117),
        (day: date(year: 2022, month: 5, day: 10), sales: 106),
        (day: date(year: 2022, month: 5, day: 11), sales: 119),
        (day: date(year: 2022, month: 5, day: 12), sales: 109),
        (day: date(year: 2022, month: 5, day: 13), sales: 104),
        (day: date(year: 2022, month: 5, day: 14), sales: 196),
        (day: date(year: 2022, month: 5, day: 15), sales: 172),
        (day: date(year: 2022, month: 5, day: 16), sales: 122),
        (day: date(year: 2022, month: 5, day: 17), sales: 115),
        (day: date(year: 2022, month: 5, day: 18), sales: 138),
        (day: date(year: 2022, month: 5, day: 19), sales: 110),
        (day: date(year: 2022, month: 5, day: 20), sales: 106),
        (day: date(year: 2022, month: 5, day: 21), sales: 187),
        (day: date(year: 2022, month: 5, day: 22), sales: 187),
        (day: date(year: 2022, month: 5, day: 23), sales: 119),
        (day: date(year: 2022, month: 5, day: 24), sales: 160),
        (day: date(year: 2022, month: 5, day: 25), sales: 144),
        (day: date(year: 2022, month: 5, day: 26), sales: 152),
        (day: date(year: 2022, month: 5, day: 27), sales: 148),
        (day: date(year: 2022, month: 5, day: 28), sales: 240),
        (day: date(year: 2022, month: 5, day: 29), sales: 242),
        (day: date(year: 2022, month: 5, day: 30), sales: 173),
        (day: date(year: 2022, month: 5, day: 31), sales: 143),
        (day: date(year: 2022, month: 6, day: 1), sales: 137),
        (day: date(year: 2022, month: 6, day: 2), sales: 123),
        (day: date(year: 2022, month: 6, day: 3), sales: 146),
        (day: date(year: 2022, month: 6, day: 4), sales: 214),
        (day: date(year: 2022, month: 6, day: 5), sales: 250),
        (day: date(year: 2022, month: 6, day: 6), sales: 146)
    ]
}

func date(year: Int, month: Int, day: Int = 1) -> Date {
    Calendar.current.date(from: DateComponents(year: year, month: month, day: day)) ?? Date()
}
```

# Wrap up
* create charts by composing marks
* Plot data with mark properties
* Customize a chart
[[Design app experiences with charts]]
[[Design an effective chart]]


* https://developer.apple.com/documentation/Charts
* https://developer.apple.com/documentation/Charts/Creating-a-chart-using-Swift-Charts
* https://developer.apple.com/documentation/charts/visualizing_your_app_s_data