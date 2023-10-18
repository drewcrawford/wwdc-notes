Get ready to develop apps and games for visionOS! Discover the fundamental building blocks that make up spatial computing — windows, volumes, and spaces — and find out how you can use these elements to build engaging and immersive experiences.

Join me in guiding you how to get started building apps in spatial computing.

* building on familiar foundations
* Blending real and virtual
* Natural input
* Privacy-first design

# Fundamentals
By default, apps launch into the shared space.  Where apps exist die by side, much like multiple apps on the ?

Each app can have one or more windows.  These are SwiftUI scenes that can be resized and reflowed like macOS.  Traditional views and control,s as well as 3d content.  Mix/match 2d and 3d.  People can reposition the window to their liking.

Volumes allow an app to display 3d content in a defined bounds, sharing the space with other apps.  Showcasing 3d content, ex chess board.  People can reposition volumes in space and viewed from different angles.  SwiftUI scenes, allowing you to do layout in familiar ways.  Use the power of RealityKit.

Sometimes you might want to have more control over the level of immersion.  Opening a dedicated full space, where your app’s volume, windows, 3d objects, etc., only one appearing across the view.

ARKit’s apis ex you can get more detailed hand tracking.  

Your app can use the full space in different ways.  Pass through.  Keep people connected with their surroundings.  

Choose to render to fully immersive space to fill the entire view.  Flexibility to deliver on creative intent.  By customizing the lighting of virtual objects as well as audio characteristics.

Foundational elements of spatial computing.  

## interactions
* eyes and hand input
* Direct input

Taps, long presses, drags
Rotation, zooms, much more.
Interactions with RealityKit entities
Stack cubes on a table using taps, smashing them, etc.

Powerful way to bring app-specific hands input into your experience.

Skeletal hand tracking.
Wireless devices

Share any window.  When people share a quick look model, we sync orientation, scale, and animations between participants.

Important that everyone in the session have the same experience.  Natural references such as gesturing to an object.  Reinforces the feeling of being physically together.

Shared context.  System manages the shared context, ensuring participants experience in the same way.

[[Design spatial SharePlay experiences]]
[[Build spatial SharePlay experiences]]

Given that the device has much knowledge, we put a lot of architecture to protect people’s privacy.

## privacy
* privacy-first design
* Curated data and interactions
	* Ex, system knows eye position and gestures and delivers as touch events
	* Hover effect, but does not tell the app where you’re looking.

For many situations the system-provided behaviors are fine.  System will ask people for more sensitive data.  Ex, skeletal hand tracking, scene understanding, etc.

How we are developing those apps.  Starts with Xcode.  Apple’s integrated development environment.

SwiftUI previews.  Extended to support 3d.  Enable shorter iteration times, right look and feel, edit live code, etc.  How the satellite looks orbiting the earth.

3 different simulated scenes, each with day and night lighting.  See your app under different conditions.  Run, debug most apps.  Iterate with a predictable environment.

A number of runtime visualizations.  Here we have a plane estimation.  Semantic meaning, collision shapes, etc.  Easy to toggle visualizations you would like to focus on from the debugger in Xcode.  Great both in simulator and in the device.

When it becomes time to test your apps, we have things like instruments.

For spatial computing, instruments 15 includes a new template, RealityKit trace.  New instruments allowing developers to understand GPU, CPU, and system power impact.  Easily observe and understand frame bottlenecks.  Total triangles submitted, etc.


For more details, check out [[Meet RealityKit Trace]]

Reality Composer Pro
* preview
* Prepare

Added particles this year.  Author and preview in RCP.  Movement, life, endless possibilities.  Clouds, rain, sparks, etc.

Adding audio into your scenes is a breeze.  Also spatially preview audio, taking into account shape and content of your scene.  Most virtual objects will use reality-kit’s physically-based materials.

Extend via open standard material X.  Node graph.

[[Explore materials in Reality Composer Pro]]

Send your scenes to your device.  Test your content directly.  Great for iteration times.

See [[Meet Reality Composer Pro]]

Another option is unity.  Write apps for spatial computing without any plugins required.  Power new immersive experiences.

[[Bring your Unity VR app to a fully immersive space]]
[[Create immersive Unity apps]]



# Where to start
Two ways to start.
* brand new app
* Existing app

New template in Xcode.
1.  Choose your initial scene type.  Window or volume.
2. Immersive space entry point.  By default, we launch to shared space.  If you set this to space, a second scene will be added along with an example button showing how to launch into the full space.

[[Develop your first immersive app]]

## Code samples.
* destination video - shared immersive playback experience
* Happy beam - game that leverages an immersive space including custom hand gesture
* World - transition between different visual modes with 3d globe.

## existing app

If your app supports iPad, that variant will be preferred.  But iPhone-only apps are fully supported.

Windows scale, rotations, etc.

[[Run your iPad and iPhone apps in the Shared Space]]

Running an existing iPad, iPhone app is just the beginning.  Add a destination in your Xcode project with just a few clicks.  After that, simply select a target device, recompile, and run.
# How to build

* windows
* Volumes
* Spaces

Spectrum of flexing up and down depending on what’s best for your app in a specific moment.

Between more presence and deeper immersion.

Important consideration when you design your app for spatial computing.  Let’s look further into how to use windows as part of your experience.

## Windows
* available with swiftui
* 2D and 3D content
* Scale and position

New Model3D view.  Displays 3D content with RealityKit.

Now this window has the satellite embedded, etc.  New depth dimension.  

SwiftUI provides gesture recognizers youre’ familiar with, such as tap, hover, rotat,e etc.  Platform provides new GRs for 3d interaction, like 3d rotations, taps, more.

Add a gesture, targeted to the satellite entity.  Use the values passed in from t he update closure to move the satellite.

Mix 2D/3D content together.  Just a few things you can do with a window.

## Volumes
* extension of a window
* Ideal for 3D content
* Can host multiple views
* Built for the shared space
* Content must remain inside the volume!

Window style - volumetric
Default size.  Units can be specified in points or meters.

Application title bar, makes it easier to identify what app we belong to.  Our volume renders a 3d model of the earth.  More content, different behaviors.

Adopt reality view.

* swiftUI view with entities
* RealityKit and SwiftUI deeply integrated
* Coordinate space conversion
* SwiftUI attachments

Initializer takes 3 parameters.
* main closure
* Update closure - state of the view changes
* Attachments view builder - add our SwiftUI views.  Translates our views into entities.

Add the attachment to content of our reality view.  Here we can see the attachment we made previously, rendering above my favorite bakery location.

Powerful features such as model3d, reality view, attachments, more.

[[Build spatial experiences with RealityKit]]
[[enhance your spatial computing app with RealityKit]]

## spaces
* focus is on your app, we hide all other apps
* Place app content anywhere
* Interact with your surroundings
* Build custom hand interactions

[[Meet ARKit for spatial computing]]
* different levels of immersion
	* Passthrough (`mixed`) Default.
	* fully Immersive (`full`)
	* `.progressive` - allows some passthrough initially, but the person can change the level of immersion by turning the Digital Crown.  

# Next steps

[[Principles of spatial design]]
[[Meet SwiftuI for spatial computing]]
[[Build spatial experiences with RealityKit]]
[[Meet Reality Composer Pro]]



