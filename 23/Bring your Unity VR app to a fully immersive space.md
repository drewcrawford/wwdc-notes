Discover how you can bring your existing Unity VR apps and games to visionOS. We'll explore workflows that can help you get started and show you how to build for eyes and hands in your apps and games with the Unity Input System. Learn about Unity's XR Interaction Toolkit, tips for foveated rendering, and best practices.

unity uses compositor services, with metal rendering.  Unity also takes advantage of arkit to recognize body position, hand tracking, etc.

Two main approaches to creating immersive experiences.
* fully immersive
* immersive (passthrough)
[[Create immersive Unity apps]]

rec room demo.

tools and technologies to bring your content to VR platform.

# Build and run workflow

* select the build target
* enable the XR plugin
* recompile native plugins
* build an xcode project
* test on xcode simulator or on device

# prepare your graphics

one choice every project makes is which rendering pipeline to use.
* universal render pipeline - ideal.  Static foveated rendering.
this is a technique that concentrates more pixel density in the center of each lens.  Less detail on the peripherals.  This results in a higher quality experience.  

when you use the universal rendering pipeline, this is applied throughout the pieline
works with all URP features, camera stacking, HDR, etc.
shader macros handle remapping

since rendering is now nonlinear space, shader macros to handle this
this enhances the visual experience

single-pass instanced rendering
now supports metal grpahics api, enabled by default.  Engine submits only one draw call for both eyes.  This reduces the cpu overhead of rendering scenes in stereo.  

depth compositing.  Make sure your app writes to the depth buffer.  System compositor uses the depth buffer for information.  Whenever the depth buffer is missing, system renders error pixel.

ex skybox.  Ensure you're drawing a depth buffer tehre.  Unisty shaders will work out of the box
check custom shaders.
# Input options

people will use hands, eyes, to interact with content.  A few ways to add interaction.

* abstract input with XR interaction toolkit.  
* map to system gestures
* access raw hand joint data

XR interaction toolkit:
high-level interaction system
author once for hands and controllers
supports 3d interactions
locomotion for comfortable navigation
visual feedback for input

interactables are objects in the scene that can receive input.  Define interactors that specify how people can interact.  manager ties these together.

1.  Decide which objects in the scene can be interacted with (interactable).  Types
	2. simple - receive interactions.  select enter, exited, etc.
	3. grab - follow the interactor around and inherit its velocity
	4. teleport - define areas or points for the player to teleport to
5. custom interactables

interactors are responsible for selecting or interacting.  Lit of interactables they could potentially hover over.  Several types
* direct - touch, etc.
* ray - from far away.  Highly configurable with curved/straight lines, visualizations, etc.  
* socket.  Shows the player that a certain area can accept an object.  Somewhere in the world.
* poke - similar to direct, but it includes direction filtering.  
* gaze - can make colliders larger so they're easier to select.

manager serves as middleman, facilitating exchange of interactions.  Handles changes in interaction state, etc.  Usually a single interaction manager is established to enable all interactors and all interactables.  Alternatively, multiple managers can be utilized, with their assortment of interactors/interactables.

specific sets of interactions, ex, different set of interactiosn per scene.

XR controller - make sense of input data you receive.  Takes input from hands/controller or pass to interactor.

not limited to just one controller.  Gives us the flexibilty to support hands/controllers independently.  Sample code shows you how to do this.

abstract input wtih XR interaction toolkit.
map to system gestures

use system gestures with unity input system.  

can access raw hand joint data.

i'm a little unclear about how much gaze info unity gets.  Some, I guess, but it might be tied to a gesture occurring at the same time?


### Translate raw joints into gameplay actions - 12:46
```swift
static bool IsIndexExtended(XRHand hand)
{
    if (!(hand.GetJoint(XRHandJointID.Wrist).TryGetPose(out var wristPose) &&
          hand.GetJoint(XRHandJointID.IndexTip).TryGetPose(out var tipPose) &&
          hand.GetJoint(XRHandJointID.IndexIntermediate).TryGetPose(out var intermediatePose)))
    {
        return false;
    }

    var wristToTip = tipPose.position - wristPose.position;
    var wristToIntermediate = intermediatePose.position - wristPose.position;
    return wristToTip.sqrMagnitude > wristToIntermediate.sqrMagnitude;
}
```

you can extend this logic to other fingers to get some basic gesture detections.

can also map hand data to a custom hand mesh visual.  ex rec room uses this to show hand gestures while you talk, etc.

unity.com/spatial

to get more information about unity's support, visit this URL.

# Wrap up
* upgrade to unity 2022 or later
* adopt the universal render pipeline
* prepare your input for hands

[[Create immersive Unity apps]]
[[Build great games for spatial computing]]
