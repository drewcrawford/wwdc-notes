
#nearbyinteraction 

Take advantage of capabilities of U1, apple's chip for ultrawideband.  Creating precise and spatialyl-aware interactions between nearby apple devices or accessoreis copmatible with U1 chip.

# Getting started
* iPhone to iPhone
* iPhone to Apple Watch
* Third-party accessories

[[Meet Nearby Interaction]]
[[Explore Nearby Interaction with third-party accessories]]

# Enhanced NI with ARKit
Leveraging device trajectory computed with ARKit.  Leverages same underlying technology that powers precision finding with airtag.

* guide user to a nearby object
* Increase distance and direction avilability
	* broadening UWB FOV
* Best for stationary devices

## Demo
Can indicate the target is behind the user.

## Enable camera assistance
No code available

Set `.isCameraAssistanceEnabled=true`.

All that's required.

Available for:
Two apple devices.
Apple device to third party UWB accessory.

## When enabled
1.  You are not repsonsible for creating the ARSession
2. This also runs the ARSession.  AS a result, you need NSCameraUsageDescription purpose string.
3. If you already have a session in your app, you need to share the session.  Can call `setARSEssion`.  In that case, an ARSession will not be created.
	4. Must call `run` with ARWorldTrackingConfirmation.
	5. `worldAlignment=.gravity`
	6. `isCollaborationEnabled=false`
	7. `userFaceTrackingEnabled=false`
	8. `initialWorldMap=nil`
	9. `sessionShouldAttemptRElocalization=false`
	10. Always inspect the error code in delegate
	11. If you don't configure the properties as required, the session will be invalidated.

When camera assistance is enabled, two new properties are available.
1.  `horizontalAngle` indicating azmithal direction.  May be nil.
2. `verticalDirectionEstimate` position relationship in vertical dimension.

Distance => m
Direction => 3d vector from your device to the object.

Horizontal angle is the angle within a local horizontal plane.  Vertical displacement between the device,s horizontal rotation, etc.  1D representation of the heading.  Phone projected onto the horizontal plane.

If direction cannot be resolved, horizontal angle might still be available.

Vertical direction estimate is a qualititative assessment of the vertical position.  Guide the user between floors.

unknown, same, above, below, aboveOrBelow (not on the same level)

## FOV
Direction information corresponds to a cone projecting from the rear of the device.  Trajectory with camera assistance allows distance, direction, horizontal angle, and vertical direction estimate to be available in more scenarios.  Effectively expanding the FOV.

## Placing objects
`worldTransform` Returns a world transform in ARKit's coordinte space that represents the given nearby object's position when available.

Can return nil if information is insufficient.  When this is important to your app experience, coach the user to take action to generate this transform.

## Delegate additions
`didUpdateAlgorithmConvergence`.  Understand why these are unavailable.  

Can provide the `localizedDescription` to the user.

When object is nil, convergence state applies to the session itself.

Reason can indicate there's insufficient total motion, horizontal/vertical sweep, and insufficient lighting.  Be mindful that multiple reasons may exist at the same time.  Guide the user sequentially based on which one is most important.

## Capabilities
We've now made `.deviceCapabilities()` more descriptive.  This returns a new NIDeviceCapability object.

Check `.supportsPreciseDistanceMeasurement` is the equivalent of `.isSupported` class variable.

Also check `suppertsCameraAssistance`.  Recommended you tailor your app's experience to device capabilities.

Be mindful to include distance-only experiences in order to best support apple watch.

# Background sessions
When your app transitions to the background, any running NISessions are suspended until the application returns to the foreground.  You needed to focus on hands-on user experiences.

Starting in iOS 16, we're going hands-free.  Playing music when you walk into the room, etc.  Other hands-free actions on an accessory.  Even when the user isn't actively using your app via accessory background sessions

[[Explore Nearby Interaction with third-party accessories]]

## Communicating with an accessory
Common to have communication channel between an accessory and your app use bluetooth LE.  When paired, you can enable NI to start and continue sessions in the background.  Let's study this.

[[What's new in Core Bluetooth - 17]]

Today you can do bluetooth stuff in the background.  Taking advantage of powerful background operations, to efficiently discover the accessory and run your app in the bg, your app can start an NISession with bluetooth LE accessory.

1.  Ensure that it is bluetooth LE paired
2. Connect
3. Accessory generates UWB data.  Tells your app and the internal NI service.
4. Provide that to the NINearbyAccessoryColnfiguration
5. Run your session with that ocnfiguration
6. Get shareable configurabtion with NISession delegate
7. send that shareable configuration to the accessory


Must impelment the new nearby interaction GAT service.  Single encrypetd characteristic called "accessory configuration data characteristic".  iOS uses this characteristic to verify the association between your bluetooth peer identifier and your NISession.  Your app cannot read from this characteristic directly

developer.apple.com/nearby-interaction

If your accessory supports multiple parallel sessions, create multiple instances of the configuration data.  That's what's necessary on the accessory.

## application
accessory must be LE paired.  Your app triggers the process.  Scan for the accessory, connect to it, and discover its services and characteristics.

Read one of your accessory's encrypted characteristics.  Only need to do this once.  Will prompt to accept pairing.

Also requirea  bluetooth connection to your accessory.  Must be able to form this connection even when bakcgrounded.

Do this even if the accessory is not within bluetooth range.  Then, implement CBCentralmanagerDelegate method to restore state when launched by CB

Handle when connection is established.

Create an NINearbyAccessoryConfiguration object with both the accessoryData and the bluetoothPerIdentiifer.

Run an NISession with that configuration and it will run while your app is backgrounded.

One more thing...
Background mode => Uses Nearby Interaction.
Can also use xcode capabilities editor to add this.  
Ensrue you have "Uses Bluetooth LE accessories" enabled to ensure you can connect to accessories in the bg.

NI session will continue to run and it will not be suspended.  So UWB are available on the accessory.  You must consume... these on the accessoory.  You will not receive runtime int he app, you will not receive delegate callbacks.

## Best practices
* Trigger LE pairing with accessory when intuitive to the user
* Process UWB measurements **on your accessory**.
* Manage battery usage by only sending data from your accessory to yyour app during a significant user interaction

# Third-party hardware
* U1-compatible development kits... out of beta
developer.apple.com/nearby-interaction

* Updated specification for accessory manufacturers

# Wrap up
* Enhancing NI with ARKit
* Accessory background sessions
* Third-party hardware support

* https://developer.apple.com/forums/tags/wwdc2022-10008
* https://developer.apple.com/forums/create/question?&tag1=337&tag2=369030
* https://developer.apple.com/documentation/nearbyinteraction/finding_stationary_objects_with_precision
* https://developer.apple.com/documentation/phase
* https://developer.apple.com/nearby-interaction/
* https://developer.apple.com/documentation/nearbyinteraction
* https://developer.apple.com/documentation/corebluetooth
