#swiftcharts 

Make your apps more engaging and informative. 

When designing a chart, first decide your app's experience.  When to use, how to use, and what design system will unify them.

[[Design app experiences with charts]]

# Focused
also 
* approachable
* accessible

How to design one of these charts which visualizes pancake sales over the last 30 days.

Goals can go in many directions:
* pattern of recent sales?
* range of sales?
* Value?
* Maximum
* outliers
* comparisons (days of week, locations)
* average
* minimum
* trend
* many more!

which are most important?
An effective chart focuses on a few pieces of information.

Food truck owners primarily want to see how their charts faired over time and on specific days.
* pattern, range, and values.
How did we get from these to the chart?  Well, let's go through the process.

* Marks
* Axes
* Descriptions
* Interaction
* Color

## Marks
Bar in a bar chart, line in a linechart, point in a scatterplot.
Visual elements that represent data.  Many kinds of marks, etc.

ex bars.  Line them up to represent change.  Stack to sohw proportions.  Put side by side to compare values from different categories.

* design for goals and data
	* pancake sales in the last 30 days
	* pattern, range, values

Consider pattern.  Want to see fluctuations and trajectories.  Time axis and sales axis.
One option is to use points to represent each day.  When we envision nice smooth data, points looks great.
Realistic data might make it difficult to see patterns.

> test your designs with real data.

Our needs callf or something else.  To make the pattern of sales easier to see, connect with line.  Great at representing change.

What if our foodtruck needed to close for 5 alternating days?  in this situation, segments collecting become more prominent than the values themselves.  Design for a variety of scenarios in your data.

Bar marks are more flexible.  here 0s are visible without creating a distraction.  Intuitive to read.  etc.

Since sales are cumulative, the weight of all bars correspond.

We've chosen a mark that makes a pattern of sales visually apparent.

How to represent nonvisually?
* Make accessible in VoiceOver.

Read through braille or speach.  People who are blind can use the app without needing to see, etc.

navigate elements on a chart.  Interact.  Play a sonification of the chart through audio graphs.

To make your chart smart, nonvisually acessible, design how voiceover will navigate data values.  use audio graphs.  Conveniently, #swiftcharts  does both.

[[Bring accessibility to charts in your app]]
[[Hello Swift Charts]]

We addressed one part of the question: how are pancake sales in the last 30 days – the **pattern**.  But what about range?  And values?

## Axes
Frame marks to provide references for values.  Here we lable the start/end dates.

* consider the range

What about vertical axis?  Values here depend on sales.  With axes like this, it's important to consider the range.

* Fixed => battery charge in settings app.  always 0-100%.
* Dynamic range.  No fixed maximum step count, so we adapt the vertical axis to fit the data.

Here, there's no limit to how many pancakes we sell.  We still fix the lower bound to 0.  Doing so is a good idea as it keeps bar heights meaningful.  a bar 2x the height means twice the sales.

Need more structure to interpret sales in the middle of the chart.
* Tailor density of grid lines and labels.

Gives you reference points.  More gridlines, easier it is to estimate values.  Some charts don't need gridlines and labels at all.  e.g. sneak peaks of larger charts.  

They appear in the followup detail chart, where you want to analyze values more precisely.

Here, min and max is too few.  Too many could be distracting.  Balance these factors to choose an appropriate density. In our cases we want around 4.  Note that as we place these girdlines, we use intuitive values such as multiples of 20.

Intuitive for people to read time in steps of 7 days.  5 dividers for a 30-day period.

Charts are copmlex visual elements, and our example still needs work.  How to convey the meaning in a way that's quick and intuitive?

## Descriptions
* Provide context

Pancake sales in the last 30 days.  This text should be part of the UX.  When we look at it in our app, the screen's title > Total sales gives some context.  Segmented control gives time range.

...but soemthing else we need to clarify is the vertical axis.  We could an axis label. "Pancakes sold".  But it's small and off to the side.  We want the meaning to be obvious.

Contextualize the data with a title?

> Pancakes sold

as title.  Providing context is important.  Can make them better  by

* sumamrize the amin take-away

e.g. above precipitation, we say *Light rain forecasted*.  Light rain is expected to start for 9 minutes and last for 30 minutes.

Let's make our title

> Total sales
> 1234 pancakes

summarizes the most critical information.  Eases readers into your charts.  Makes a chart more approachable and accessible for everyone.  Examining the details of chart may be time-consuming or challening

* Use audio graphs
Adds important descriptions for voiceover.  With audiographs, #voiceover  can describe the axes.

Audio graphs provide several summaries about the data, including one you can customize.  Descriptiosn of the x/y axes are *critical* for communicating the chart.  Ensure voiceover has access some way.

We can make it even more effective with

## interaction
Empower people to explore and understand their data at a deeper level.  Higlihgt sections, etc.    Toggle between days/weeks/months/years.

* Use large touch targets

Instead of makign them the same size as the amrks, adding padding and stretchign to full height makes it easier to use our tool.  Scrub across, etc.

Interaction isn't only about touch.  People use many ways.

* Design multiple types of input

1.  Touch
2. mouse
3. keyboard
4. voice ocntrol
5. switch control
6. voiceover

* Design accessibility labels

Ensure vO can access data values.  When VO navigates onto one of the bars, it reads values like 
> June 1, 36 pancakes

It's:
* succint
* spell entire words
	* VO reads out "June" instead of Jun or "6-1"
	* 36 is the number of pancakes
* Context first
	* Makes it easier to quickly look for a specific value
	* It also makes the data easier to interpet, you know where you are

ex, cycling chart shows vertical bars but no visible labels, just elevation and such.  Too many to navigate over individually.  A well-designed label might show a section of bars as 

> From 3.6 miles to 4.4 miles: Climb 100 feet, descend 5 feet

However, if a chart is a tiny preview, maybe you summarize with one label.

One more topic:

## Color
Personality and enhance clarity.  So far, we've been designing in black and white.  You can use color to distinguish categories.

Use it to communicate intensity, such as heat in the weather forecast.  Even remove color to draw attentiont o featuers int he chart => min and max heartrate within the day.

* consider using color to enhance
Addition to make charts easier to understand and not the only way to convey critical information.

ex, cupertino vs san francisco.  We could use a line for each city, but how can we make which one be which city?

clarify with symbols - circles vs squares.  now we need a legend.  Can add color to enhance.  Some people with color blindness may not be able to distinguish color otherwise  See also: differentiate without color


* Consider associated meanings

ex, red and green for low vs charged levels.  In US, it makes sense to color green vs red for stocks.  In some countries like china, people expect the opposite.  

System blue and green work well.  NO obvious meaning tied to these colors. What if we wanted to customize colors to match the look and feel of our app?

* balance visual weight
If one overpowers the other, a hierarchy can be implied.  ex. dark purple vs bright pink.

Highlight recent activity => particular section

* Choose distinct colors
A good rule is to pick colors that are easy to differentiate by name and contrast well with each other.

Ensuring that colors have high contrast *both* from the background *and* from each other is better design and more accessible.  

Make colors distinct for people with colorblindness, use filters.

* Respect system settings
	* dark mode, light mode
	* increased contrast

# Wrap up
* marks
* axes
* description
* interaction
* color

an effective chart
* focused
* approachable
* accessible

Remember, the chart we designed is part of a chart system. Use the same design process for other charts.

https://developer.apple.com/documentation/Charts
https://developer.apple.com/documentation/Charts/Creating-a-chart-using-Swift-Charts
https://developer.apple.com/documentation/charts/visualizing_your_app_s_data
