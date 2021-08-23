# Key performance metrics
How to prioritize?  Xcode organizer
* Battery
* launch time
* hang rate
* memory
* disk writes
* scrolling
* terminations

Insight section to highlight performance priotities.

# Discovering regressions
App performs poorly in a power/performance area relative to recent releases.

If a metric is trending up and latest value is higher than average, it's flagged as a regression.

Regressions tab: one-stop shop.  

What are terminations?  [[Why is my app getting killed]]

[[Triage TestFlight crahes with Xcode Organizer]]

Task timeouts have increased across all iPhones for typical and top percentiles.  Apps have 30 seconds to execute tasks before they're terminated.  Failing to end tasks appropriately can cause your app to terminate, causing slow launch.


# Disk write insights
Why are disk writes so important?  
* Good device health
* Better performance
* Longer battery life

These reporst are collected from devices which share info.  Stacktrace.

For each signature, we can find detailed stack traces showing the cause of the write, and statistics.  To identify the problem area, see the top signature.

Insights can decode your stack trace to understand e.g. sqlite stuff.

Reduce app writes and improve performance.  
* Use of serliazed files – plists, xml, json
* Sqlite configuration
* etc

# App Store Connect APIs
* Metrics
* Diagnostics

[[Identify Trends with the Power and Performance API]]

Populations section provides detailed structure about devices etc.

Diagnostic signatures.  will have a link to details.

Insights available via API as well.Take action immediately.  

# Next steps
* Discover your apps performance regressions
* Implement optimization suggestions
* Plan to stay up to date on your app's performance

