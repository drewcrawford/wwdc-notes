How you can enhance voice communication with new framework. #pushtotalk

# Introducing PushToTalk
What is it?
Supports walkie-talkie style communication
To provide a great experience, users need to quickly access audio transmission features.  At the same time, must be power efficient.

Consistent and accessible UI.  Record and stream audio to your server
Receive and play audio from the background
New push notification type that notifies your app when new audio is available for playback.
When you receive this notification, launched in the background.

## Compatibility
* backend servers and infrastructure
* Audio configuration and encoding
* Custom wireless accessory integration

## demo
When there's an active channel, a blue pill appears in the status bar.  PIll shows system UI.  Displays the name of the channel and an image provided by the app.  Respond to the message, leave the channel, etc.

PTT system UI can be accessed from the lock screen.  Reeive and respond without having to unlock the device.
# Configuring PushToTalk
## Configuring your xcode project
* configure background mode
* Enable PTT capability
* Push notification capabilitiy required
* Request microphone permission
## Joining a channel
* create a channel manager
* create a channel descriptor
* join the channel
```swift
func setupChannelManager() async throws {
    channelManager = try await PTChannelManager.channelManager(delegate: self,
                                                               restorationDelegate: self)
}
```

Provide a channel manager delegate and restoration delegate.  Same shared instan ce being returned, channel manager in instance variable.  Important to initialize as soon as possible during startup.

Ensures that the channel manager is intiialized quickly so that existing channels can be restored.

```swift
func joinChannel(channelUUID: UUID) {
    let channelImage = UIImage(named: "ChannelIcon")
    channelDescriptor = PTChannelDescriptor(name: "Awesome Crew", image: channelImage)
  
    // Ensure that your channel descriptor and UUID are persisted to disk for later use.
    channelManager.requestJoinChannel(channelUUID: channelUUID, 
                                      descriptor: channelDescriptor)
}
```

It is only possible to join a channel from the foreground.

```swift
func channelManager(_ channelManager: PTChannelManager, 
                    didJoinChannel channelUUID: UUID,
                    reason: PTChannelJoinReason) {
    // Process joining the channel
    print("Joined channel with UUID: \(channelUUID)")
}

func channelManager(_ channelManager: PTChannelManager,
                    receivedEphemeralPushToken pushToken: Data) {
    // Send the variable length push token to the server
    print("Received push token")
}
```

Called with the push token that can be used to send PTT notifications.  This token will only be active for the life of the PTT channel.  APNS push tokens are variable-length, do not hardcode the length.

```swift
func channelManager(_ channelManager: PTChannelManager, 
                    failedToJoinChannel channelUUID: UUID, 
                    error: Error) {
    let error = error as NSError

    switch error.code {
    case PTChannelError.channelLimitReached.rawValue:
        print("The user has already joined a channel")
    default:
        break
    }
}
```

```swift
func channelManager(_ channelManager: PTChannelManager,
                    didLeaveChannel channelUUID: UUID,
                    reason: PTChannelLeaveReason) {
    // Process leaving the channel
    print("Left channel with UUID: \(channelUUID)")
}
```

restoring previous channels, when the app is terminated or device reboots
```swift
func channelDescriptor(restoredChannelUUID channelUUID: UUID) -> PTChannelDescriptor {
    return getCachedChannelDescriptor(channelUUID)
}
```

return from this method as quickly as possible.  Do not perform long-running, blocking tasks such as network requests.

providing updates
* channel descriptor
* service status change

```swift
func updateChannel(_ channelDescriptor: PTChannelDescriptor) async throws {
    try await channelManager.setChannelDescriptor(channelDescriptor, 
                                                  channelUUID: channelUUID)
}
```
update the name or image

indicate network situation

```swift
func reportServiceIsReconnecting() async throws {
    try await channelManager.setServiceStatus(.connecting, channelUUID: channelUUID)
}

func reportServiceIsConnected() async throws {
    try await channelManager.setServiceStatus(.ready, channelUUID: channelUUID)
}
```


# Transmitting and receiving audio
* multiple ways to begin transmission
* app ui
* system ui
* corebluetooth accessory

Enables microphone access
System activates the AVAudioSession

