#xcode 
#testflight 



# Organize crash reporting
Can view feedback in ASC.  
* App Store and TestFlight
* All platforms and app extensions
* Ranked issues
* Distribution metrics
* Open log in project
* Simple setup

Incredible new features we added to xc13 and crashes organizer.

1.  Speedy crash delivery
2.  crash history
	1.  1 year.  Filter by time period, etc.
	2.  Historical graph month-to-month basis
3.  More filtering options
	1.  Versions, products, destinations, app store or testflight
4.  More distribution metrics
	1.  Inspector panel shows across versions, testflight/appstore.
5.  Shareable crash reports
	1.  xc13 introduces a new way to collaborate by adding the ability of shared crash reports from the organizer.
	2.  Organizer downloads and focuses on this issue, allowing you to zero-in on the crash and start investigating
6.  TestFlight crash feedback
	1.  New inspector to show you feedback from testers

We think this added context is invaluable.

## Demo
From the sidebar on the left, I can tell that xcode has opened to the crash organizer.

In toolbar, you'll notice all filter options that are available.  Underneath the filter bar is the crashes list, to see all versions, builds, and products.

Feedback appears in an inspector panel.

Can now share crashes (with a link into xcode).

# Terminations and MetricKit
## Terminations
* Tracks unexpected terminations
* Categorized reasons
	* timing out on launch
	* system memory limit
* compare by app version
* On-screen and background

[[Ultimate Application Performance Survival Guide]]
[[Why is my app getting killed]]

## MetricKit

MetricKit can collect crash logs in code

```swift
//  Capture crash logs in your code

import MetricKit

class Subscriber: NSObject {
    override init() {
        super.init()
        MXMetricManager.shared.add(self)
    }
    
    deinit {
        MXMetricManager.shared.remove(self)
    }
}

extension Subscriber: MXMetricManagerSubscriber {
    func didReceive(_ payloads: [MXDiagnosticPayload]) {
        payloads.forEach {
            if let crashDiagnostics = $0.crashDiagnostics {
                // Begin analyzing crash diagnostic payload.      
            }
        }
    }
}
```

* MXCrashDiagnostic payloads delivered on re-launch
* macOS support

[[What's new in MetricKit]]

# Crash logs
* Devices window
* Sharing form device
* XCTest
* Console app

[[understanding crashes and crash logs - 18]]

* https://developer.apple.com/documentation/Xcode/diagnosing-issues-using-crash-reports-and-device-logs



