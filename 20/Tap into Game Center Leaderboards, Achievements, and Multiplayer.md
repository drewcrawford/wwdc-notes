#gamecenter 

# Leaderboards
* Ranks players based on scores
* Friend and global ranks
* Challenge other players
* Shared across platforms
* Score formats, score types, sort orders

## Traditional (classic) leaderboards
Always active, do not end

## Recurring leaderboards
Short-lived leaderboards that repeat periodicaly
Useful for periodic timed events
* a 15-minute competition every hour
* A 1-hour challenge every sunday at noon
* A new leaderboard every week

Players get a fresh start to compete for top spots

Initial start date
Frequency (time between starts)
Duration (between start and end)

Can introduce a delay between occurences
(getter) Current start date
(getter) Next start date

| Example                                  | Initial start date | Frequency | Duration |
|------------------------------------------|--------------------|-----------|----------|
| 15 min competition<br>repeats every hour | June 25 00:00 UTC  | 60 min    | 15 min   |
| 1 hour contest<br>Every sunday at noon   | June 28 12:00 UTC  | 7 days    | 60 min   |
| A new leaderboard<br>every 24 hours      | June 30 00:00 UTC  | 24 hours  | 24 hours |

Duration must not be greater than the frequency.

## submitting score to leaderboard

```swift
//classic leaderboard
// Use the class method to submit score to one or more leaderboards at once
GKLeaderboard.submitScore(self.points, context: 0, player: GKLocalPlayer.local,
    leaderboardIDs: ["my.leaderboard.id"]) { error in
}
```

```swift
//recurring leaderboard
// Use the class method to submit score to one or more leaderboards at once
GKLeaderboard.submitScore(self.points, context: 0, player: GKLocalPlayer.local,
    leaderboardIDs: ["my.recurring.leaderboard.id"]) { error in
}
```
score will be posted to active leaderboard if one exists

Specific instance:

```swift
// Submitting to a specific occurrence of a recurring leaderboard
GKLeaderboard.loadLeaderboards(IDs:["my.recurring.leaderboard.id"]) { (fetchedLBs, error) in
    if let lb = fetchedLBs?.first {
       lb.submitScore(self.points, context: 0, player: GKLocalPlayer.local) { error in
       }
    }
}
```

If the user is offline, the server may find out about leaderboards after the leaderboard has rotated.  So that may not work.

## displaying leaderboards
* builtin UI
* custom UI

builtin UI:

```swift
// Launching in-game UI

// Display a list of leaderboards
let vc = GKGameCenterViewController(
             state: .leaderboards)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil)


// Or directly display scores for a specific leaderboard
let vc = GKGameCenterViewController(
             leaderboardID: "YOUR_ASC_LEADERBOARD_ID",
             playerScope: .global,
             timeScope: .allTime)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil)
```

Access previous occurrence

```swift
// Accessing previous occurrence

// Load current occurrence of a recurring leaderboard
GKLeaderboard.loadLeaderboards(IDs:["my.recurring.leaderboard.id"]) { (fetchedLBs, error) in
    if let current = fetchedLBs?.first {
       // Load previous occurrence using the current occurrence
       current.loadPreviousOccurrence { (prevOccurrence, error) in
           // Do something with the previous occurrence
       }
    }
}
```

Occurrences to continue to exist up to 30 days from expiry
Current and one previous occurrence accessible to local player.  Seems it also keeps around some number of occurrences the user submitted scores to, but kinda unclear to me

## app store connect
Need to setup leaderboards here

Occurrences may not overlap
Localize.  

# Achievements
Collectible item indicating the player has successfuly reached a goal.
Fundamental part of GameCenter since its inception.

* Locked
* In-progress
* Completed
* Hidden

Some players will scan the achievements list as a guide.  In this sense, the list gives users an opportunity to tell users what to think about.

Consider using hidden achievements to surprise / delight.  e.g. an achievement for failing spectaucarly.  
Achievements now on all platforms in the dashboard.

[[Design for Game Center]]

## limits
* 100 achievements
* Each can award 100 points
* overall up to 1,000 points

Make sure to leave room for future achievements

## implementation

1.  Authenticate the local player.   [[Tap into Game Center Dashboard, Access Point, and Profile]]
2. Report progress to GameKit.  As long as the progress is 0, will mark as locked.

```swift
if let achievement = GKAchievement(identifier: identifier) {
    achievement.percentComplete = percentComplete
    GKAchievement.report([achievement]) { error in
        if let error = error {
            print("Error in reporting achievements: \(error)")
        }
    }
}
```
Finally, display achievement progress.

Integrate the new access point.  Achievements tab is front-and-center.  

```swift
// Showing the Game Center achievements page

let viewController = GKGameCenterViewController(state: .achievements)
viewController.gameCenterDelegate = self
present(viewController, animated: true)
```

## `GKAchievementDescription`
Identified with achievement ID.

Also has achievement's title, etc.  I guess you can use this to display in your own UI, or to access programmatically.

Complete, incomplete, and placeholder images.

## app store connect
Set up achievements here.

# Multiplayer gaming
## overview
* Real-time -> fighting action etc.
* turn-based -> blackjack / board games
* server hosted -> MOBA/battle royale

Ways to find players
* auto-match
* friends
* contacts
* recently played
* nearby

"Cross-platform"
## new streamlined UI
* fewer steps
* imporved player suggestions

### demo

## api review
* Create `GKMatchRequest`
* ...

Adopt `GKInviteEventListener`.

### turn-based matchmaking
follows a similar flow.  The important difference is you can start immediately, since it will always be your turn.

## new MatchMakerViewController options
`.restrictToAutomatch`
`.nearbyOnly`

## new restriction
* game invite custom message
* voice chat

```swift
// Check if personalized communication is restricted
if GKLocalPlayer.local.personalizedCommunicationRestricted {
    // Disable UI for Voice chat
}
```

developer.apple.com/gamecenter
[[Design for Game Center]]

