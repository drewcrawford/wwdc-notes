Go beyond detecting cats and dogs in images. We'll show you how to use Vision to detect the individual joints and poses of these animals as well — all in real time — and share how you can enable exciting features like animal tracking for a camera app, creative embellishment on an animal photo, and more. We'll also explore other important enhancements to Vision and share best practices. To learn even more about what's new in the Vision framework, watch "Explore 3D body pose and person segmentation in Vision” and "Lift subjects from images in your app.” And to learn more about building live camera-tracking experiences, check out "Integrate with motorized iPhone stands using DockKit"

# Animal body pose
Imagine you left your cat/dog at home alone, and you spent the day at work.  When you are back, you find this mess in your house!  what were your pets doing all day?

`VNDetectHumanBodyPoseRequest`.  19 joints for full body.  Since vision interacts with the real world, we don't only carea bout humans.

Vision has `VNRecognizeAnimalsRequest` for cats/dogs.  The request generates a bounding box with labels for cat/dog and confidence level.

what about if you want to know more about the animal?  What is it doing?

new API in vision.  `VNDetectAnimalBodyPoseRequest`.
* collection of animal body joints
* cats and dogs
	* 25 animal body landmarks including tail and ears
* available in vision starting in iOS 17, ipados 17, tvoS 17, macOS 14

input can be image or video.  After creating a request in vision, the request will produce a collection of joints to define the skeleton of the animal.  For animal body pose, 6 joint groups have been defined.

* head -> ears, eyes, nose
* forelegs -> front legs, 
* hind legs -> hind legs
* trunk -> neck
* tail -> tail - 3 joints
* all -> all joints

1.  create request
2. create request handler `VNImageRequestHandler`
3. provide request to handler via `perform`
4. observations will be returned in request results
5. location of joints, etc.

you can use another group if you need to only access some of the animal joints.  To draw the skeleton of the animal, iterate over all the recognized points and connect the joints.

## Considerations
* can detect up to two animals in an image
* image size at least 64px
* performance
	* neural engine can keep up with live capture

can develop youer own models to recognize poses
* stretching
* Standing
* running
* curling up

VNRecognizeAnimalsRequest
VNDetectAnimalBodyuPose -> full body length mask of the animal.
tells you waht kind of animal was detected, location, and pose

 pose API can be used with videos.  Bring your own algorithms to analyze motion and determine what type of activity your animal is doing.  Even go further to understand animal behavior by tracking poses over time.

camera tracking apps - to learn more, see [[Integrate with motorized iPhone stands using DockKit]]

can also write funny apps for your pets.  ex, putting a hat and sunglasses on a dog.

let's demonstrate the emoji app using this cute dog.  I will use the same sample app and switch from skeleton view to emoji view.

since this little dog is walking slowly, I added some skates emojis on top of his ? joints.


# Other updates
more updates in vision that you may find helpful.

New stateful requests.  VNTargetedImageRequest -> now stateful request.  Vision has 3 new derived stateful requests all named with the Track verb.  Makes it easier to use for tracking.

Compute Device.  `MLComputeDevice`.  Alows you to query where requests get executed and specify which device to use.  

CoreML/CreateML now have multilabel classification compatible with vision.  Train classifiers with 1+ labels.

[[Discover machine learning enhancements in Create ML]]

Barcodes -> new revision 4.  Includes the new MSIPlessey, color inverted QR codes.  v1 is deprecated.

Text recognition -> thai, vietnamese.

New revision for `FaceCaptureQuality`, 3.  Improve quality/accuracy.  For additional information about all tohse new updates, see developer docs.

# Wrap up
* animal pose is fun and easy
* lots of great features in vision
* Explore 3D body pose and person segmentation in Vision

[[Explore 3D body pose and person segmentation in Vision]]
[[Lift subjects from images in your app]]

# references
* https://developer.apple.com/documentation/vision
