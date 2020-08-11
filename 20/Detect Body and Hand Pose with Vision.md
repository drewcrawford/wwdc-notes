#vision

# People
* Face detector
* Face landmarks
* Human detector

# Hand pose
1.  `VNImageRequestHandler`
2.  `VNDetectHumanHandPoseRequest`
3.  `performRequests`
4.  get back `VNRecognizedPointsObservation`

## How are landmarks represented?
`VNPoint`, with a `CGPoint` location.

If there's a confidence, you get `VNDetectedPoint` with `confidence`

If they have identifiers, `VNRecognizedPoint` with `identifier`.

```swift
extension CameraViewController: AVCaptureVideoDataOutputSampleBufferDelegate {
    public func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
        var thumbTip: CGPoint?
        var indexTip: CGPoint?
        
        defer {
            DispatchQueue.main.sync {
                self.processPoints(thumbTip: thumbTip, indexTip: indexTip)
            }
        }

        let handler = VNImageRequestHandler(cmSampleBuffer: sampleBuffer, orientation: .up, options: [:])
        do {
            // Perform VNDetectHumanHandPoseRequest
            try handler.perform([handPoseRequest])
            // Continue only when a hand was detected in the frame.
            // Since we set the maximumHandCount property of the request to 1, there will be at most one observation.
            guard let observation = handPoseRequest.results?.first as? VNRecognizedPointsObservation else {
                return
            }
            // Get points for thumb and index finger.
            let thumbPoints = try observation.recognizedPoints(forGroupKey: .handLandmarkRegionKeyThumb)
            let indexFingerPoints = try observation.recognizedPoints(forGroupKey: .handLandmarkRegionKeyIndexFinger)
            // Look for tip points.
            guard let thumbTipPoint = thumbPoints[.handLandmarkKeyThumbTIP], let indexTipPoint = indexFingerPoints[.handLandmarkKeyIndexTIP] else {
                return
            }
            // Ignore low confidence points.
            guard thumbTipPoint.confidence > 0.3 && indexTipPoint.confidence > 0.3 else {
                return
            }
            // Convert points from Vision coordinates to AVFoundation coordinates.
            thumbTip = CGPoint(x: thumbTipPoint.location.x, y: 1 - thumbTipPoint.location.y)
            indexTip = CGPoint(x: indexTipPoint.location.x, y: 1 - indexTipPoint.location.y)
        } catch {
            cameraFeedSession?.stopRunning()
            let error = AppError.visionError(error: error)
            DispatchQueue.main.async {
                error.displayInViewController(self)
            }
        }
    }
}
```
* Four landmarks for each finger
* Four for thumb
* 1 for wrist

### fingers

Index finger -> handLandmarkRegionKey`Index` finger.
Can retrieve TIP of finger.

1.  Tip
2.  DIP (Distal Interphalangeal joint)
3.  PIP (Proximal interphalangeal point)
4.  MCP Metacarpophalangeal joint

Pattern repeats for each finger.
Thumb is a bit different.

### thumb

1.  Tip
2.  IP
3.  MP
4.  CMC

### wrist

Wrist.  Only included in `VNRecognizedPointGroupKeyAll`.

### demo

see code above

```swift
init(pinchMaxDistance: CGFloat = 40, evidenceCounterStateTrigger: Int = 3) {
        self.pinchMaxDistance = pinchMaxDistance
        self.evidenceCounterStateTrigger = evidenceCounterStateTrigger
    }
    
    func reset() {
        state = .unknown
        pinchEvidenceCounter = 0
        apartEvidenceCounter = 0
    }
    
    func processPointsPair(_ pointsPair: PointsPair) {
	//once we've collected 3 frames of evidence, we move states
	//until we accumulate enough evidence, we're in the 'possible' state
        lastProcessedPointsPair = pointsPair
        let distance = pointsPair.indexTip.distance(from: pointsPair.thumbTip)
        if distance < pinchMaxDistance {
            // Keep accumulating evidence for pinch state.
            pinchEvidenceCounter += 1
            apartEvidenceCounter = 0
            // Set new state based on evidence amount.
            state = (pinchEvidenceCounter >= evidenceCounterStateTrigger) ? .pinched : .possiblePinch
        } else {
            // Keep accumulating evidence for apart state.
            apartEvidenceCounter += 1
            pinchEvidenceCounter = 0
            // Set new state based on evidence amount.
            state = (apartEvidenceCounter >= evidenceCounterStateTrigger) ? .apart : .possibleApart
        }
    }
```
	

