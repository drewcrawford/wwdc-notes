#createml 

Hands are very important and very cool


# Detecting hands
* Detect if hands exist
* Detects where hands are

[[Detect Body and Hand Pose with Vision]]

Short single-handed poses/actions.

# Poses
Expression ssuch as "peace" or "stop"
Equally expressive in still photo or video

# Actions
Moving expression like "follow me", "go away"
Only expressive as a sequence of frames

* Hand pose classification
* hand action classification

# Availability
Big Sur, 14+
* Create ML app and framework
* Trained models compatible with earlier OS releases

[[Build dynamic iOS apps with the Create ML framework]]


# Hand pose classification
* Train models to classify hand poses
* Customized to your app's needs

## Workflow
1.  Collect data
2.  Train model
3.  Integrate into app

### Training data
A folder of categorized hand images
Includes a background class

### Background class
Catch-all category for what you don't care about.  

* Random poses
* Transitional poses

Augmentations to extend my data model.

## Integration
1.  Camera provides a stream of frames
2.  Vision request, `VNDEtectHumanHandPoseRequest`
3.  `VNHumanHandPoseObservation`
4.  `MLMultiArray` => `keypointsMultiArray`

```swift
func session(_ session: ARSession, didUpdate frame: ARFrame) {

    let pixelBuffer = frame.capturedImage 
    let handPoseRequest = VNDetectHumanHandPoseRequest()
    handPoseRequest.maximumHandCount = 1
    handPoseRequest.revision = VNDetectHumanHandPoseRequestRevision1

    let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
    do { 
        try handler.perform([humanBodyPoseRequest]) 
    } catch {
        assertionFailure("Human Pose Request failed: \(error)")
    }

    guard let handPoses = request.results, !handPoses.isEmpty else {
        // No effects to draw, so clear out current graphics
        return
    }
    let handObservation = handPoses.first
```

Only need one instance of `VNDetectHumanHandPoseRequest`.  

We will be doing detection on every frame via ARKit.  Only one way to grab frames from a camera feed.  

```swift
if frameCounter % handPosePredictionInterval == 0 {

    guard let keypointsMultiArray = try? handObservation.keypointsMultiArray() 
else { fatalError() }
    let handPosePrediction = try model.prediction(poses: keypointsMultiArray)
    let confidence = handPosePrediction.labelProbabilities[handPosePrediction.label]!

    if confidence > 0.9 {
       renderHandPoseEffect(name: handPosePrediction.label)
    }
}

func renderHandPoseEffect(name: String) {
	switch name {
        case "One": 
            if effectNode == nil {
               effectNode = addParticleNode(for: .one)
            }
        default:
			removeAllParticleNode()
	}
}
```

Only check occasionally to smooth out jitter.  Confidence interval helps too.

```swift
let landmarkConfidenceThreshold: Float = 0.2

let indexFingerName = VNHumanHandPoseObservation.JointName.indexTip

let width = viewportSize.width
let height = viewportSize.height

if let indexFingerPoint = try? observation.recognizedPoint(indexFingerName),
   indexFingerPoint.confidence > landmarkConfidenceThreshold {
    
    let normalizedLocation = indexFingerPoint.location
    indexFingerTipLocation = CGPoint((x: normalizedLocation.x * width,
                                      y: normalizedLocation.y * height))
} else {
    indexFingerTipLocation = nil
}
```

> Vision's origin point is in the lower-left corner of an image.  

# Chirality
Left vs right.  

left hand right hand, unknown.  Unknown wil appear if older version is deserilaized.

Refer to 2020 session [[Detect Body and Hand Pose with Vision]]

## Multiple hands
* EAch prediction is run on a single hand
* The prediction of one hand in the frame does not affect the prediction of others.

## ex
```swift
// Working with chirality

let handPoseRequest = VNDetectHumanHandPoseRequest()
try handler.perform([handPoseRequest])
let detectedHandPoses = handPoseRequest.results!

for hand in detectedHandPoses where hand.chirality == .right {
    // Take action on every right hand, or prune the results
}
```

Hand action classification is another new technology

# Action classification
Train models to classify sequences of poses
Customized to your app's needs

[[Build an action classifier with create ml]]

# Hand actions
Hand action is a sequence of hand poses
Hand action describes hand motions
Use videos to capture hand actions

## Training data
Videos
Include a background class

Videos in a single folder if annotated.
Annotation file with entries "label", "video", "start" and "end"
Should include a background class

Action duration is a training parameter.  CreateML randomly samples according to this value.

New augmentations for video

Can use left hand for poses and right hand for actions.  
## Input size

Provide correct number of poses.  My model is expecting 45x3x21.  
45: Number of poses the classifier needs to recognize
21: Number of joints provided by vision
3: x/y coordinate components, somethign joint

### Prediction window size
* Number of hand poses to input to the classifier
* 30 fps * 1.5s = 45 frames

Camera frame rate.  The frame rate at inference time matches framerate at training time.

ARKit.  Number of arriving poses per segment.  Since ARKit provides frames at 60fps.  Need to downsample.

## code
```swift
var queue = [MLMultiArray]()
// . . .
frameCounter += 1 //step down to 30fps
if frameCounter % 2 == 0 {
    let hands: [(MLMultiArray, VNHumanHandPoseObservation.Chirality)] = getHands()
    for (pose, chirality) in hands where chirality == .right {
        queue.append(pose)
        queue = Array(queue.suffix(queueSize))
        queueSamplingCounter += 1
        if queue.count == queueSize && queueSamplingCounter % queueSamplingCount == 0 {
            let poses = MLMultiArray(concatenating: queue, axis: 0, dataType: .float32)
            let prediction = try? handActionModel?.prediction(poses: poses)
            guard let label = prediction?.label, 
              let confidence = prediction?.labelProbabilities[label] else { continue }
            if confidence > handActionConfidenceThreshold {
                DispatchQueue.main.async {
                    self.renderer?.renderHandActionEffect(name: label)
                }
            }
        }
    }
}
```

When a new hand pose arrives, I add to the queue and repeat until full.  When full, I can start to read content.

Depending on usecase, too many rquests may be a waste of resources.  Choose the sampling rate as a tradeoff between repsonsiveness and number of predictions per second.

## Demo recap
* Match trained framerate
* Use a FIFO queue to collect windowed sequence of poses
* Read the queue content at frame rate tuned to your usecase

# Quality considerations
* Distance to hand (<11ft)
* Lighting conditions
* Gloves
* Training data.  We used 500 images per class.  Action classifier, 100 videos per class.

# Wrap up
* Hand poses and actions
* Train classifiers with Create ML
* Integration in your app with Vision
* Chirality and multiple models

* https://developer.apple.com/documentation/createml
* https://developer.apple.com/documentation/vision


