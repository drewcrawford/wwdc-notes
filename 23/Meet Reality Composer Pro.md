Discover how to easily compose, edit, and preview 3D content with Reality Composer Pro. Follow along as we explore this developer tool by setting up a new project, composing scenes, adding particle emitters and audio, and even previewing content on device. Once you're familiar with the basics of Reality Composer Pro, check out "Explore Materials in Reality Composer Pro" and "Work with Reality Composer Pro content in Xcode" to learn more advanced techniques and tips.

using RC pro, create immersive 3d experiences.  Check out fantastic national parks, points of interest, etc.

We'll go through various features that we can leverage to build a scene.

# Project setup
can either open developer tool, or create xrOS app which creates its project automatically.  Recommended if you want to use a RC pro project in an xcode app.
swift package.  Swift packages not only make it easier to work with RC pro, but package into framework for xcode app to use.

[[Understand USD fundamentals]]
[[Create 3D workflows with USD]]

any usds in our rkassets will be compiled.  In xcode, if we select a package, a 3d preview appears.  Open in RC pro in upper right corner.
# UI navigation
center - viewport.  wasd, arrow keys.  plug in a mac-compatible controller and fly thruogh that way.
leading side - hierarchy panel.  Search, select, and organize the 3d objects in the scene.

trailing side - inspector panel.  Provides an easy way to edit properties, position, scale, etc.
add component - lets us view a builtin RK components on our objects.  
Editor panel - project browser.  Navigate through files.
shader graph, audio mixer, statistics.

# Composing scenes
3 ways to add assets to project.

1.  Import assets into project browser that are on your computer.  import button in project browser.
2. content library - curated library of assets
3. object capture.  To learn more, check out meet object capture for iOS.

location pin demo.


# Particle emitters
create FX like this flame.  Two parts, the particle and the emitter.
particle - color, properties, forcefields, rendering
emitter - timing, shape, spawning

cloudchunks -> clouds

scenes are shown as tabs.  Plus button in hierarchy can add emitter.  Or we can add an emitter to any object in the scene with 'add component' at the bottom of inspector.  

particles are fun to play with.  get the reuslts I look for by experimenting.  Get them to a state that pleases you.  High count of particles may have performance implications, pay close attention to the number of particles being used.

demos


# Audio authoring
played on one or more objects, etc.  

an audio file group can be constructed from audio files in a scene.  Each time we play the group, a random file in the group si played.

audio source - defines how these are emitted into scenes.

| source                     | position | direction |
| -------------------------- | -------- | --------- |
| spatial                    | yes      | yes       |
| ambient (e.g. always east) | no       | yes       |
| channel (bg music)                    | no       | no          |

demos

[[Work with Reality Composer Pro content in Xcode]] - code needed to play audio
# Statistics
general
physics
animation
particle mitters
audio materials
geometry
textures

ex, look at number of triangles for example.

# On-device preview

device in upper-right.  We see the scene apepar before our eyes in AR.  Pinch and drag to rotate.  Zoom to scale.  Easy to get a sense of what our content looks like at intended platform.  Similar steps of what we went through for another park, etc.

Using the same workflow, I added an additional 3d terrain for main scene.  Added locations for harbors, beach, etc.  Ocean sounds audiofile.  

* assemble 3D scenes
* add realitykit components
* optimize and preview

[[Explore materials in Reality Composer Pro]]
[[Work with Reality Composer Pro content in Xcode]]


# Resources
* https://developer.apple.com/documentation/visionOS/diorama
