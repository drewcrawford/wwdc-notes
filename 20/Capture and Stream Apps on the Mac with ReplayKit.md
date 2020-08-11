* In-app screen recording
* in-app screen capture
* in-app screen broadcast

Originally launched for iOS and tvOS.  This year, coming to macOS.

# Goals
* feature parity
* HD quality
* Low performance impact
* Minimal power usage
* Privacy safeguards
* User consent prompt

# Screen recording
* Record app visuals and audio
* Record microphone input
* Share recordings

app calls `RPScreenRecorder`.  Shared instance will write samples into movie file.  Presents with `RPPreviewViewController`.

## Demo
Explanation of how IB works.  Explanation of how a combined start/stop button works.

`RPPreviewViewController` -> editing, saving.

## Screen recording
* applications do not have access to recorded video
* ReplayKit handles creation of movie
* User handle sharing/saving of movie
* Developer requests `startRecording` and `stopRecording`.

What about apps that want access to the movie?

Can now call `stopRecording(withOutputURL: url)`

* developer supplied URL
* ReplayKit will write/save video
* Direct access to video file

What if I want more control of application's video/auto content?

# Screen capture
* record app visuals and audio
* Record microphone input
* Samples sent directly to application

## Demo
Explanation of how buttons work again.

`RPScreenRecorder.shared().startCapture`.  Two handlers
* first handler is the sampler handler.  This is executed every time RPKit gives you a sample, whether it's audio/video/microphone.
* Second handler is the completion handler.  Only once.  Indicates that capture has started.

What happens when you're ready to take things global?

# Live broadcast
* broadcast app visuals and audio
* Xcode templates

Use `RPBroadcastActivityController` to display a picker for 3rd-party services.

## API
`RPBroadcastActivityController` -> allow user to select broadcaster
`RPBroadcastController` -> manage broadcast, start, resume, etc.
`RPBroadcastControllerDelegate` -> handle broadcast events

## demo
Explanation of how delegates work.

## RPBroadcastActivityController
* Present broadcast picker macOS
* CGPoint represents top left corner of broadcast picker?  There's a lot of conflict over which corner specifically, he says "bottom left" twice, the slide has "top left"
* Windiw is passed in as reference coordinate space

Due to macOS supporting multiple screens/windows, we changed the API for this workflow.

With ReplayKit on macOS, there's a menu bar item shown during recording.

# Status bar interactions
* privacy safeguard
* user action stop
* respond to screen recorder delegate

# Game controller framework
* start and stop recording from controller
* Recordings automatically saved to photos/desktop

Good idea to make sure you're using KVO for availability and recording properties so you can update your state as needed.

# Recap
* in-app screen recording
* in-app screen capture
* in-app screen broadcast
* controllers

# Next steps
* sample code project
* ReplayKit documentation


