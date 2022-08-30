#shareplay 

# Start SharePlay from your app
New API to start shraeplay without a facetime call.

If your app is entitled for shareplay, you get a button in the picker for free.  Not the optimal experience, since the user can't start a group activity through system UI and will need to reinteract with your app.  Let's see how you can adopt new APIS.
```swift
// Register GroupActivity
let itemProvider = NSItemProvider()
itemProvider.registerGroupActivity(WatchTogether())

// Provide the ItemProvider to the ShareSheet
let configuration = UIActivityItemsConfiguration(itemProviders: [itemProvider])

UIActivityViewController(activityItemsConfiguration: configuration)
```

Can tune the behavior with `.allowsProminentActivity`

```swift
let shareSheet = UIActivityViewController(activityItemsConfiguration: configuration)

// Show SharePlay non-prominently
shareSheet.allowsProminentActivity = false
```

what if your content doesn't support shareplay?  Can make not show up in share sheet by excluding the activity  type.

```swift
let shareSheet = UIActivityViewController(activityItemsConfiguration: configuration)

// Exclude SharePlay activity
shareSheet.excludedActivityTypes = [.sharePlay]
```

Can place a button directly in your app

```swift
let controller = GroupActivitySharingController(WatchTogetherActivity())
present(controller, animated: true)
```
# GroupSessionmessenger updates
You may have run into 64kb payload size limits.
We've now made it so payload size is 256kb.  With this change, your app doesn't need to worry about breaking up your message.  Simply send your message.

As part of group sesison messenger, can now choose reliability.  Unreliable messaging is faster.

Each time we receive an update from our GR, we will send our newly added oint usign unrealiable messaging.

```swift
var strokeGesture: some Gesture {
    DragGesture()
        .onChanged { value in
            canvas.addPointToActiveStroke(value.location)
        }
        .onEnded { value in
            canvas.addPointToActiveStroke(value.location)
            canvas.finishStroke()
        }
}
```

clients catch up with reliable communication of all points.



# Best practices

Staged GroupActivity.  

Suppose you start shareplay.  Adam is trying to resume a show.  Wew ant to jump into there at a specific time rather than start over.  If the wrong device activates teh activity, the show may start over.

Playback case => each device contribute its initial playback state to the others.  Same principle applies to any experience you create.  Each person that joins a session should ocntribute their understanding.

Ownerless sessions are a tough concept to grasp.  Owners drop off, maybe that ends the session?

System is still able to call end on the group sesison on a user's behalf.  Whiel it may be a hard concetp to grasp, making sure that your app doesn't have a sense of ownership will result in a better user experience.

# Wrap up
* start shareplay from your app
* explore unreliable messenger
[[Make a great shareplay experience]]
[[Display ads and other interstitials in SharePlay]]





* https://developer.apple.com/forums/tags/wwdc2022-10140
* https://developer.apple.com/forums/create/question?&tag1=91030&tag2=383030
* https://developer.apple.com/design/human-interface-guidelines/shareplay/overview/
* https://developer.apple.com/shareplay/

