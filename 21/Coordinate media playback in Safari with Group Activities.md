#webkit #safari 


Now with group activities, it's easier than ever to provide a shareplay experience to your users wherever they are.

# Coordination and Safari
* GroupActivities in iOS 15
* GroupActivities in macOS Monterey
* SharePlay in Safari 15

Safari will be launched for a URL with shared content.  Pause/play/seek.  See the same commands reflected in their local video at the same time.
# Preparing your app
```swift
struct WatchTogether: GroupActivity {

    var contentIdentifier: String

    func metadata() async -> GroupActivityMetadata {
        var metadata = ActivityMetadata()
        metadata.fallbackURL = URL(string: "https://example.com/title/\(contentIdentifier)")
        return metadata
    }
}
```

Not just your website but the specific content that is played.  when invitation is issued, safari will be launched and asked to load this URL.
# Adopting Media Session
* Standard web API
* Expose metadata to the browser (play state, duration, artwork,etc)
* Handle actions from the browser
* Displays inside Now Playing.

With play action handler, can do preflighting etc.

```swift
if (navigator.mediaSession) {
    navigator.mediaSession.setActionHandler('play', () => video.play() );
    navigator.mediaSession.setActionHandler('pause', () => video.pause() );
    navigator.mediaSession.setActionHandler('seekto', details => {
        video.currentTime = details.seekTime;
    });
}

let updateMediaSessionState = function() {
    if (!navigator.mediaSession)
        return;
    let playbackState = video.paused ? 'paused' : 'playing';
    navigator.mediaSession.playbackState = playbackState;

    let positionState = { video.duration, video.playbackRate, video.currentTime };
    navigator.mediaSession.setPositionState(positionState);
};

for (var event of ['playing', 'pause', 'durationchange', 'ratechange', 'timechange'])
    video.addEventListener(event, updateMediaSessionState);

navigator.mediaSession.metadata = new MediaMetadata({
    title: myPlayer.titleString,
    artwork: [{ src: myPlayer.artworkURL }]
});
```


# Adopting Coordinator
* Send playback intents to UA
* Communicates with users in session
* Works with Media Session

* Experimental web API
* Safari only, though GroupActivities framework can be adopted by other mac browsers
* Receive only

```swift
navigator.mediaSession.addEventListener('coordinatorchange', coordinator => {
    if (coordinator)
        coordinator.join();
    controls.inSessionIcon.style.hidden = !coordinator;
});

controls.inSessionIcon.addEventListener('click', event => {
    let coordinator = navigator.mediaSession.coordinator;
    if (coordinator && coordinator.state == 'joined')
        navigator.mediaSession.coordinator.leave();
});

controls.playButton.addEventHandler('click', event => {
    if (navigator.mediaSession.coordinator)
        navigator.mediaSession.coordinator.play();
    else
        video.play();
});
controls.pauseButton.addEventHandler('click', event => {
    if (navigator.mediaSession.coordinator)
        navigator.mediaSession.coordinator.pause();
    else
        video.pause();
});
controls.timeline.addEventHandler('onchange', event => {
    if (navigator.mediaSession.coordinator)
        navigator.mediaSession.coordinator.seekTo(event.target.value);
    else
        video.currentTime = event.target.value;
});
```

# Wrap up
* Users will love SharePlay in your apps
* Bring SharePlay to macOS Safari with media Session and Coordinator

[[Coordinate media playback with group activities]]
[[Design for Group Activities]]
[[Develop advanced web content]]

* https://developer.apple.com/documentation/avfoundation/media_playback_and_selection/supporting_coordinated_media_playback
* https://developer.apple.com/documentation/GroupActivities