```swift
func startTransmitting() {
    channelManager.requestBeginTransmitting(channelUUID: channelUUID)
}

// PTChannelManagerDelegate

func channelManager(_ channelManager: PTChannelManager, 
                    failedToBeginTransmittingInChannel channelUUID: UUID,
                    error: Error) {
    let error = error as NSError

    switch error.code {
    case PTChannelError.callIsActive.rawValue:
        print("The system has another ongoing call that is preventing transmission.")
    default:
        break
    }
}
```

if the user has a cellular call active, they will not be able to begin transmission.
To stop transmitting, call the channel manager's stopTransmitting method.  To handle failures, such as when the user was not in a transmitting state, the delegate has an associated `failedToStopTransmitting` meethod.
```swift
func stopTransmitting() {
    channelManager.stopTransmitting(channelUUID: channelUUID)
}

func channelManager(_ channelManager: PTChannelManager, 
                    failedToStopTransmittingInChannel channelUUID: UUID, 
                    error: Error) {
    let error = error as NSError

    switch error.code {
    case PTChannelError.transmissionNotFound.rawValue:
        print("The user was not in a transmitting state")
    default:
        break
    }
}
```

delegates

```swift
func channelManager(_ channelManager: PTChannelManager,
                    channelUUID: UUID, 
                    didEndTransmittingFrom source: PTChannelTransmitRequestSource) {
    print("Did end transmission from: \(source)")
}

func channelManager(_ channelManager: PTChannelManager,
                    didDeactivate audioSession: AVAudioSession) {
    print("Did deactivate audio session")
    // Stop recording and clean up resources
}
```

system activates audio session for your half.  Your signal that you can now beginr ecording.  **Do not start/stop your own audio session.**

While your transmisison is active, audio sesison may be interrupted by other sources – phone calls, etc.  Handle these.

Receiving audio

* new APNS push type wakes your app
* you received push token when joining the channel
* app must report the active remote speaker
* activates the AVAudioSession

notification payload

APNS push type must be set to `pushtotalk`
APNS topic header must include the `.voip-ptt` suffix to your bundle identifier
payload should contain speaker data
Set priority to 10 => request immediate delivery
expiration to 0 => prevent older pushes from being delivered later

```swift
func incomingPushResult(channelManager: PTChannelManager, 
                        channelUUID: UUID, 
                        pushPayload: [String : Any]) -> PTPushResult {

    guard let activeSpeaker = pushPayload["activeSpeaker"] as? String else {
        // If no active speaker is set, the only other valid operation 
        // is to leave the channel
        return .leaveChannel
    }

    let activeSpeakerImage = getActiveSpeakerImage(activeSpeaker)    
    let participant = PTParticipant(name: activeSpeaker, image: activeSpeakerImage)
    return .activeRemoteParticipant(participant)
}
```

When you receive a push payload, need to construct a push result type to indicate what action should be performed.  To indicate that ar emote user is speaking, return a push result that returns the active participants' information.  Name, optional image.

If your server decides to kick a user, may indicate this in the push payload.  Return `.leavechannel` push result.  Return a result from this method as quickly as possible, do ont block the thread.    If you do not have the image stored locally, can return only the name.  Download the iamge on a separate thread, once the image is retrieved update the active remote participant later.

```swift
func stopReceivingAudio() {
    channelManager.setActiveRemoteParticipant(nil, channelUUID: channelUUID)
}
```

This indicates to the system you are no longer receiving audio on the channel and syustem can deactivate your audio channel.  This updates the UI, etc.

# Best practices
Provide a PTT UI within your app
PTT utilizes shared resources
superceded by calls, may fail, etc.
set up audio session on app launch.  activated/deactivated by the system, you configure your category to play+record
sound effects are provided by the system
Respond to AVAudioSession notifications.  Route changes, failures, etc.

Battery life
When your app is not being used by the user, it will be suspended by the system
* let the system activate and deactivate the audio session, do not do this by yourself
* Use APNs to wake your app for a PTT event

[[Power down Improve battery consumption]]

Reduce latency
* app is suspended when not transmitting or receiving
* network connections are disconnected
* adopt QUIC
* [[Reduce networking delays for a more responsive app]]
# next steps
* Migrate existing PTT apps
* New apps should use PTT
* Test and submit feedback

* https://developer.apple.com/forums/tags/wwdc2022-10117
* https://developer.apple.com/forums/create/question?&tag1=381030&tag2=387030
* https://developer.apple.com/documentation/pushtotalk
