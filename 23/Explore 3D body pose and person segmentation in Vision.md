Discover how to build person-centric features with Vision. Learn how to detect human body poses and measure individual joint locations in 3D space. We'll also show you how to take advantage of person segmentation APIs to distinguish and segment up to four individuals in an image. To learn more about the latest features in Vision, check out “Detect animal poses in Vision” from WWDC23.

# Human body pose

2d body pose part of vision since 2020
landmark points define 2d skeleton

[[Detect Body and Hand Pose with Vision]]

Expanding to 3d with VNDetectHumanBodyPose3DRequest
17 joints by name or group

position in meters relative to scene
relative to root joint
initially we support 1 skeleton for the most prominent person

heightEstimation - either an estimate or a reference height of 1.8 meters.

cameraOriginMatrix - helps understand where the camera is.
pointInImage -> project points back to image.
cameraRelativePosition -> helps work across iamges.

new Geometry 3D classes in Vision

Like ARKit, vision represents 3d position as a 4x4 matrix.

VNPoitn3d -> VNRecongizedPoint3D -> ...

local position -> relative to parent joint.

You often need to determine the angle between child and parent joint.  There's a function for this given here.

# Depth in Vision
New AOPI on VNImageRequestHandler to take in depth.

Existing API will fetch Depth from URL.  

AVDepthData
Disparity vs depth map
* disparityfloat16, 32
* depthfloat16,32

camera calibration
[[Discover advancements in iOS camera capture Depth, focus, and multitasking]]

available in capture session and from file
portrait images captured by camera app store depth as disparity
LiDAR enables high-accuracy measurement of scene


# Person instance mask

VNGeneratePersonSegmentationREquest
* single mask for all people in the frame

VNGeneratePersonInstanceMaskREquest
* up to 4 fg people (individual masks)

[[Lift subjects from images in your app]]

considerations with crowded scenes.  You may miss people in the background, or merge people in close contact.

images with crowds.  Use face-detection API to filter four or more people
Use person segmentation request.

# Wrap up
* expansion into 3d and depth
* interacting with multiple people

[[Detect animal poses in Vision]]


# Resources
* https://developer.apple.com/documentation/vision
