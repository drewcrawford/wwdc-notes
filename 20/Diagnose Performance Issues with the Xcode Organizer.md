Providing you with actionable tools and data.

# Xcode Organizer
What is the Xcode organizer and how do I get to it?

We aggregate data and send back to you through the organizer

[[Improving Battery Life and Performance - 19]]
[[What's New in Energy Debugging - 18]]

We provide variou smetrics
* battery usage
* launch time
* hang rate
* memory
* disk writes

# New data in the Organizer
* scroll hitches
* disk write diagnostics

## hitches
What is a scroll hitch?
When a rendered frame does not end up on-screen at an expected time, when a user is scrolling in your app.

This usually causes frames to be dropped

Hitch time (ms) / scroll duration (s).  Result is the "hitch rate".  Estimate of the severity of hitches as perceived by the user.

<5ms/s -> good
5-10ms -> warning
\>10ms/s -> poor

[[Eliminate animation hitches with XCTest]]

## disk write diagnostics
Created when an app performs more than 1GB of logical writes within 24 hors
Optimal use of storage ensures
* better performance
* longer battery life
* good device health

# demo

Can compare versions of the app..

We're thrilled to show data even sooner.  In xcode 12, we've lowered the required usage threshold by a factor of 5.  If your app's usage was below the old threshold, you might see data for the first time.

Increase of disk writes

# Next steps
* compare metrics for different versions of your app
* Discover any scroll hitches in your app
* Deep dive into disk writes

