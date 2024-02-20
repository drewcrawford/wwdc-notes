Explore how you can use Unity to create engaging and immersive experiences for visionOS. We'll share how Unity integrates seamlessly with Apple frameworks, take you through the tools you can use to build natively for the platform, and show you how volume cameras can bring your existing scenes into visionOS windows, volumes, and spaces. Discover how to incorporate visionOS features like passthrough and scene understanding, customize your visuals with Shader Graph, and adapt your interactions to work with spatial input.

Used by 10s of thousansd of apps.  Use unity to build immersive apps.

Two main approaches.

Immersive (passthrough)
Fully immersive

Great tool for creating immersive experiences with familiar tools.

Demo.

# Achieve your visual look

Rendered using realitykit.  Your materials and shaders need to be translated to this new environment.

Unity PolySpatial.  
* materials
* mesh renderers
* particle effects
* sprites
* simulation features
* monobehaviors, scriptable objects, standard tools
## materials
* PBR 
* custom materials
* (some) effect materials

PBR - universal render pipeline, can use Lit, Simple Lit, or Complex Lit.  Built-in pipeline, you can use standard shader.  All these are translared to RK PBR.

Custom material types are supported through UnityShaderGraph.  Converted to MaterialX, a standard interchange format.  MaterialX becomes ShaderGraphMaterial.  'many' nodes are supported.

Handwritten shaders are not supported, but you can use them through rendertextures in unity.  Can use that as a texture input to shader graph.

unlit shader.  Create objects that take on a solid texture unaffected.

occlusion - lets passthrough show through the object.  Use with world mesh data to help coontent feel more integrated.

## meshes/models

* meshrenderer
* skinnedmeshrenderer
pipelines
* universal render pipeline
* built-in render pipleline

rendering features such as post processing FX or custom pipeline stages are not available RK performs final rendering.

Particles and sprites
* simple: translated to realitykit
* complex: translated to baked meshes
sprites are translated to 3d meshes.

## simulation features
* physics
* aniamtion/timeline
* pathfinding
* monobehaviors
* other non-rendering features


# Play to device
available for the first time.  Lets you see an instant preview of your scene, and make live changes.
work swith simulator, works on device.

Rapid iteration.
* contetn placement
* materials and textures
* shader graphs
* interaction
* debug via editor
* 

# Explore volume cameras

control how you're brought into the real world.  bounded vs unbounded.

your application can switch between the two at any time.  Bounded volumes exist in the shared space as volumes.  Dimensions, transform in unity, as well as a specific real-world size.

Respositioned, but cannot be resized by user.

Dimensions and transform define the region of your scene that app displays in a volume.  Specified in scene units.

By manipulating dimensions/transform of volume camera, different parts of scene can be brought into the volume.  scaling the camera causes more/less items to come into view, but the volume remains the same size as perceived in VR by the user.

Content is clipped when it intersects the volume.  Consider placing the same mesh a second time with back-facing material to fill in clipped sections.

Unbounded volume displays in a full space.
No dimensions
Unity units are mapped to real-world units
One active at a time


# Build interaction

Input types
8look / tap
hands
head pose
augmented reality
bluetooth devices

## look/tap
* requires input colliders
* look/tap from a distance
* direct touch
* up to two simultaneous tap actions can be in progress
* delivered as WorldTouch events

requires input and hands packages.  unbounded volumes only.  Hands require permission

Augmented reality
* plane detection
* world mesh
* image markers
* unbounded volumes only
* requires extra permission

bluetooth devices
* keyboards
* controllers
* other system-supported devices
As some inputs are only availabel in unbounded volumes, decide what to build

| x                 | bounded volumes | unbounced volumes | accessed via                    |
| ----------------- | --------------- | ----------------- | ------------------------------- |
| look/tap          | yes             | yes               | input system, touchspace events |
| head pose         | no              | yes               | input system, head pose events  |
| hands             | no              | yes (permission)  | hands package                   |
| AR data           | no              | yes (permission)  | AR foundation package           |
| bluetooth devices | yes             | yes               | input system                                |

demo

Adapt existing interactions.

If you're already working with touch, ex iphone, add appropriate input colliders and continue to use tap

If using controllers, redefeine interactions for either tap/hand based input depending on how complex they are.

existing hand-based input should work without changes

existing UI panels, can bring UGUI or UI toolkit to this platform.  

Other UI systems work as long as they use mesh and renderer or draw to RenderTexture.
# Prepare for the platform

Unity 2022.Upgrade now.

Convert shaders to Shader Graph.
Adopt the universal render pipeline.
Move to the Input System package
Think about volumes

# Takeaways
* unity and realitykit work great together
* you can get ready today
[[Bring your Unity VR app to a fully immersive space]]
[[Build great games for spatial computing]]

# Resources
* https://create.unity.com/spatial
