#design

Widgets
Smart stacks

Remember, people can interact with widgets in different ways.

1.  Ideation
2.  Creation

# Ideation
## Principles
Need to understand the most useful information and experiences.
* personal -> allow for a deeper emotional connection
* informational -> overview of information from a variety of surfaces.  Save people from doing actions in app
* contextual -> helps surface the right info at the right moment.

Our decisions

Calendar: day of the week, date, next meeting.  Start time, location.  For 2 events, collapses less relevant information.  No more events today.  Tomorrow.  Birthdays.

Photos: Surface the best photos, not the most recent.  Featured photos, memories from today in years' past.

Weather: city name, current temp, forecast.  Change resolution depending on current events like rain.

Maps: commute, etc.

## editing
Widgets jiggle like apps, you can flip around and edit.

Weather: change location.  By default, we have your location, so you don't need to do additional work.

Principle here is rather than having a complex widget that supports 2 cities for example, we can have small widgets for each city.  

What options can you offer to people that would maximize utility and flexibility?

## multiples

For stocks, we had 2 ideas.  Summary of watch list, vs deep dive on single stock.

# Creation
## Sizes and Interactions
small, medium, large.

small supports a *single* tap target.  Deep link somewhere in the widget.

Calendar -> links into day view.
News -> Brings you to news story

3 tap styles.  These appear to influence the region that is highlighted, but not the animated transition.

Fill style is best when you deep link into a piece of content.  Every small widget is fill-style, since it only supports 1 tap target.

Cell-style is best when you're selecting an item in a wiget that lives in its own shape, like here in our files widget.

Content-style is great for when you're selecting a piece of content that lives uncontained in a widget.  Effect here seems to be applying a touchup/highlight-like effect to the text.

## Content and Personality
*What are people looking for when they launch my app?*

Consider cribbing from visual style of the app.  e.x, our widgets look like the apps.

Or maybe from the app icon?  

## layout
Layout that expands across all 3 sizes, where we add additional info at each size.

Layout that is completely unique across sizes.  Small news displays 1 story, medium widget has a list of 2 stories..

Make sure not to scale up your smaller widget into your larger widget.  Think about info for each size.

If you don't have more info to show, it's fine to only support smaller sizes.

We recommend only displaying 4 pieces of information in the small widget size.  Think like 4 smaller icons.

## Patterns
Common layout patterns.

Follow default 16pt layout margins across all sizes.  

For layouts with 4 circles, use tighter 11pt margins.

Shape corners that appear in corners should seem concentric with the widget's corner radius.  Since corner radius changes, we provide a swiftUI containers to assign to shapes in your widgets.

Use SFPro or similar variants of SF.

Support dark mode.

Placeholders.

# Tips
Only use a logo if your app is an aggregator of content from different sources.

Logo should sit in the top-right corner.  Avoid using wordmarks in your widget.
Avoid putting app icon in widget.
Avoid putting app name in widget, since the app name appears below anyway.
Don't put text that instructs a user or talks to them, instead if youfeel there's something important, do it graphically.

When displaying chronological information, don't use words like "last updated" or "last checked", just say "20m ago"





