Explore how Game Center, Apple's social gaming network, helps players discover and engage with your game. Learn about Game Center and App Store features that can help you connect with new players and keep them coming back, as well as Apple technologies designed to deliver powerful gameplay experiences.

# Game Center overview

Apple's social gaming network.  Designed to help players experience the joy of games with their friends.  With GC, players can see what games their friends are playing.

Every player has a profile that they can customize with a nickname and avatar.  We've made it easy for players to connect and play together.  Friend suggestions, add friends to quickly invite to games.  Prepare scores, and see what games they're playing.

Activity - brings together fun things a player and their friends are doing.

We also have a widget / cc thing.

 see what your friends are playing from the appstore.  Giving players more ways to discover and download your game.
# Getting started

1.  Enable GC in ASC.
2. GC capability in xcode
3. APIs

By just enabling GC, your game will automatically appear across all places we mentioned.  Helping to boost discovery of your game.
# Key features

## Identity

GC lets players build a consistent identity across games they're playing.  Starts with their profile and allows them to share their gaming activity with their friends.  Unlock all this functionality and allow for greater discovery of your game.

Option to build more personalized experiences on top of gc identity in a few ways.

Access Point.  Displays a GC avatar.  Easily access any other GC features you support, like viewing leaderboards or achievements.

Player avatar
access to nicknames and avatars.  Integrate directly into your game.   Speedup onboarding experience.

Save a player's game state so their progress will be synced to all devices, letting them pick up where left off.
## achievements

Special milestones that let players know when they've reached a particular goal.  Share progress with friends.  Great for motivating players to keep going and more deeply engage with your game.  Great for completionists who want to collect 'em all.

* standard
* progressive -> have progress
* hidden

See progress toward unlocking them, and we'll let them know when they're close.

Give them a celebratory moment by displaying a notification within your game.  GK to display achievement banner.  Players can see activity appear across device.  Dashboard, GC widget, app store.  Any time a player unlocks a new achievement, friends will see activity and get inspired to see your game as well.

Add from ASC.  
## leaderboards

For various levels or modes.  Ex we created 6 leaderboards, one per map.
Show how many points a player has earned throughout their gameplay, how they rank among friends.  See when a friend passes them, as well as how many points they need to beat a score.

GC stores and displays scores in GC interface.  Or, you can use GK to send/store data and display leaderboards within your own custom interface.

When you send score data, players will see leaderboard activity appear across device.  Such as dashboard, GC widgets, and app store.  Can help players feel challenged, etc.

Add in ASC.  Services, Leaderboards.  An exciting way for players to challenge themselves and friends.
## multiplayer

* real-time and turn-based
* friend invites
* auto matching
* shareplay support.  Start from group facetime call or iMessage group.  Great way to get more players introduced to your game.  Works without requiring any additional development.

see docs.
# Going further
## In-app events.
across the app store, including on your product page, in search/recommendations, and in editorial curations such as the today tab.

Reach new players, keep current players engaged, and reconnect with lapsed players.

* set up in ASC
* create up to 10 apps at a time
* Publish 5 on the app store
* Measure event performance with app analytics
see docs.

## pre-orders

Product page will show an expected release date, building additional anticipation.  People can also preorder your game from search results and today/games tabs, if your game has been featured.

set up in ASC
Download release date can be up to 180 days in the future
Use preorder badge in marketing efforts
auto-download within 24 hours of release
We'll also notify customers.

## Game technologies
* controller support
* metal
* unity plugins
Even more engaging and immersive gameplay experience.  Controller support will display on product page.
Metal.  Full potential of apple silicon.
Connect your game to gamecenter, corehaptics, game controller frameworks, and more.

Check out apple technologies for game developers page.

# Resources

* https://developer.apple.com/help/app-store-connect/configure-game-center/overview-of-game-center
* https://developer.apple.com/documentation/gamekit/enabling_and_configuring_game_center
* https://developer.apple.com/game-center/
* https://developer.apple.com/design/human-interface-guidelines/technologies/game-center/introduction