```swift
private func handleGestureStateChange(state: HandGestureProcessor.State) {
        let pointsPair = gestureProcessor.lastProcessedPointsPair
        var tipsColor: UIColor
        switch state {
        case .possiblePinch, .possibleApart:
            // We are in one of the "possible": states, meaning there is not enough evidence yet to determine
            // if we want to draw or not. For now, collect points in the evidence buffer, so we can add them
            // to a drawing path when required.
            evidenceBuffer.append(pointsPair)
            tipsColor = .orange
        case .pinched:
            // We have enough evidence to draw. Draw the points collected in the evidence buffer, if any.
            for bufferedPoints in evidenceBuffer {
                updatePath(with: bufferedPoints, isLastPointsPair: false)
            }
            // Clear the evidence buffer.
            evidenceBuffer.removeAll()
            // Finally, draw current point
            updatePath(with: pointsPair, isLastPointsPair: false)
            tipsColor = .green
        case .apart, .unknown:
            // We have enough evidence to not draw. Discard any evidence buffer points.
            evidenceBuffer.removeAll()
            // And draw the last segment of our draw path.
            updatePath(with: pointsPair, isLastPointsPair: true)
            tipsColor = .red
        }
        cameraView.showPoints([pointsPair.thumbTip, pointsPair.indexTip], color: tipsColor)
    }
```

```swift
//look for a double-tap to clear the screen
@IBAction func handleGesture(_ gesture: UITapGestureRecognizer) {
        guard gesture.state == .ended else {
            return
        }
        evidenceBuffer.removeAll()
        drawPath.removeAllPoints()
        drawOverlay.path = drawPath.cgPath
    }
```

## Many hands
`maximumHandCount` on the request.  Default is 2, so if you want more than 2 you have to adjust this.

Setting this parameter to a large number will have a latency impact.  Recommended that you tune this for your application.

## VNTrackObjectRequest
Optionally, use `VNTrackObjectREquest`
[[Object Tracking in Vision - 18]]

You might want to use a hand pose request to find the hand, and then use object tracking to follow it around, if all you need is the general location and not the pose.

Or for robustness, to keep track of which hand is which.

## Accuracy considerations
* Near edges of the screen
* Parallel to viewing direction
* Gloves
* May sometimes detect feet

# Human body pose

## Multi-person pose estimation

To analyze imaes, the flow is very similar.
0.  Request handler
1.  `VNDetectHumanBodyPoseRequest`
2.  Use handler to perform request

## Landmarks
VNRecognizedPointGroupKey for each group

### face
* Nose
* Left/right eye
* left/right ear

### arms
*Subject's* right arm, not the one at the right of the image.
* shoulder
* elbow
* wrist

### torso
* left/right shoulder
* neck
* left/right hip
* root

Note that the shoulder is also included in arms, so these points exist in more than 1 group.

### left/right leg
* hip
* knee
* ankle

again, hip belongs to more than one group

## considerations
* bent-over or upsidedown
* flowing or robe-like clothing
* People occluding each other
* Near edges of screen
* Same tracking considerations as for hand pose

## comparison to ARKit body pose
|              | Vision                   | ARKit                                                            |
|--------------|--------------------------|------------------------------------------------------------------|
| Landmarks    | 19                       | 19                                                               |
| Confidence   | Yes                      | No                                                               |
| Purpose      | Offline or live analysis | Live motion capture                                              |
| Availability | macOS, tvOS, iOS, iPadOS | Rear-facing camera,  ARSession, Supported iOS and iPadOS devices |

