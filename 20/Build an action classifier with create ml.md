#createml

[[Building activity classification models in create ml - 19]]

But what if you wanted to classify actions from videos?

# What is action classification?
* Classification task
* Powered by body pose estimation
* Huamn body actions
* Prediction in a time window of video

[[Detect Body and Hand Pose with Vision]]

# How it works
data, train, model

# Training an action classifier in create ML

* One action type per video
* several seconds
* one person
* entire body

Put in folders
Can include a single folder "other"

Montage videos.  We can trip and split videos and group into folders.  Or, we can create an annotation file in JSON/CSV.

New template -> Action Classification.

He has around 50 videos per action.

For short actions, set the interations window to 2 seconds.  This appears to be a fixed window, not variable with the interesting part in the center.
CreateML automatically captures the number of frames.

CreateML will automatically split validation data.

It trains in 2 parts.

* Feature extraction -> pose estimation etc.
* Training

## Feature extraction
Apose includes 18 landmarks.  Hands, legs, hips, eyes, etc.  A landmark includes x/y coordinates, and confidence score.

# iOS app
We have video, however the model takes poses.  So we use `VNDetectHumanBodyPoseRequest()`.

```swift
let poseArray1 = try pose1.keypointsMultiArray()

let modelUInput = MLMultipArray(concatenating: poseMultiArrays, axis: 0, dataType: .float)
```

See input/output stuff in xcode.

## Demo

# Best Practices

* 50+ examples per class
* Diverse subjects
* Include a folder of other poses such as resting
* Keep camera stable
* Good contrast between subject and background
* Avoid loose, flowing clothing

## Choosing training parameters
* Prediction window should include an entire action
* Consistent framerate between training and test videos

## Using model
* Selecting a single person out of multiple people
* counting action repetitions
* scoring or judging quality of an action

