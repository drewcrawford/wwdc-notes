Discover how you can create incredible photo and video experiences in your camera app when integrating with DockKit-compatible motorized stands. We'll show how your app can automatically track subjects in live video across a 360-degree field of view, take direct control of the stand to customize framing, directly control the motors, and provide your own inference model for tracking other objects. Finally, we'll demonstrate how to create a sense of emotion through dynamic device animations. To learn more techniques for image tracking, check out “Detect animal poses in Vision” from WWDC23 and "Classify hand poses and actions with Create ML” from WWDC21.

# Introduction to DockKit

framework that allows the iPhone to act as central compute for motorized camera stands.  Extends FOV to 360pan, 90 tilt.  User to focus on content and without worrying about being in frame in any camera app.

May include power, disabling tracking, LED indicator to let you know if tracking is active, etc.  All magic happens in application / system services on iPhone itself.

Video capture, live streaming, conferencing, fitness, enterprises, etc.

Instead of just talking, let me demo this to you.

Dock responds by rotating 180 degrees.

With DockKit stands, you can interact with the space and objects around you.  Books, rearrange space, etc.

DockKit applications allow the storyteller to focus on the story without worrying about fov.  

# How it works

System tracker runs inside camera processing pipeline.  Decide which subject to track, etc.  Framing the subject appropriately by driving the motors.

Motor contorl is achieved through DockKit daemon, stand.  Actuation commands sent via DockKit protocol, and sensor feedback closes the loop on motor control.

Camera frames reach DockKit at 30fps.  Face/body bounding boxes are generated, fed to multi-level tracker.  Track for each person, EKF filter, etc.  

Motor positional and velocity feedback.  Phone IMU to arrive at trajectory/actuator commands.  By default, DockKit tracks primary object with center framing.  

# custom control

Getting reference to dock.

1.  Register for accessory state changes
2. Respond to notifications (dock/undock)

ex control cropping.

* framing mode (left/center/right)
* ROI

 `setFramingMode(.right)`

Here, all video frames intend to be made square.  Default framing could cause someone's face to be cut off.  Set region of interest.  Upper left pointer of display is origin.  ROI is defined in normalized coordinates.  Now we crop the subject to that place specified (like a mask I guess).

Set `setSystemTrackingEnabled(false)` before doing custom tracking.

x/y axis.  Tilt is around x axis.  Pan (yaw) is around Y axis, aligned with base of stand.

Custom inference.
Can also use the vision framework, custom ML model, etc., construct observations to feed into DockKit to track the object.

Here, origin is lower left?  Normalized coordinates.

You want to specify the observation type.
`.humanFace` ensures system multi-person tracking is still in effect.

`corrected` -> relative to bottom left corner of screen.

For simplicity, I"m focusing on thumbtip.  But i could use any finger joint or the whole hand to construct observation.

pass observation to dockkit to track.
# Device animations

Bring the device to life through animations.

* New affordance for confirmation
* convey emotion
* Emulate physical interaction

built-in animations
* yes
* no
* wakeup
* kapow

[[Classify hand poses and actions with Create ML]]

Consider creating your own animations with custom motor control

# Wrap up
* compute for motorized stands
* Customizable object tracking
* Mechanized motion as an affordance

[[Detect animal poses in Vision]]


# Resources
* https://developer.apple.com/documentation/CreateML
* https://developer.apple.com/documentation/DockKit
* https://developer.apple.com/documentation/vision
