#shareplay 

We're going to take you through how to design and build a great SP Experience.
# Discover a new way to connect
imessage: we connect by sharingi mages
facetime: audio/video

but someo f the moments people have are more than just sharing a conversation.  We share *experiences*.

#groupactivities

When someone in an FT call starts an activity, shareplay brings the group directly into your app, where you can create any type of experience.  Cooking together, etc.

[[What's new in SharePlay]]

# Foster presence
What does it mean to be present?  Think of SP like ap ortal, transporting people through their phones and into space.

Imagine everyone's transported into a room where they can use a record player.  

Offer there are reminders taht there are other poeple doing the experience with them.  Design considerations to foster presence.

# Design for groups
many apps are desiend for a single person.  A great personalized experience might not translate to a great group experience.  Opportunity to think of your app from a different perspective.  Design specifically for groups.

Cooking together, yoga, watching teams, etc.  

All of these are good candidates for shraeplay because people love to do them together in person.  Create rich, coordinated experiences for groups.

If you're bulding a coordinated media experience, you can adopt `aVPlaybackCorodinator.corodinateWithSession()`.  System will keep everyone in sync.  

Sometimes we share experiences that participnats view differently.  ex, Heads Up.  
`GruopsessionManager.send(_:to:)`.  


# Build a seamless experience
In order to build a great SP experience, think through the different stages of a sahred activity.  Starting, during, and ending.

`registerGroupActivity`.  People will see the option to shareplay the activity from the sahre sheet.  

Support the ability to SP directly in uI with `GroupActivitySharingController`.  

System will display info about the group activity, title, subtitle, image.  `GroupActivityMetadata`.  Consider a web-based URL for participnats who might not have the app installed on device already.

Just because someone's on a mesages group or facetime call doesn't mean they're joined.  People can navigate to details view.  

Can also show similar information in app's UI.  Great way to know everyone is on the same page as you.  maintain a sense of presence with the group.  

Consider supproting a lobby where people can wait to join before joining the experience.  

Everyone will interact at the same time.  We built a relationship with our devices that things change when you touch them.  But it's possible things willchange due to someone else's interaction.
When things share during an SP session, let peopel know why.  

you might have a fullscreen experience where we hide the status bar.  But the pill in the status bar givesp eople access to controls.  In the apple TV app, a single tap will show the status bar again.  We recommend you do this.


There may be instances where someone navigates away from your app while in group activity.  Remember to support PIP.  `AVPictureInPictureController`.

# Best practices
* register your group activity => people can start SP From your app
* configure descriptive metadata
* accommodate later joiners
* provide context in your UI (why changing)
* make controls easy to access
* PiP

# Wrap up
* shareplay directly from your app without the need for ongoing FT call
* Do the portal exercise
* Design for groups (not individuals)
* Consider best practices

* https://developer.apple.com/forums/tags/wwdc2022-10139
* https://developer.apple.com/forums/create/question?&tag1=91030&tag2=448030
* https://developer.apple.com/design/human-interface-guidelines/shareplay/overview/
* https://developer.apple.com/shareplay/

