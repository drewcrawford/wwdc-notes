#carplay 

The smarter, safer way to use your iPhone in the car.

We'll start with a quick refresher on apps that are supported.  New app types.  New tool to support the development of your apps.  Navigation apps.

# CarPlay apps
* primarily intended for drivers
* Only target usecases relevant while driving
* omit compelx and less common use cases
* Entitlement is needed

App types
* navigation
* audio
* communication
* EV charging parking
* quick food ordering

Many of you have even more apps to enable.  Adding
* fueling
* driving task

## Templates
Your app supplies data
iOS draws UI on your behalf
Your app is appropriate for the road
* font sizes
* low complexity
* consistency
* app UI works great in all cars

You ahve several tempaltes to choose from in yoru app.  From grid template to the list template showing a table.  Tehse templates should be familiar as a developer and as an iOS user.

They will be familiar to your users driving with carplay as they appear all throughout carplay.

Which templates can you use?

|                   | audio | communication | driving task | ev charging | fueling | navigation | parking | quick food ordering |
| ----------------- | ----- | ------------- | ------------ | ----------- | ------- | ---------- | ------- | ------------------- |
| action sheet      |   no    | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| alert             | yes   | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| grid              | yes   | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| list              | yes   | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| tab bar           | yes   | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| information       | no      | yes           | yes          | yes         | yes     | yes        | yes     | yes                 |
| point of interest |  no     | no            | yes          | yes         | yes     | no         | yes     | yes                 |
| now playing       | yes   | no            | no           | no          | no      | no         | no      | no                  |
| contact           | no    | yes           | no           | no          | no      | yes        | no      | no                  |
| map               | no    | no            | no           | no          | no      | yes        | no      | no                  |
| search            | no    | no            | no           | no          | no      | yes        | no      | no                  |
| voice control     | no    | no            | no           | no          | no      | yes        | no      | no                  |

Fear not, you'l find this chart in our developer docs.  The thign to take away is that teh templates you can use depend on type.  Only tempaltes that are relevant and appropriate are permitted.

## Fueling apps
* apps designed primarily to help users fuel cars
* similar to EV charging, but for other fuel types
* should be more than just finding locations
* actions at the pump

## Driving task apps
* enable a wider range of very simple tasks
* Must primarily target use cases for drivers
* Examples
* controlling car accessories
* driving or road status and information
* tasks at start or end of drives

ex: road status.  Informs users about important road information.  Built using POI template.  
ex: trailer controller.  Users are best served doing non-driving tasks using iPhone.
ex: Mileage logger.  
ex: Express lane transponder.  

* consider singelscreen apps with minimal functionality
* Target tasks that can be completed in a few seconds
* do not enable complex or infrequent use cases, first-time setup, etc.
* don't enable tasks not needed while driving, even if car-related

## CarPlay simulator
ways to test
* xcode simulator
	* built-in carplay window
* iPhone in CarPlay car, or with aftermarket head unit
* iPhone with CarPlay simulator

Mac app that replicates a CarPlay environment
Download "Additional tools for Xcode"
Connect your iPHone to your Mac with a cable

beneifts
* test your app on a real iPhone
* convenience of testing at your desk
* full suite of developer tools
* complete iPhone functionality
* test multiple car configurations

Configuration
* display size
* fps
* driving orientation

typical sizes to test
* 800x480
* 1280x720
* 900x1200

cluster display
see an instrument cluster stream.  Most relevant to navigation apps.  Instrument cluster displays either the map or turn car in their fov.


## Maps in instrument cluster
Back in iOS 13, we added APIs to enable navigation apps to appear int he dashboard.  you edited your info plist and implemented required delegates.  They notify your app when it is appearing/disappearing in dashboard, and gives you a UIWindow.

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <!-- Indicate support for CarPlay dashboard -->
    <key>CPSupportsDashboardNavigationScene</key>
    <true/>
    <!-- Indicate support for instrument cluster displays -->
    <key>CPSupportsInstrumentClusterNavigationScene</key>
    <true/>
    <!-- Indicate support for multiple scenes -->
    <key>UIApplicationSupportsMultipleScenes</key>
    <true/>
    <key>UISceneConfigurations</key>
    <dict>
        <!-- For device scenes -->
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>Phone</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppWindowSceneDelegate</string>
            </dict>
        </array>
        <!-- For the main CarPlay scene -->
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlaySceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay Dashboard scene -->
        <key>CPTemplateApplicationDashboardSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationDashboardScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Dashboard</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayDashboardSceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay instrument cluster scene -->
        <key>CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationInstrumentClusterScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Instrument-Cluster</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayInstrumentClusterSceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

```swift
extension TemplateApplicationSceneDelegate: CPTemplateApplicationInstrumentClusterSceneDelegate {
    
    func templateApplicationInstrumentClusterScene(
        _ templateApplicationInstrumentClusterScene: CPTemplateApplicationInstrumentClusterScene,
        didConnect instrumentClusterController: CPInstrumentClusterController) {
        // Connected to Instrument Cluster
        TemplateManager.shared.clusterController(instrumentClusterController, didConnectWith: templateApplicationInstrumentClusterScene.contentStyle)
    }
    
…

    func instrumentClusterControllerDidConnect(_ instrumentClusterWindow: UIWindow) {
        // Window in which to draw instrument cluster contents 
       self.instrumentClusterWindow = instrumentClusterWindow
    }
```
A few more considerations specific to the cluster.  Delegate:
* zoom in and out
* show compass
* show speed limit

Your view may not be completely visible!
Override `viewSafeAreaInestsDidChange`
use SALG

## Demo
Even if your app is primarily template-based, you can use it to ensure artwork works great in light/dark.  

I'm certain drivers using your navigation app will love it too.  For more info, check out carplay developer portal.


* https://developer.apple.com/forums/tags/wwdc2022-10016
* https://developer.apple.com/forums/create/question?&tag1=50&tag3=452030
* https://developer.apple.com/carplay
* https://developer.apple.com/design/human-interface-guidelines/carplay/




