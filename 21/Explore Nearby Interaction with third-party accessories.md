U1.  Precise and spatially-aware interactions between nearby devices.  Powers precision finding with AirTag and fluid handoff gestures between iPod and HomePod mini

# Recap
Creating and running a session `NISession`

Conform to NISessionDelegate.

Create NIConfiguration.  
`NINearbyPeerconfiguration`

Once you can `.run`, NI will start providing your app a stream of nearby object updates, each containing distance and optionally direction with nearby devices in the session.

[[Meet Nearby Interaction]]


# Permission prompt updates
Prompt appears the first time your app runs a session in the new lifetime.  
Since this permission is one-time, it can lead to additional prompts in certain situations.

this year, you can grant apps permission in a new way.

1.  System shows prompt first time your app runs `NISession`.  Important to make sure that the timing coincides with a clear user intent
2.  OK => Permission while the app is in use.  Whether the user accepts or denies, permission prompt will not show again.
3.  Starting in iOS 15, Nearby Interaction appears in Settings.  If users change their mind, they can head over to Settings to change.

* Persistent while the app is in use
	* Requires purpose string in Info.plist
	* `NSNearbyInteractionusageDescription`
* Users can change permission in Settings
* Inspect session invalidation error code

# Third-party accessory API
fira consortium
NXP, qorvo
Hardware and ifrmware that will interoperate.
U1-compatible development kits
Sample app code
Together, serves as a starting point
Specification for accessory manufacturers (dev preview)
developer.apple.com/nearby-interaction

Suppose you have two regions.  Larger region, fucntionality A.  Small region, functionality B.

Expects your app and accessory to have some capability to exchange data.  particular technology is up to you.

Suppose your accessory already supports bluetooth.  You'll be able to utilize your existing bluetooth capability for data exchange.  If your accessory is connected to local network or connects to the internet, great shape.

Will serve you for what to do next.  Session between two iphones, 

## Configuring for accessory interaction

`NINearbyAccessoryConfiguration`

U1-compatible Ultra-wideband hardware will know how to generate this configuration data upon request.  This means that code you're running on the accessory itself will need to generate this data and then send over the data channel.

Save accessory token.  Use token and then ame to correlate updates back to this accessory and display relevant, rich UI.

Need an NISession instance, set its delegate.  Call `.run()` using configuration object we created.  Just like nearby interaction, native configuration data, the accessory also needs configuration data from the nearby interaction.

Data needs to be in a format called shareable configuration data.  When you run a session, nearby interaction will provide the shareable configuration data to your app via delegate.

Use datachannel to send this to the accessory.

`didGenerateShareableConfigurationData`.  useful in case you're interacting with multiple accessories.  Plan to send data ASAP.  

Generally, managing data connectsion to different accessories can take on many differnet forms depending on your usecase.

Optimize for sending to the accessory as soon as possible.  If not sent quickly, your session may timeout.  A timeout in a session will be communicated to your app through didRemove delegate callback.

If reason is `timeout` and I have high confidence the accessor is nearby, i can retry interacting with it.  

All I have to do is run again with the same configuration.  Keep in mind that the cached configuration will remain valid as long as session was not terminated.  If session was terminated, I'll have to repeat this flow.

Code on the accessory has to manage.  Can be done in different ways, depending on usecase.

U1 compatible hardware will know what tod o with the shareable configuration data.  So provide it, as-is, ASAP, to the U1 hardware onboard.

How will hardware or accessory know to generate data?  Defined in a specification document earlier this spring.  Document is intended for chipset and module manufacturers, contains details for ultrawideband solutions.

On top of interop spec, we are releasing a spec for accessory manufacturers.  So this document is for you.

Let's see what happens once code on accessory receives and provides to ultrawide hardware?

Shareable configuration data, ultrawideband hardware will start running.  If both accessory and iphone running are nearby, session will start providing stream of `NINearbyObject` updates.

Can interact with multiple objects by creating a session for each one.

## handling nearby object updates
Updates are delivered to session delegate through `didUpdate` callback.

Smooth distance (over time) to guard against rapid changes.  If the user makes sudden movements or standing on boundary between zones.

[[Design for spatial interaction]]


# Resources for getting started
# Wrap up
* New permission model
* Nearby Interaction with accessories
* Development kits and sample app code
* Specification documents

https://developer.apple.com/nearby-interaction/
https://mfi.apple.com
https://developer.apple.com/documentation/nearbyinteraction
