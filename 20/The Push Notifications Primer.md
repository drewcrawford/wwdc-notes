# Introducing Push Notification
* engagement
* provide updates
* dynamic experience

## advantages
* foreground not required
* Power efficient
* Best engagement

* Alert notifications -> visible alerts
* Background notifications -> allow your app to get runtime when it's not in the foreground


# Implementing Alert Pushes
[[Designing Notifications - 18]]
* visible alerts
* display new information
* can be interactive
* foreground or background
* customizable

```swift
//registering for remote notifications
class AppDelegate: UIResponder, UIApplicationDelegate, UNUserNotificationCenterDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        UIApplication.shared.registerForRemoteNotifications()
        UNUserNotificationCenter.current().delegate = self
        return true
    }
```

```swift
//app delegate callbacks
func application(_ application: UIApplication,
                   didFailToRegisterForRemoteNotificationsWithError error: Error) {
    // The token is not currently available.
    print("Remote notification is unavailable: \(error.localizedDescription)")
}

func application(_ application: UIApplication,
                   didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
     // Forward the token to your provider, using a custom method.
     self.forwardTokenToServer(token: deviceToken)
}
```

```swift
func forwardTokenToServer(token: Data) {
    let tokenComponents = token.map { data in String(format: "%02.2hhx", data) }
    let deviceTokenString = tokenComponents.joined()
    let queryItems = [URLQueryItem(name: "deviceToken", value: deviceTokenString)]
    var urlComps = URLComponents(string: "www.example.com/register")!
    urlComps.queryItems = queryItems
    guard let url = urlComps.url else {
        return
    }

    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        // Handle data
    }

    task.resume()
}
```

```swift
//request authorization
@IBAction func subscribeToNotifications(_ sender: Any) {
    let userNotificationCenter = UNUserNotificationCenter.current()
    userNotificationCenter.requestAuthorization(options: [.alert, .sound, .badge]) { (granted, error) in
        print("Permission granted: \(granted)")
    }
}
```

```js
{
    "aps" : { //how to render the notification
       "alert" : { //what text to use for the notification
            "title" : "Check out our new special!",
            "body" : "Avocado Bacon Burger on sale"
        },
        "sound" : "default", //should be included when you want the device to play a sound
        "badge" : 1, //absolute value
   },
   //custom data
    "special" : "avocado_bacon_burger",
    "price" : "9.99"
}
```
```swift
//did receive response
func userNotificationCenter(_ center: UNUserNotificationCenter,
                            didReceive response: UNNotificationResponse,
                            withCompletionHandler completionHandler: @escaping () -> Void) {
    let userInfo = response.notification.request.content.userInfo
    guard let specialName = userInfo["special"] as? String,
          let specialPriceString = userInfo["price"] as? String,
          let specialPrice = Float(specialPriceString) else {
        // Always call the completion handler when done.
        completionHandler()
        return
    }

    let item = Item(name: specialName, price: specialPrice)
		addItemToCart(item)
  	showCartViewController()
    completionHandler()
 }
 ```

# Implement Background Pushes
Similar to alert pushes, but there are some differences.
[[Background Execution Demystified]]

* fetch data in background
* stay up to date
* Will launch if necessary
* System managed.  Only so many bg operations per day.  BG updates will not be performed if the device is under certain constraints, such as low battery.

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
       UIApplication.shared.registerForRemoteNotifications()
       return true
    }
```

Do not need to make your application a `UNUserNotificationCenterDelegate`.  That's because the `UNUserNotificationCenterDelegate` is only used for alert notifications.

```js
{
    "aps" : {
       "content-available" : 1 //only field that's required.
	   //tells the system that this is a BG notification
	   //and your app should be launched
    },
    "myCustomKey" : "myCustomData"
}
```

```swift
func application(_ application: UIApplication,
                     didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                     fetchCompletionHandler completionHandler:
                     @escaping (UIBackgroundFetchResult) -> Void) {
    guard let url = URL(string: "www.example.com/todays-menu") else {
        completionHandler(.failed)
        return
    }

    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        guard let data = data else {
            completionHandler(.noData)
            return
        }
  
        updateMenu(withData: data)
        completionHandler(.newData)
    }
}
```

# Next steps
* Developer portal
* Implementation
* Download the sample app