## Action classification training with body pose
Crop training videos to contain a single actor
If more than one is found, the largest one is used

Alternatively, use `MLMultiArray` buffers of poses for training
Use Vision to compute training samples offline

## Action classification inference with body pose
Always create inference examples from Visions's `VNIdentifiedPointsObservation keypointsMultiArray`
Use of ARKit body pose will produce undefined results
Inference speed varies across devices

You will need to pose estimate every frame, because that's what the model is built for / expects.
But consider classifying more occasionally / spacing out, to avoid overloading older devices.

[[Explore the Action & Vision app]]


```swift
extension GameViewController: CameraViewControllerOutputDelegate {
    func cameraViewController(_ controller: CameraViewController, didReceiveBuffer buffer: CMSampleBuffer, orientation: CGImagePropertyOrientation) {
        let visionHandler = VNImageRequestHandler(cmSampleBuffer: buffer, orientation: orientation, options: [:])
        if self.gameManager.stateMachine.currentState is GameManager.TrackThrowsState {
            DispatchQueue.main.async {
                // Get the frame of rendered view
                let normalizedFrame = CGRect(x: 0, y: 0, width: 1, height: 1)
                self.jointSegmentView.frame = controller.viewRectForVisionRect(normalizedFrame)
                self.trajectoryView.frame = controller.viewRectForVisionRect(normalizedFrame)
            }
            // Perform the trajectory request in a separate dispatch queue
            trajectoryQueue.async {
                self.setUpDetectTrajectoriesRequest()
                do {
                    if let trajectoryRequest = self.detectTrajectoryRequest {
                        try visionHandler.perform([trajectoryRequest])
                    }
                } catch {
                    AppError.display(error, inViewController: self)
                }
            }
        }
        // Run bodypose request for additional GameConstants.maxPostReleasePoseObservations frames after the first trajectory observation is detected
        if !(self.trajectoryView.inFlight && self.trajectoryInFlightPoseObservations >= GameConstants.maxTrajectoryInFlightPoseObservations) {
            do {
                try visionHandler.perform([detectPlayerRequest])
                if let result = detectPlayerRequest.results?.first as? VNRecognizedPointsObservation {
                    let box = humanBoundingBox(for: result)
                    let boxView = playerBoundingBox
                    DispatchQueue.main.async {
                        let horizontalInset = CGFloat(-20.0)
                        let verticalInset = CGFloat(-20.0)
                        let viewRect = controller.viewRectForVisionRect(box).insetBy(dx: horizontalInset, dy: verticalInset)
                        self.updateBoundingBox(boxView, withRect: viewRect)
                        if !self.playerDetected && !boxView.isHidden {
                            self.gameStatusLabel.alpha = 0
                            self.resetTrajectoryRegions()
                            self.gameManager.stateMachine.enter(GameManager.DetectedPlayerState.self)
                        }
                    }
                }
            } catch {
                AppError.display(error, inViewController: self)
            }
        } else {
            // Hide player bounding box
            DispatchQueue.main.async {
                if !self.playerBoundingBox.isHidden {
                    self.playerBoundingBox.isHidden = true
                    self.jointSegmentView.resetView()
                }
            }
        }
    }
}
```
```swift
//trying to find the bounding box around a person
func humanBoundingBox(for observation: VNRecognizedPointsObservation) -> CGRect {
        var box = CGRect.zero
        // Process body points only if the confidence is high
        guard observation.confidence > 0.6 else {
            return box
        }
        var normalizedBoundingBox = CGRect.null
        guard let points = try? observation.recognizedPoints(forGroupKey: .all) else {
            return box
        }
        for (_, point) in points {
            // Only use point if human pose joint was detected reliably
            guard point.confidence > 0.1 else { continue }
            normalizedBoundingBox = normalizedBoundingBox.union(CGRect(origin: point.location, size: .zero))
        }
        if !normalizedBoundingBox.isNull {
            box = normalizedBoundingBox
        }
        // Fetch body joints from the observation and overlay them on the player
        DispatchQueue.main.async {
            let joints = getBodyJointsFor(observation: observation)
            self.jointSegmentView.joints = joints
        }
        // Store the body pose observation in playerStats when the game is in TrackThrowsState
        // We will use these observations for action classification once the throw is complete
        if gameManager.stateMachine.currentState is GameManager.TrackThrowsState {
            playerStats.storeObservation(observation)
            if trajectoryView.inFlight {
                trajectoryInFlightPoseObservations += 1
            }
        }
        return box
    }
```
```swift
mutating func storeObservation(_ observation: VNRecognizedPointsObservation) {
		//this is a ring buffer.  If it's full, we remove the first one
        if poseObservations.count >= GameConstants.maxPoseObservations {
            poseObservations.removeFirst()
        }
		//append thee next one
        poseObservations.append(observation)
    }
```
```swift
//a throw is detected when the trajectory analysis detects a throw
//then we call this func to find out what kind of throw was made
mutating func getLastThrowType() -> ThrowType {
        let actionClassifier = PlayerActionClassifier().model
        guard let poseMultiArray = 
		//get our observations in the correct format
		prepareInputWithObservations(poseObservations) else {
            return ThrowType.none
        }
        let input = PlayerActionClassifierInput(input: poseMultiArray)
        guard let predictions = try? actionClassifier.prediction(from: input),
            let output = predictions.featureValue(for: "output")?.multiArrayValue,
                let outputBuffer = try? UnsafeBufferPointer<Float32>(output) else {
            return ThrowType.none
        }
        let probabilities = Array(outputBuffer)
        guard let maxConfidence = probabilities.prefix(3).max(), let maxIndex = probabilities.firstIndex(of: maxConfidence) else {
            return ThrowType.none
        }
        let throwTypes = ThrowType.allCases
        return throwTypes[maxIndex]
    }
```
```swift
func prepareInputWithObservations(_ observations: [VNRecognizedPointsObservation]) -> MLMultiArray? {
    let numAvailableFrames = observations.count
    let observationsNeeded = 60 //need 60 observations
    var multiArrayBuffer = [MLMultiArray]() //put them in this buffer

    // swiftlint:disable identifier_name
	//iterate over observations in our buffer
    for f in 0 ..< min(numAvailableFrames, observationsNeeded) {
        let pose = observations[f]
        do {
            let oneFrameMultiArray = try pose.keypointsMultiArray()
            multiArrayBuffer.append(oneFrameMultiArray)
        } catch {
            continue
        }
    }
    
    // If poseWindow does not have enough frames (60) yet, we need to pad 0s
    if numAvailableFrames < observationsNeeded {
        for _ in 0 ..< (observationsNeeded - numAvailableFrames) {
            do {
                let oneFrameMultiArray = try MLMultiArray(shape: [1, 3, 18], dataType: .double)
                try resetMultiArray(oneFrameMultiArray)
                multiArrayBuffer.append(oneFrameMultiArray)
            } catch {
                continue
            }
        }
    }
    return MLMultiArray(concatenating: [MLMultiArray](multiArrayBuffer), axis: 0, dataType: MLMultiArrayDataType.double)
}
```
```swift
mutating func getLastThrowType() -> ThrowType {
        let actionClassifier = PlayerActionClassifier().model
        guard let poseMultiArray = prepareInputWithObservations(poseObservations) else {
            return ThrowType.none
        }
        let input = PlayerActionClassifierInput(input: poseMultiArray)
        guard let predictions = try? actionClassifier.prediction(from: input),
            let output = predictions.featureValue(for: "output")?.multiArrayValue,
                let outputBuffer = try? UnsafeBufferPointer<Float32>(output) else {
            return ThrowType.none
        }
        let probabilities = Array(outputBuffer)
        guard let maxConfidence = probabilities.prefix(3).max(), let maxIndex = probabilities.firstIndex(of: maxConfidence) else {
            return ThrowType.none
        }
        let throwTypes = ThrowType.allCases
        return throwTypes[maxIndex]
    }
```