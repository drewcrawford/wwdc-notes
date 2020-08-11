#avfoundation 

We recommend that you take advantage of recording stereo whenever possible.

# Beamforming
Can focus audio in particular directions.

To get stereo, we record from all mics simultaneously and do some special processing.

Left-right are in distinct directions, and the forward direction is in the direction fo the back camera.  Note that his diagrams are a bit confusing, but it appears the sides of an iPhone (like where you'd find the lock button or volume controls respectively) are the left/right channels.

upperFront stereo -> the reversed direction.

However, users can hold the device in many different orientations.  And we can't do the same thing in landscape.  But we can do a similar model.

`inputOrientation` -> matches orientation of device.
x2 datasources (point at user or point like camera)
8 cases.

## Recommendations for setting input orientation
* Make audio match video
* Follow UI orientation if you do not record video
* Hold the same orientation when recording
* Meet your user's expectations
* Find the right logic for you

## Getting the correct directions
The system has to know the geometric relationship between the mics and your app's audio
The data source determines the forward direction, which is given special emphasis
The directions for left and right depend on input orientatioin and datasource
Test comprehensively!


```swift
newDataSource.setPreferredPolarPattern(.stereo)
try preferredInput.setPreferredDataSource(newDataSource)
//note: if there's some other app that happens to be in control, you might not get what you want here
try session.setPreferredInputOrientation(orientation.inputOrientation)
```

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitcollection?) {
	updateDataSource()
}
```

Maybe don't change while recording?

# Wrap up
Record stereo for the most immersive audio experience
Choose a datasource that matches your intended focus direction
Set stereo input orientation to match user expectation
