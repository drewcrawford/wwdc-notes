Find out how you can develop great gaming experiences for visionOS. We'll share some of the key building blocks that help you create games for this platform, explore how your experiences can fluidly move between levels of immersion, and provide a roadmap for exploring ARKit, RealityKit, Reality Composer Pro, Unity, Metal, and Compositor.

#xros

# Types of spatial games

Standalone spatial computing device
high reoslution, high VOB, high refresh rate
spatial audio
world mapping and hand tracking.

spectrum of immersion

* shared space -> your game can live in the space with other games and apps.  There might be a virtual chessboard on the player's desktop, etc.  Apps live together and the player can interact with whichever one they want.
Full space -> closes all other windows and volumes, focuses on your content, etc.  Might be suitable for an action game, something you're actively palying, but it still interacts with the real world.

fully immersive -> game takes over the whole view.  Instead of your room, there's an environment, adn you can no longer see the real world.

2D games
* run in a virtual window
* put your window anywhere
* Look at and tap to select
* bluetooth controller

2d/3d
Render objects in separate layers, and get a real parallax effect.
Volumetric effects
Hand gesture support

Design your game type
* arcade
* casual
* tabletop
* interactive
* transformative


# Rendering, audio, and input

work differently on this device.  

shared space:
Content is drawn together with content from other apps, system UI, and passthrough.  Since rendering is shared, framework ensures all apps are good citizens, doesn't interfere.

Can use surface shaders and geometry shaders through material X.

describe the scene instead of rendering it.

dynamic foveation - higher resolution where the player is looking
automatically applied by the device

sampled real-world lighting
physically based shaders

custom materials
custom shaders through materialX
edit the shader graph in reality composer pro, or other grpahic packages.
Custom IBLs

System effects
The purpose of this is to make your game work better with other apps and protect the player.
* depth mitigation
* near-field vignetting - fades content when the player gets too close.  
	* prevents issues with your content clipping against the near plane
* breakthrough
* grounding shadows - when placed near real-world objects, to make them feel more integrated.

fully-immersive rendering works similar to shared rendering.  In this mode, you render an environment; there is no passthrough.

You render an environment.

Rendering in this mode works simlar to the shared mode.  Since you control everything the player sees, you have more freedom.

* full control over lighting.

compositor rendering.
* metal rendering on xrOS
* Custom Metal shaders and post processors
* Virtual content hides passthrough.

[[Discover Metal for immersive apps]]

## spatial audio
Bring objects to life in the player's space.  matching reverb, etc.

If you bring audio with standard iOS APIs, the audio will be positioned relative to the app window.

If you want sound to come from different objects, play through realitykit.  Specific entities in your scene.

Also have the option to build your own audio, using any apple or external API.  If you want it to be headtracked, need to use ARKit to get the player's head position.

## types of input
* system gestures
* game controllers
* hand tracking
* scene understanding.

Look & tap, drag, magnify
based on collision shapes

Requires CollisionComponent and InputTargetComponent.

game controllers
* look & tap has a maximum of two simultaneous inputs
* bluetooth game controllers can be used when you need more.  trackpads, mice, keyboards, etc.

hand tracking
can be used to implement custom gestures
requires permission
hands are tracked when visible to the camera
fast movements are harder to track

scene understanding (ARKit)
creates a virtual mesh representing the room
requires permission

direct manipulation can enhance the experience
but phyiscal movement can get tiring
look & tap can be more comfortable
game controllers are another option

multiplayer
supports all multiplayer and networking capabilities
web-based, socket-based, multi-peer, etc.
socialized play through shareplay
cross-platform networking



# Development frameworks

various options
compatible games, native 2d, metal, unity, realitykit, etc.


if you have already made an iPhone/iPad game, it's probably compatible.  
if you want your game to be stereoscopic, you need the APIS.

To take full advantage of the API, need to do that.

spritekit, realitykit, swiftui, etc.

go the seession

[[Bring your Unity VR app to a fully immersive space]]
[[Create immersive Unity apps]]

realitykit.


# Build with RealityKit
ECS, extensibility, etc.

On the rendering side, RK supports both USD models as well as custom meshes.  
also materialX and IBL-lighting.

attachments - connect rich swiftui directly to realitykit objects.

realityview and the swiftui hierarchy holds the 3d rendering.

by default, your 3d content is in a volumetric box.  Confined to the window, etc.
realitykit is a swift API.  Great language for building games, but you can also use another language.

breaking out of the box
* windows
* volumes
* spaces - all around the player in the real world
	* shared - run together with other apps
	* full space - immersive
* anchors - surface, image, body
* portals

preparing content with reality composer pro
* load and render USD files with realitykit
* prepare usd files with reality composer pro
* convert to USD for use with realitykit

[[Meet Reality Composer Pro]]
[[Explore materials in Reality Composer Pro]]

# Deeper dive
* other sessions to explore

[[Build spatial experiences with RealityKit]]
[[Enhance your xrOS app with RealityKit]]
[[Explore the USD ecosystem]]
[[Work with Reality Composer Pro content in Xcode]]
[[Bring your ARZKit app to xrOS]]

