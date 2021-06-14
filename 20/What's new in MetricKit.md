#metrickit

# Using MetricKit
Designed to provide data in phases where you wouldn't ordinarily have the app.  Such as betatesting or production.

1.  Link MetricKit
2.  Instantiate `MXMetricManager`
3.  Implement `MXMetricManagerSubscriber` delegate

Daily aggregated metrics - `MXMetricPayload`.  

# New Metrics
## CPU instructions
Daily cumulative instructions retired
Allows for quantification of total workload

## Scroll hitches
User perceptive animation delays
Time ratio for `UIScrollView` scrolling
[[Eliminate animation hitches with XCTest]]

## App exit reasons
Reasons for application exit (terminations)
Foreground and background reasons
[[Why is my app getting killed]]

# Diagnostics
Diagnostics are needed for root cause analysis.
* Targeted for specific instances
* actionable

Implement `MXMetricManagerSubscriber` delegate.  Functions identically to MetricKit metrics.  Maps 1:1 with the metric payload.

`MXDiagnostic` contains metadata at regression time.  There is specific data for each diagnostic.  But consistently `MXCallStackTree`.  Unsymbolicated.

[[Identify Trends with the Power and Performance API]]

## Hang rate
`MXHangDiagnostic`  Unresponsive for long periods.  Time spent hanging, backtrace of main thread.

## CPU exception (power)
"CPU exceptions".  CPU time ocnsumed, total sampled time, backtraces of threads spinning.

## `MXDiskWriteExceptionDiagnostic`
Total writes caused, backtrace.  1GB threshold.

## `MXCrashDiagnostic`
Exception type, code, signal, termination reason, VM in (for bad access), backtrace.


# Wrap up
[[Diagnose Performance Issues with the Xcode Organizer]]
[[Eliminate animation hitches with XCTest]]