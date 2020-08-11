Get it?  The ARtist's AR toolkit 

🤓

#usd 

1.  DCC
2.  reality converter
3.  reality composer
4.  on device




# Exporting content from DCC
Geometry
* fbx
* obj
* usd
* gltf

textures
* png
* jpg

reality coverter and usdz tools are available at developer.apple.com.

Export instances separately as opposed to merging them into geometry.
# Converting assets to USDZ
#realityconverter

Image-based lighting (IBL)
Default materials

# Back to DCC
Static assets are often in OBJ.
GLTF is often used for offline rendering, such as raycasting.

Off-the-shelf assets are available, often in usdz for mat.  ex, https://sketchfab.com.  
But when you find an asset online and not in usdz format, you now have the capability to make it yourself.

# Back to reality converter
Converter can accept drags of entire folders to avoid doing textures separately.  

Can specify in cm instead of m as the unit.

File->export all can batch export.

# importing into reality composer
#realitycomposer 
Drag usdz in and this should work.

In composer, we don't automatically animate (unlike converter), so you have to play manually.

Can edit the individual clouds in the usdz.  Need to right click and expand the group.


# Creating an experience
## Taxi sequence
1.  show, "move from front", ease in
2.  moveto endpoint
3.  hide, "move to rear", ease out
4.  moveto start point

## Police sequence

1.  show
2.  move
3.  orbit to do the turn.  
4.  move
5.  hide

## cloud sequence
Different behaviors that have slightly different rates targeting different objects.

# viewing on device
Can hit "edit on iOS" to drop to iPad.  then convert back.

While this is loading (? before it drops down on table), you view a scene.  Idea here is

1.  Create "fake" launch scene
2.  On "scene start", change scenes

This creates a splash screen for peopel to look at while your content is loading.

## exporting
Can now expert as a usdz in addition to the reality export.

Need to enable this in preferences for some reason.

If you're looking to embed in website, etc., you'll want usdz export.  If you want xcode/quicklook, etc., stick with reality export.

# Wrap up
* reality coverter
* reality composer
* AR quick look

With these tools you can create even more compelling content for your experience.

# next steps
samples in realitykit


