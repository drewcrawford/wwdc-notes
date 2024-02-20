bLearn how people can interact with your media app directly from HomePod. We'll show you how to add a media intent to your iPhone or iPad app and help people stream your content to a HomePod speaker over AirPlay simply by using their voice. Explore implementation details and get tips and best practices on how to create a great experience for music, audiobooks, podcasts, meditations, or other media types. To learn more about creating a great AirPlay experience, check out "Tune up your AirPlay audio experience” from WWDC23.

allow apps currenrtly supporting mediakit intents to play content on homepod

# Homepod support
1.  Intent sent to iOS
2. airplay back

homepod communicates over wifi.  Devices need to be on the same network.  Don't need to be in physical proximity.

later I will talk about a few things to make UX even better.

* play media
* add to playlist/library
* like/dislike

many existing media apps are now available on homepod.  Podcasts, radio, etc.  Any app that supports media intents is automatically supported.

[[Explore enhancements to App Intents]]

# Integrating your app
apps with sirkit media intents just work.

playback using siri
[[Introducing Siri MediaKit Intents - 19]]
[[Expand your SiriKit Media Intents to More Platforms]]

media name, media type.
intents you might get
* play with an app name.  Intent without any search criteria.  Music apps could play personal mix or station, audiobook might resume, etc.
* for play 'music', we have a mediaType
* entityName -> media name field only.  Up to the app to determine ex artist, song, etc.
* mentioning "the song Nervous" will set mediatype and name
* popular hits -> mediaType: music, sortOrder: popular
* play something next -> playbackQueueLocation is next
* if you r app supports audiobooks, you might receive something like 'my' and 'audiobook'

siri executes the handle step.
1.  handleInApp -> start the app and play in bg
2. continueInApp -> start the app in foreground
you want bg audio for the best experience.


handle callbacks within 10 seconds

app-specific vocabulary.  Can provide purchased content, etc.  Normally any foreground app can start, but during siri requests you can start audio in bg.  Have audio session properly configured.  

1.  set audio category to playback.  
2. mode/policy will affect how we do interruptions.  spokenAudio will paulse playback instead of ducking.  Useful for apps that play podcasts or audiobooks.

if you don't set to playback.  We stop playback when locked.  

* set audio category mode, and route-sharing policy
* populate now playing via MPNowPlayingInfoCenter
* receive remote comamnds using MPRemoveCommandCenter
* buffered playback - smoother playback with nod elays
[[Tune up your AirPlay audio experience]]


# Additional considerations
siri will route media requests to primary device.  The one set up to share yuor location on appleID/find my.

requesting user will need to be registered in home and recognize my voice.  Find this setting in the home app.  Enabling personal requests is not required.  If you don't have recognize my voice enabled, won't be able to play media from your or anyone else's devices.

notification from homepod.  

# Wrap up
sirikit media intents work on homepod
buffered airplay is best

# Resources
* https://developer.apple.com/documentation/sirikit/media/managing_audio_with_sirikit
* https://developer.apple.com/documentation/sirikit
