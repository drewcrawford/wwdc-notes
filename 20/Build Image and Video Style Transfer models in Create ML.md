#createml 
Blend a style and content image together
...real-time!

120fps -> A13 bionic

# Model parameters
* style strength -> how seriously we take the style image
* style density -> Basically, the grid is divided into regions.  More density produces smaller regions.  The model tries to make a region of image look like the style.

# Demo
# Recap
* training checkpoints
* snapshots
* ?

[[Control training in Create ML with Swift]]

# Style Transfer with ARKit 4
ARKit -> CVPixelBuffer -> Style transfer model -> Stylized result -> Metal

Can add a 3D object by adding an ARAnchor.

Use person segmentation and run each channel through a diffrent style transfer model.
