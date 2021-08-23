#screentime

3 years since we introduced screentime for iOS.

Over the last 3 years, ST brought great new abilities.

* Monitor time spent
* Set limits
* Share usage with family
* Communication limits

# Screen Time API
Available on iOS, iPadOS 15.
100% swift and swiftUI code for easy integration.
Screen time API is designed/built with 3 guiding principles.

1.  Provide restrictions
2.  Protect privacy
3.  Enable new experiences

Frameworks, taken together comprise the API.

* Managed settings
* Family controls.  Drives privacy policy
* Device activity.  Giving your app great new ability to run code without launching your app

## Managed settings framework
* Set restrictions on device and keep them in place
* Provide web content filtering
* Shield apps and websites with custom UI

## Family controls
Leveraging family sharing, prevents access without guardian approval.

* Authorizes access to screen time API
* Prevents removal and circumvention
* Privacy preserving tokens for apps used by your family

## Device activity
How do you execute your code?  DeviceActivity schedules an event.
* Executes code on start and end of Device Activity schedules
* Executes code on usage threshold

## Combined
1.  Install on both devices. 
2.  Approve on child device
3.  Parent device sets rules
4.  Child's device => create schedules, events
5.  Extension called when schedule/event occurs

# Demo
* Setup and authorizing family controls
* Shield discouraged apps
* Remove shields for meeting goal
* Customize the shields

```swift
AuthorizaitonCenter.shared.requestAuthorization ...
```

This shows a prompt, requiring parental control.

To prevent misuse, return failure if signed in icloud is not family sharing child.

* Can't sign out of iCloud
* On-device web content filters will be installed automatically

## Shield discouraged apps
Subclass `DeviceActivityMonitor`.  

swiftUI `.familyActivityPicker`.  Allows guardian to choose between apps, websites, and categories used by the family.

## Accessing media restriction
* effectiveMaximumMovieRating
* effectiveMaximumTVShowRating
* No authorization is required

## Events
Called when events reach their usage threshold.  `eventDidReachThreshold`.  

Setting restriction to `nil` willr emove the shield from previously-shielded apps.

# Custom shields
Shields can be customized.  

New extension points in managed settings.  Customize look fo the sheild by specific properties.  Material, title, icon, body, etc.

Another point lets you create button handlers.

## Demo
subclass `ShieldConfigurationProvider`.  

Once this struct is configured and returned by configuration extension, OS will display these cusotmization over any app.

Now that we've styled the shield, I can use the extension point to configure the action handlers.  

Ability to defer action in the shield is very powerful because it gives 

# Wrap up
* Manged settings
* Device activity (usage)
* Family controls (authorization)

Continue to evolve based on feedback from you.  File feedback please.

* https://developer.apple.com/documentation/ManagedSettings
* https://developer.apple.com/documentation/FamilyControls
* https://developer.apple.com/documentation/DeviceActivity






