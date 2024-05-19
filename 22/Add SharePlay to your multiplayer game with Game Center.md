Learn how to let your players jump into games with friends they're on FaceTime calls with, using SharePlay. We'll show you how easy it is to turn on SharePlay support if you are already using the Game Center multiplayer UI. And if you've built a custom interface, we'll give you the few lines of code you need to support SharePlay.

* Friends
* leaderboards
* achievements
* player activity

multiplayer
* invitation
* matchmaking
* real-time gameplay
* voicechat

# Shareplay
If your players are on a ft call or messages group, immediately jump into a game together.  
Simply start an audio call from the groupchat.  Friends will receive invitation as a ft call.  Launch game.

Click play now, select multiplayer.  Play with friends.  
You will get sp integration with one small update.  I'll show you how to enable.
Currently, if you present your GKMatchMaker vc, players will see mode selection screen by default.  To adopt integration, first stop is to add entitlement for group activities.  xcode, signing & capabilities.

Mode selection will now show shareplay.  

* add the group activities capability
* call gamekit's shareplay apis
* start group activity
* put players in the recipients list
* find a match

# Wrap up
People can play with friends using shareplay
game center UI: simply enable group activities capability
custom ui: new APIs in GKMatchmaker

[[Tap into Game Center Leaderboards, Achievements, and Multiplayer]]

