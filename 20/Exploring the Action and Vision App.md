# Observing and giving feedback
Let your iPhone or iPad step in as a coach
Always with you
Use the camera and other sensors
Provide realtime feedback
You already have the perfect tool in your pocket.
Fast CPU, GPU, and Neural Engine
Comprehensive and cooperative APIs
All on device
* preserve user privacy
* no latency

# today in sports and fitness
Sport analysis can help everyone to improve
Billions of sports enthusiasts up to pro athletes

# Improve in sports and fitness
Common analysis is done by
* what did the body do?
* Which objects are in motion?
* What is the field of play?
* Give feedback to the user

We picked a simple to understand sport
A fun sample code application

# what is bean bag toss
* boards are 2 ft x 4 ft
* six inch diameter hole
* 25 feet apart
* players throw bean bags
* score for landing on the board
* extra score for landing in the hole

# analysis
* why did I miss?
	* how was the bag flying?
	* what was my pose?
	* how fast was my throw?
	* how do I keep score?
	* Can I show off with different shots?


# demo
* beanbag path
* skeleton on top of player

# Key algorithms playing together
* Find boards -> custom model, `VNCoreMLRequest`
* Scene stability -> `VNTranslationalImageRegistrationRequest`

* measure boards -> `VNDetectContourRequest`
* find player -> `VNDetectHumanBodyPoseRequest`

Gameplay
* find throws -> `VNDetectTrajectoryiesRequest`
* analyze throw type -> CoreML predict from model
* Measure speed -> Measure speed from trajectory

# Detect boards and recognize them
Custom Object Detection model
Create ML object detection template
Our training data
* positive
* negatives

Inference through Vision.

# Why fixate the camera?
Some algorithms require a stable scene
Only need to analyze the playing field once
The user shows clear intent.  So we don't need a "start" button.

`VNTranslationalImageRegistrationRequest`.  Analyzes movement from one frame to the next.
Once below threshold the scene is stable.

# Contour detection
[[Explore Computer Vision APIs]]
* `VNDetectContoursREquest`
* Use the detected object as region of interest
* Simplify the contours for analysis

# Player
`VNDEtectHumanBodyPoseRequest`
 * points of the body joints
 * Use the points to analyze joiont angles

# Demo
Note these seem to be modeled as a giant state machine.
https://developer.apple.com/documentation/vision/building_a_feature-rich_app_for_sports_analysis

Seperate VCs for each state that are presented.  then they receive state change notifications.

# Trajectory detection
* VNDetectTrajectoriesRequest
* finds moving objects
* filters out noise

This request is stateful, which is a bit different than other vision requests.
We feed it the first frame, nothing happens
in frame 5, we finally detect the trajectory.  

Throw gets reported in a `VNTrajectoryObservation`.  But it started much earlier.  

`VNTrajectoryObservation` -> composited frame differentials of the throw
points are the centroids of the object
Protjected points are 5 points that describe the exact parabola being tracked
EquationCoefficients describe the parabolic equation of `y = ax^2 + bx + c`.

Note that there is a second parabola on the bottom - this is the parabola of the shadow of the beanbag!
I can use the UUID to keep track of which parabola.  

Can filter by
* frame analysis spacing – avoid doing work for all frames
* trajectorylength - filter out short trajectories
* minimim/maximum object size - filter out objects that aren't interesting

all this improves performance

## Requirements
Requires a stable scene
Object travels on a parabola (lines are parabolas)
Requires SampleBuffers with timestamps
Bounces create new trajectories – same when leaving the frame
Use Region of Interest.  
Use your business logic

## Demo

# Action classification
Create ML action classification teomplate
Our training data
* classes
* negatives

## demo

# Measuring
Understanding the playing field
* we know the physical size of the board
* Measure the contour of the board
* Pixel measurement corresponds to th eobject size
* trajectory is in the sma plane we can calculate the board
[[Explore Computer Vision APIs]]

Release angle
* bodypose at the beginning of the throw
* Calculate angle between relevant points – elbow-wrist vs horizon

# CoreML stuff
## object detection
* Captured with iPhone
* Included our boards
* Expected environment
* Included people
* Included beanbags
* Varied distance and angles


## action classification
* captured with iPhone
* Varied distances and angles
* Initially 3 classes: underhand, overhand, ?
	* However that means all actions were recognized as some action
	* Needed to add negative class
* Prediction window.  Must include entire target action.  Some actions may take more than others.  Prediction window needs to be large enough for the full action.
* Need to determine when and how often to perform the prediction.  Here we didn't want to continuously perform predictions.  Instead we do this once a throw is completed.

# Best practices for live processing
challenge
* camera has only finite buffers
	* Buffers you hold are not available to the camera
	* Release them as soon as you can
* Load on the system varies
	* Use `didDrop` to find out why?
* Multiple tasks can run on multiple queues
	* Don't wait on rendering your results

## Dealing with live playback
* similar problem as live capture
* Don't go frame by frame, because we want to play at the same time.
* Use `AVPlayerItemVideoOutput` and a `CADisplayLink`

# Wrap up
Analyzing actions and sports is exciting



