Designer on the interface team.  Today, I'll be building great app experiences using charts.

Throughout apple to enhance our products.
* Health
* Fitness
* Weather

We know that developers love them as well, and we see inspiring examples in every category.
Charts can be found everywhere, and when wlel-designed, can reveal subtleties in data that cannot be communicated easily in text.
Build visual interest.

App has a tab for entering orders, and another for viewing recent transactions.  Information could be much more useful.  This year, we're introducing Swift Charts #swiftcharts 

Principles we follow when designing experiences with charts at apple.  More informative verison of the food truck app.  To build a great experience, consider three things
# When to use charts
Common cases.
* change
* Proportion
	* completing, progressing, emptying
* Comparison

To decide whether any of these are appropaite, consider the experience first.  How will a chart support the core goals.


*Charts provide focus*.  Only the most important info should become a chart.  Charts direct attention to the information you want them to understand.

If we can use charts to turn a list of transactions into actionable information, food truck owners will welcome them.

## Key info
* Recent sales
* Popular items
* Top locations and days
## heading
* Charts provide focus

# How to use charts
To illustrate recent sales, showing change is appropriate.
* Bar chart, for each of the last 30 days?
* Title: ~~Sales in past 30 days?~~
* Describe chart contents
	* "Total sales in Past 30 days"
* Using a complete sentence can be easier.
> Sales for the past 30 days totaled 1,234 pancakes.

Title interpret the data
> Sales for the past 30 days are up 12%, totaling 1,234 pancakes

Each of these approahces is a good way of describing a chart, but this overview is just one way.

## Incorporate details
Here are some additional perspectives to consider.

* Macro => total, average
* medium => subsets.  Weekdays vs weekends, morning vs afternoon.  Style of pancakes, location where sold, etc.
* micro level => last transaction, largest sale.  May want to call out in your charts.

Some perspectives we've identified could be useful.

maybe augment the chart with these details.  Set of rows under the chart?  Each row provides a summary statistic, and the chart is updated to match.  Daily average, difference between weekday vs weekend, etc.

This amount of info requires a large surface to work effectively.  As functionality increases, so will its size.

Small charts are static => watch complications, stocks, etc.  Static charts rarely exist in isolation, provide a preview of a larger chart in another view.  Don't require gridlines, labels, or interactivity since they create the expectation that additional detail is just a tap away

Interactive charts are larger and include more detail.  Typically the width of your view but not ful-height.  Include axis lines, labels, interactivity.  Ability to change the time range or time scope will aid exploration.

Largest and most interactive charts will require deep exploration of data.  Important to introduce additional functionality gradually.

## Progressively reveal complexity
Use a small static chart higher in the nav hierarchy
path to expanded versions of the chart.
Progression should maintian continuity by preseving values, context, and state.  Keep in mind that when someone xpresses interest in the chart, they want to see more of what they've already seen.  maintain shape and numbers apparent in an earlier view.

Add information, but showing something different can be frustrating or disorienting.

We currently have two tabs
* placing orders
* sales

Interactive chart will support detailed anaysis with various views, etc.


## wrap up
* describe chart contents
* incorporate details
* progressively reveal complexity

# chart design systems
When your app includes more than one chart, you've created a chart design system.

## Use familiar forms
start with common chart style to aid comprehension.  If someone has already used a similar chart, they're more likely to understand yours.  Bar chart, line chart, etc.  Scatter plot is less common, may require extra guidance.

Introduce things clearly.  e.g. the activity rings aniamtion.  Then we split them apart.

Ideally a new form is central to your ap, not supplementary.  Prominence will encourage peopel to explore and understand it.

Supporting charts => use familiar forms

## Differences matter
consider chart variations.
1.  Change time scope to be 12 months?
2. Changing items => change color to highlight the difference?  This ensures someone will read the description.
3. Range of sales.  Maybe a change the way it's represented.  modifying the style of bars is appropriate.  Now conveys a different subject, time range, and metrics.  Purposely distinct.

Apply these principels to the other two charts.

To complement recent sales => most popular style.    We can use a bar with different colors.
Separating bars compares sizes more clearly but it starts to look like a time series.
By making them horizontal, I can accentuate the difference.  Also lets me make the bars longer.  I've omitted labels tof ocus only on the top style.

Sales on the two cities where the chart operates.  Split bars per location.  Convert into lines? Chart title summarize the data.

## wrap up
* Use familiar forms
* differences matter
# wrap up
* when to use charts
* how to use charts
* chart design systems

[[Design an effective chart]]
[[Hello Swift Charts]]


https://developer.apple.com/documentation/Charts
https://developer.apple.com/documentation/Charts/Creating-a-chart-using-Swift-Charts
https://developer.apple.com/documentation/charts/visualizing_your_app_s_data