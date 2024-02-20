Discover best practices when creating 3D content for Quick Look on visionOS. We'll explore a few different ways to prepare your models for Quick Look, cover important considerations for 3D quality and performance, and show you how to use Reality Composer Pro and Reality Trace to inspect and fine-tune your content.

What is QuickLook?

[[Discover Quick Look for spatial computing]]

Before we dive deeper, let's talk a bit more about how the system presents 3d content in quicklook.

* presented in volume window
* People reposition volumes in space
* View from different angles
* Placed at the center of volume
* Initial orientation directly facing towards you
* Window bar for reposition
* Volume size based on the size of 3d model
* Important: Keep all animations inside of the volume!

# Model size
* use `metersPerUnit` from USDZ file to determine scale
* Initially shown at 100% scale within a certain size range
* QL imposes a minimum scale to ensrue that you can enjoy all content
* also has an upper bound limit in case you are viewing very large object

100% scale affordance to view model's authored size by tapping on button below the volume window.

QL automatically shows ground plane and shadow below your model to vgive you a better udnerstanding of model size and its position relative to the ground.  We suggest that you do not add your own ground plane!!



# Model creation
Accepts usdz as its primary format to handle 3d models.  Usdz is at the heart of 3d content for apple platforms.  Designed to be lightweight and optimized for shading.

How you can create usdz files?
* all usdzs from iOS
* digital content creation tools
* Scan 3d models of your real-world objects
* Room Plan

Remember how i mentioned there is a backstory to 3d room model from earlier?  Let's see how we made this.

He started with RoomPlan sampel application.  After a short time, eh can preview the 3D layout of his room, export/drop a USDZ file right to mac.  Import into Blender.

Reality Composer Pro.

# Visual quality

We can preview the model in QL right on device in RC pro.  Important to ensure that most interesting part of 3d model is presented facing towards you to create an engaging experience.

We have much better view of model.  Let's explore this asset even further.  New perspectives, details, etc.  Use a pinch and drag gesture to rotate the 3d model and view it from different sides.  Gives me a good view of different parts of our room.

Considerations for enhancing video quality.  When multiple objects are rendered in same location, they can overlap and appear as a single flickering object

* avoid overlapping geometry
* avoid high-frequency normal maps
* Use opacity texture for fine details and render with larger triangle grid
[[Explore rendering for spatial computing]]

# Performance
Many factors that determine 3d performance.  File size, texture count, materials, etc.

First identify potential limitations.  To simplify this task, we introduced some helpful new tools which can significantly improve your workflow.

RC pro has many details about geometry complexity, etc.

RealityKit Trace (instruments) has more info.  Get more information into the individual rendering frames within your 3d content.  Attach to QuickLook process.

[[Meet Reality Composer Pro]]
[[Meet RealityKit Trace]]

## Best practices

Optimize models for filesize.  
Balance asset quality with filesize.
ex, use less detailed textures or lower-quality audio sources.
Only include assets used in final content
Less than 25 MB for better sharing experience

Optimizing 3d models for geometry
Remove unnecessary vertices or meshes
Merge small meshes into a single larger one
Recommend less than 200 mesh parts, less than 100k vertices in total

Optimizing 3d models for texture.
Use grayscale for non-color inputs
Multiple grayscale maps into a single texture
Use constant values instead of texture where possible
Recommend max 2048 by 2048 texture size
8-bit per channel textures
Spend your texture budget with areas that add the most value and realism

Optimizing 3d materials
Combine mesh parts to share with same material
System has to compile custom materials when loading
Balance material complexity vs screen size.  Ex if you need a small part of the screen transparent, it may be more efficient to use a separate all-transparent material vs having a complex material with multiple cases
Be cautious of overlapping transparency.  Rendering transparent objects in realtime is more complex.  Only have overlapping transparency when needed
Use materialX unlit surface to save real-time lighting computation

Optimizing 3d models for physics
Reduce total collider count
Use static over dynamic where possible

Optimizing 3d models for animation
Limit skinned animations and number of weights per vertex
Deformations / skinned animations should follow geometry guidelines

Optimizing 3d models for particles
Limit particle emitters and particles per emitter
Experiment with shapes and animation styles of particle effects for reducing overdraw

# Summary
* consider different scenarios to test your QuickLook 3d models
* Leverage performance profiling tools
* Identify sweet spot between visual quality and 3d performance

[[Discover Quick Look for spatial computing]]



