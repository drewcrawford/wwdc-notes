# Code-along setup
developer.apple.com/documentation/gamekit/adding_recurring_leaderboards_to_your_game

1.  Signing can capabilities => team
2.  Add an app record to ASC.
# Leaderboards recap
## Classic leaderboards
* Always active, do not end
* Maintain all-time, weekly, and daily rankings
* Useful for cumulative scores
	* Total XP earned
	* Current number of coins

## Recurring
* Short-lived leaderboards that repeat
* Keeps history of prevoius leaderboards
* Useful for periodic, timed events
	* One-hour challenge every sunday at noon
	* A new leaderboard every week


# Sample game walkthrough
* TitleScreenViewController => title screen.  Presents GameViewControler when play is tapped
* GameViewController => SKScene.  
	* ScoreNode
	* CountdownNode
	* setupBoatNodeWithActions
		* fadein/out boat sprites
	* touchesEnded
		* remove boats
	* timeIsUp
		* Nodes are removed and final score displayed
* Daily leaderboard
* Live rankings
* Changing over time

# ASC configuration
Features => Game Center.  Leaderboards, recurring leaderboards.

* reference name => used in ASC
* ID => used within app
* score format type => in gamecenter UI
* Sort order => High to low, etc.
* Score range => reject scores outside this range.
* Start date and time - first occurence of the leaderboard, in UTC.  
* Duration - length of time
* Restarts every.  >= duration so that recurrences may not overlap


# Using leaderboard APIs
## Enabling authentication
Uncomment the body.  Sets authentication handler on the local player.
## Submit scores
Two submitScore methods.  Class and instance.  When you submit to class, it will go to wahtever occurrence is active.  Will fail if no active occurence.

For instance method, load an instance of the current occurrence.  

Context can encode game specific information.  

## Show leaderboards
For recurring leaderboards, timeScope must be `.allTime`.

Add leaderboard node as a member of GameScene.  Convert GK data to your own struct type for use in your own situation.

Keep leaderboard node in sync.  Call `updateLeaderboard` node in completon handler for `submitScore`.

## Progress (difference between last leaderboard)
Using async will make more readable.

# Wrap up
[[Tap into Game Center Leaderboards, Achievements, and Multiplayer]]
[[What's new in Game Center Widgets, friends, and multiplayer improvements]]

* https://developer.apple.com/documentation/gamekit/creating_recurring_leaderboards
* https://developer.apple.com/design/human-interface-guidelines/game-center/overview/introduction/

