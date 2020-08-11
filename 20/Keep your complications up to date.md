# Keep your complications up to date
# Example app
Kite Flying.  Independent watch app.  Multiple complications.
Actively encouraging.
Location aware.
Socially connected.
# Foreground opportunities
Any time our app is launched, it’s a great time to update complications.
Use CK APIs to reload timeline.

```swift
class ExtensionDelegate: NSObject, WKExtensionDelegate {

    func updateActiveComplications() {

       let complicationServer = CLKComplicationServer.sharedInstance()

        if let activeComplications = complicationServer.activeComplications {

            for complication in activeComplications {

               complicationServer.reloadTimeline(for: complication)

            }
        } 
    }
}
```

Typically you should be more selective and only update complications that need to be updated.

```swift
//getCurrentTimelineEntry
class ComplicationController: NSObject, CLKComplicationDataSource {

    func getCurrentTimelineEntry(for complication: CLKComplication, 
        withHandler handler: @escaping (CLKComplicationTimelineEntry?) -> Void) {

        switch (complication.family) {
  
        case .modularSmall:
           let template = CLKComplicationTemplateModularLargeTallBody.init(
                               headerTextProvider: headerTextProvider, 
                               bodyTextProvider: bodyTextProvider)

            entry = CLKComplicationTimelineEntry(date: Date(), 
                        complicationTemplate: template)
        }

        handler(entry)
    }
}
```

# Background app refresh
* Up to four times per hour
* Actual number is dependent on conditions such as how many other processes are running, and battery usage.

```swift
private func scheduleBAR(_ first: Bool) {
        let now = Date()
        let scheduledDate = now.addingTimeInterval(first ? 60 : 15*60)

        let info:NSDictionary = [“submissionDate”:now]

        let wkExt = WKExtension.shared()
        wkExt.scheduleBackgroundRefresh(withPreferredDate: scheduledDate, userInfo:info)
        { (error: Error?) in
            if (error != nil) {
                print("background refresh could not be scheduled \(error.debugDescription)")
            } 
        }
   }
```

When the task is ready, our app will be made active.

```swift
//handle BAR
class ExtensionDelegate: NSObject, WKExtensionDelegate {
    func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {

        for task in backgroundTasks {

          switch task {
          case let backgroundTask as WKApplicationRefreshBackgroundTask:

                if let userInfo:NSDictionary = backgroundTask.userInfo as? NSDictionary {
                   if let then:Date = userInfo["submissionDate"] as! Date {
                      let interval = Date.init().timeIntervalSince(then)
                      print("interval since request was made \(interval)")
                   }
                }

                self.updateActiveComplications()

                self.scheduleBAR(first: false)

                backgroundTask.setTaskCompletedWithSnapshot(false)
```

```swift
//handleBAR data provider
class ExtensionDelegate: NSObject, WKExtensionDelegate {

    var healthDataProvider: HealthDataProvider

    func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
        for task in backgroundTasks {
            switch task {
            case let backgroundTask as WKApplicationRefreshBackgroundTask:

                healthDataProvider.refresh() { (update: Bool) -> Void in
                    if update {
                        self.updateActiveComplications()
                    }
                    self.scheduleBAR(first: false)
                    backgroundTask.setTaskCompletedWithSnapshot(false)
                }
```

* Only one outstanding request.  If you need periodic updates, schedule the next one before marking the current one complete
* No networking.
* Four seconds of active time
* 15 seconds of total time.  Remember to mark tasks completed!

# Background URL session
Allow your app to schedule and receive data even when the app isn’t running.

* Up to 4 requests per hour
* Multiple outstanding tasks

```swift
//instantiate background URL session
class WeatherDataProvider : NSObject, URLSessionDownloadDelegate {

    private lazy var backgroundURLSession: URLSession = {
        let config = URLSessionConfiguration.background(withIdentifier: “BackgroundWeather")
        config.isDiscretionary = false
			//so the app is launched in the background
        config.sessionSendsLaunchEvents = true

        return URLSession(configuration: config, delegate: self, delegateQueue: nil)
    }()
```

```swift
//schedule backgroundURLSessionTask
func schedule(_ first: Bool) {

        if backgroundTask == nil {

            if let url = self.currentWeatherURLForLocation(delegate.currentLocationCoordinate)
            {
                let bgTask = backgroundURLSession.downloadTask(with: url)

                bgTask.earliestBeginDate = Date().addingTimeInterval(first ? 60 : 15*60)

                bgTask.countOfBytesClientExpectsToSend = 200
                bgTask.countOfBytesClientExpectsToReceive = 1024

                bgTask.resume()

                backgroundTask = bgTask
            }
        }
    }
}
```

