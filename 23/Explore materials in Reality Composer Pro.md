Learn how Reality Composer Pro can help you alter the appearance of your 3D objects using RealityKit materials. We'll introduce you to MaterialX and physically-based (PBR) shaders, show you how to design dynamic materials using the shader graph editor, and explore adding custom inputs to a material so that you can control it in your visionOS app. To get the most out of this session, we recommend first watching “Meet Reality Composer Pro.” Once you're ready to learn how you can incorporate your models and materials into an Xcode project, check out "Work with Reality Composer Pro content in Xcode."

# Materials in xrOS

Can modify geometry, etc.  We use PBR.  

CustomMaterial -> hand coded in metal.

ShaderGraphmaterial -> xrOS.  exclusive way to create custom materials for xrOS.

based on materialX, initially created by ILM.

Physically based, custom.

PBR -> simpler use cases.  Constant nonchanging values.
Custom shaders -> precise and custom control over the appearance of 3d objects.  Incorporate animation, adjust geometry, create special FX, etc.

# Material editing
[[Meet Reality Composer Pro]]

Demo.

Assign material to the model.

position node -> outputs place where material is rendered.

position->separate->modulo.

drag new connection to empty space to create connected node in 1 step.  

ifGreater -> topography vs line color.


# Node graphs

Help simplify complex materials and create your own nodes to reuse parts of graphs.  Give a material ar eal topographical map look.

Basically a combinator.

Instances are live copies that adopt any changes made to original node.

# Geometry modifiers

Surface shaders operate on PBR attributes.
Geometry modifiers operate on the geometry.
Both surface and geometry shaders can be built on the shader graph.

Use a geometry modifier to raise terrain by correct amount.  

# Wrap up
* overview of materials in xrOS
* Design materials using shader graph
* reusability with node graphs
* Geometry modifiers for mesh modifications

https://developer.apple.com/documentation/visionOS/diorama

[[Work with Reality Composer Pro content in Xcode]]
[[Meet Reality Composer Pro]]
