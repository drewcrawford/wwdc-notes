Learn how you can get iPhone involved in your Apple Watch-based workout apps with HealthKit. We'll show you how to mirror workouts between devices and take a ride with cycling data types. Plus, get to know HealthKit for iPad.

Healthkit provides centralized database to show your users data, etc.

Workout api provides some of most powerful apis

Mirrored workout session APIs
New cycling data types
HealthKit on iPad

# Mirror workout on iPhone
[[New ways to work with workouts - 18]]
[[Build a workout app for Apple Watch]]

First I set up a handler using the HealthStore.  Ready to receive the session from my apple watch.  Every time iPhone app is launched in fg/bg, i'll implement the mirrored start handler to receive the active workout session passed from apple watch.

Start with an activitytype of cycling.  Call startWatch api to launc the app on my paired apple watch and pass the workout configuration.

HK launches companion app in the bg, gives me 10 seconds to start live acctivity, and calls handler to start mirroring.  



# Collect new cycling metrics
* speed, power, cadence
* Support for bluetooth cycle devices
* Healthkit to automatically calculate/save FTP (functional threshold power)


# Display workouts on iPad

Ensrue that authorization sheet is shown over appropriate scene.  

`healthDataAccessRequest` view modifier in SwiftUI.  

* update authorization for iPad
* Update support for cycling metrics
* Check out mirrored session APIs
* Use sync identifiers/version numbers to keep data consistent across server/devices
* file feedback!

[[Synchronize health data with HealthKit]]

# Resources
* https://developer.apple.com/documentation/healthkit/workouts_and_activity_rings/building_a_multidevice_workout_app
* https://developer.apple.com/documentation/healthkit
* https://developer.apple.com/documentation/Updates/HealthKit
* https://developer.apple.com/documentation/Updates/watchos
