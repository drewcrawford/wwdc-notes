My app adopted a background mode, but it's not running as much as I'd like it to.  Why?
We recommend [[Advances in App Background Execution - 19]]

# Goals
app goals
* updated content
* freshness
* reliability

system goals
* all-day battery
* performance
* privacy
* respecting user intent


# Factors affecting runtime
* critically low battery
* background app refresh switch
* airplane mode
* low power mode
* ongoing iCloud restore
* settings
* display on/off state
* device temperature
* system budgets
* process contention
* app usage
* app switcher
* rate limiting
* camera in-use
* device lock state

We'll cover the top 7.

1.  Critically low battery
2.  low power mode
3.  app usage
4.  app switcher
5.  background app refresh switch
6.  system budgets
7.  rate limiting

## critically low battery
* below 20% battery
* system preserves battery
* essential work only

## low power mode
* user indicated
* system limits background activity
* restrict non-essential activity

```swift
ProcessInfo.processInfo.isLowPowerModeEnabled
NSProcessInfoPowerStateDidChange
```

## app usage
Sometimes the system must prioritize certain apps over others.
System will prioritize apps that are most important to the user.  e.g., the apps they use the most.
We sometimes have indirect cues from the user on which app they need to run...

## app switcher
If the user swipes up to remove an app, we don't run it.  When the system determines which apps to run, it constrains to the apps still visible in the app switcher.

## background app refresh switch
May not be obvious that this applies to every mode of background execution.  

```swift
UIApplication.shared.backgroundRefreshStatus
UIApplication.backgroundRefreshStatusDidchangeNotification
```

## system budgets
To ensure that opportunistic activities do not deplete data, we have energy and data budgets.  Every time an app runs, it deducts.
Budgets are slowly made available throughout the day.
Limit your work per launch.

## rate limiting
Background launch frequency
system managed
Different for each background mode.

# Background App Refresh tasks
Refreshing a social media feed, emails, etc.    You might expect that they would be evenly distributed.  In reality, you may observe a skew.  Why?

System learns usage patterns and understood that the user launches app at specific times.  Launches were shceduled just before typical launch times.

Gap in launches?  Maybe the user was in low power mode.  Maybe the battery was critically low.  Any factors could have played a role.  But you have the most influence over system budgets.

* minimize power consumption
* Avoid using unneeded hardware
* Finish work as quickly as possible
* Signal completion via completion handler

Minimize cellular data usage
* only download waht's necessary
* thumbnails instead of full images
* Enqueue URLSession background transfers

Keep cellular usage under 100kb each refresh.  
# Background pushes
Alerts the application, not necessarily the user
Low priority and preserve power
* muted message conversations
* email pushes
* New episode

Note that the system does *not* gate background pushes by app usage.

How does rate limiting apply to background pushes?  Instead of running upon every push, system delays the delivery of some pushes to limit the amount of execution.  In our diagram, 14 pushes results in 7 launches.

Note the time interval between runs in this diagram was 15 minutes.  So we had many times per hour, and the app will be up to date before the user launches it.  Focus on the interval rather than the delivery rate.

# URLSession background transfers
Transfers managed by the system
Continues after app exits
Unconstrained by runtime limits
* file downloads
* Photo uploads
* Social media feed content

Can prevent transfer from using cellular.
Can request `sessionSendsLaunchEvents`.  Requests that the system launch your app after the background transfer is complete.

There are 2 main types of background URL sessions, discretionary and nondiscretionary.

`isDiscretionary` indicates work that can be deferred.  Perhaps when the phone is plugged in and on wifi.
Work enqueued from the background is always discretionary.

For discretionary transfers, we ignore factors "app usage" and "rate limiting".

If you use `sessionSendsLaunchEvents`, we will attempt to launch you if conditions permit.  Distinct from the transfers, we have rate limiting for launches.  If the app was run recently, rate limiting won't apply.

For non-discretionary transfers that you've enqueued in the foreground, transfers continue after you leave the app.  For these, only 2 factors apply: app switcher and system budgets.  Budgets are greatly relaxed and unlikely to get in your way.

If you request `sessionSendsLaunchEvents`, we use 5 factors.  Skipping "app usage" and "rate limiting" (but respecting "low power mode", "critical battery", "background app refresh switch" which were not enabled for the transfer.)

# Background processing tasks
Several minutes of runtime at system-friendly times to do maintenance work
Intensive work is possible.
Runs when plugged in and the user is not actively using the device.

We use 
* "app usage" (relaxed: "if they've foregroudned your app in the last couple of weeks you should be fine"), 
* "app switcher"
*  "background app refresh switch"
*  "network budget (only)" and 
*  "rate limiting" "as long as the user plugs in every day, your bg processing task should be able to run daily"

# wrap up

Consider how top factors affect your app
Choose the right mode or modes for the job
Reduce energy and data usage

