# What are hangs
https://www.youtube.com/watch?v=qJRKedSUHg4

New app we're developing.  Food truck, which will help manage food trucks which sell donuts.  

App's main thread is responsible for processing user interactions.  Busy doing work, waiting on another system resources.  >=250ms.  

Mian thread is also unavailable to process new user interactions until the hang is resolved.  It appears the app is compeltely stuck.

* People avoid apps that are frequently unresopnsive
* Force quits
* switch apps
* delete your app

Trackign down hangs is critical to gaining and maintaining user base.  Ensure people will enjoy using your app.

[[Understand and eliminate hangs from your app]]

1.  Development
2. beta
3. public release

MetricKit supports collecting non-aggregated methods from individual users on your beta or public release app.
Xcode Metrics organizer provides aggregated metrics.  There are gaps here, specifically when developing.  Or when trying to understand what sourcecode has caused.

We've been busy introducing several new tools to help.  Let's introduce each of them briefly before we cover them in more detail.

* Thread performance checker
* Hang Detection in Instruments
* On-device hang detection – without xcode or tracing.  Real-time notifications and diagnostics in development signed or testlflight
* Xcode Reports Organizer

# Development tooling
Thread Performance Checker.  Notifies you in issue navigator. 

TPChecker alerts me to a hang risk caused by a priority inversion.  Higher priority thread is attempting to be synchronized with a lower priority thread.  Our hang may be caused by MT waiting on lower-priority threads.

Time Profiler instrument.  New in xcode 14, this adds hang detection and labeling.  Time profiler to ocnfirm hang occurs, that it was caused by a priority inversion, and figure out what the lower priority threads were busy doing.

Note that MT is idle.  Workther thread has lots of CPU usage.  This is likely the thread MT is waiting for.  Let's examine waht the worker thread was doing during the hang.

New standalone Hang Tracing instrument, add to any trace document.  Allows you to configure a hang duration threshold to find specific periods of unresponsiveness.

Even with great testing coverage, beta and release environment can discover new hangs.
# Beta tooling
* On-device hang detection
* Available in developer settings


Settings->Develoepr->Hang Detection

After installing your app, it will appear in the list of monitored apps.  Show chronological list of logs.  These diagnostics are best effort and processed at low priority.  Processing can take longer, espeiclaly if the system is busy.  Passive notification is displayed when new diagnostics are available.  

Text-based log and a tailspin.  For deeper investigation, open tailspin in instruments, for viewing interaction etc.  

Symbolicate and view it on a larger screen.
# Public release tooling
Supports Hang Reports.  To deliver aggregated hang diagnostics from customer device.  Collected data is from customers who consented to app analytics.  Main thread stacktrace that lead to the hangs.

Leftside navigation in xcode organizer.  Similar stack traces grouped together.  Signatures are shown sorted based on user impact.  Each hang log contains MT stack trace, hang duration, and device/os version, etc.  Each signature provides aggregate statistics about how many hang logs the signature was repsonsible for.  Breakdown by os version and device.

To identify hangs most affecting customers, pay close attention to top signatures.  Top signature repsonsible for 21% of hang time.

If app store has symbol info, we symbolicate it.  We can infer this hang was cuased by synchronously reading a file from disk.  It's important to tackle performance problems that are most affecting customers.  Check this data after each app release to validate that previous hangs were resolved, etc.

App Store connect APIs

[[Identify Trends with the Power and Performance API]]

In xc13.2, can now receive notifications monitoringpower/performance metrics.  Click to enable in the upper right hand corner.

# Performance regressions

[[Diagnose Power and Performance regressions in your app]]

# Submitting symbols
* Improves xcode organizer experience
* One-click navigation from function name in stack trace, to definition
* Only essential information extracted
	* line numbers, names, etc.
* Securely stored and never shared

# Wrap up
* fix hangs early
* Enable thread performance checker
* enable on-device hang detection
* use hang report regularly
* enable regression notifications
* submit symbols





https://developer.apple.com/documentation/Xcode/improving-app-responsiveness