Don’t mark the task as completed until you want to be no longer notified!

```swift
//handle task completion
class ExtensionDelegate: NSObject, WKExtensionDelegate {

   var weatherDataProvider:WeatherDataProvider

    func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
       for task in backgroundTasks {
           switch task {
            
                case let urlSessionTask as WKURLSessionRefreshBackgroundTask:

                    weatherDataProvider.refresh() { (update: Bool) -> Void in
                        weatherDataProvider.schedule(first: false)
                        if update {
                            self.updateActiveComplications()
                        }
                        urlSessionTask.setTaskCompletedWithSnapshot(false)
                    }
```

```swift
//didFinishDownloadingTo
class WeatherDataProvider : NSObject, URLSessionDownloadDelegate {
    func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {
        if location.isFileURL {
            do {
                let jsonData = try Data(contentsOf: location)
                if let kiteFlyingWeather = KiteFlyingWeather(jsonData) {
                    // Process weather data here.
                }
            } catch let error as NSError {
                print("could not read data from \(location)")
            }
        }
    }
```

```swift
class WeatherDataProvider : NSObject, URLSessionDownloadDelegate {
func urlSession(_ session: URLSession, task: URLSessionTask, 
                     didCompleteWithError error: Error?) {
        print("session didCompleteWithError \(error.debugDescription)”)
        DispatchQueue.main.async {
           self.completionHandler?(error == nil)
            self.completionHandler = nil
        }
    }
}
```

## Intermediate requests
`urlSession(willBeginDelayedRequest:)`.  Might update just before request starts if it’s been a long time since scheduled
`urlSession(didReceiveChallenge:)` -> challenges
`urlSessionDidFinishEvents` when all events are delivered.
Don’t schedule a new task!  The current task has not been completed.
Instead just call `setTaskCompletedWithSnapshot()`.

## Guidelines
* Four seconds of active time
* 15 seconds of total time

# Complications pushes
* 50 per day per watch
* bundleID ends in `.watchkitapp.complication`.  If your bundle ID isn’t in this format, it may be rejected.
* Create a certificate based on this.
* Needs “Remote Notification” background mode
* And push notification capabilities.
* Use PushKit.

```swift
class PushNotificationProvider : NSObject, PKPushRegistryDelegate {

    func startPushKit() -> Void {
        let pushRegistry = PKPushRegistry(queue: .main)
        pushRegistry.delegate = self
        pushRegistry.desiredPushTypes = [.complication] //matches .complication certificate
    }
    func pushRegistry(_ registry: PKPushRegistry, 
                      didUpdate pushCredentials: PKPushCredentials, for type: PKPushType) {
        // Send credentials to server 
    }
    func pushRegistry(_ registry: PKPushRegistry, 
                        didReceiveIncomingPushWith payload: PKPushPayload, 
                        for type: PKPushType, completion: @escaping () -> Void) {
        // Process payload
        delegate.updateActiveComplications()
        completion()
    }
```

```js
{
    “Aps”: {content-available”: 1}, //needs this
    “Encouragement”:”high”,
    “Count”:42
}
```

* 50 per day
* Doesn’t change with multiple complications
* 4 seconds active time
* 15 seconds total time

|                        	| Data Source     	| How Often                  	| Guidelines                        	| Delivery              	| Scheduling  	|
|------------------------	|-----------------	|----------------------------	|-----------------------------------	|-----------------------	|-------------	|
| Foreground updates     	| Local or remote 	| Whenever user launches app 	| Active + ProcessInfo              	| Application Lifecycle 	| N/A         	|
| Background App Refresh 	| Local           	| 4 per hour                 	| Four CPU seconds 15 seconds total 	| WatchKit              	| WKExtension 	|
| URLSession Background  	| Remote          	| 4 per hour                 	| Four CPU seconds 15 seconds total 	| WatchKit + URLSession 	| URLSession  	|
| Push Notifications     	| Remote          	| 50 per day                 	| Four CPU seconds 15 seconds total 	| PushKit               	| Server      	|
[[Meet watch face sharing]]
[[Build complications in SwiftUI]]
[[Creating independent watch apps - 19]]
