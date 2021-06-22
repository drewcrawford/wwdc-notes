#gamecenter 

* Leaderboards
* Achievements
* Multiplayer
* Friends and player profiles
* Game discoverability

Not only can I read more about the game, I can also see a complete list of friends playing this game.

See more of the games they've been playing.

Automatically appear on the product apge for all GC-enabled games.

Great opportunities for players to find new games to play.  Nothing you have to do to make your game eligible.

Adding 2 new widgets:
* Friends are playing
* Continue Playing => resurfaces games you've played before

No additional code needed for feature to adopt.  

# Friends and player profiles

## Friends
Friend requests continued to be sent through messages.  But a brand new inbox as well.

Flow within messages improvements.  macOS.

## API
Privacy-friendly access to a player's friends so you can create unique experiences.

# In-game
Last year, we had a dashboard and refreshed UI.

Use a combination of native GC UI and custom UI that fits your game.

Access point (button).  More ways to keep up with GC rankings etc.

# Multiplayer

Suggestions based on message groups.

Multiplayer API called "fast start".

Controller support for GC UI.  No new API to adopt required.

You will need to add controller support for your own UI.  See [[Tap into virtual and physical game controllers]] to learn more.

# Friends
New APIs to leverage your players' friends list.  

In-game "add friends" button.  We recommend using SFSymbols.  When the user taps on the button, call the API to allow your player to add friends to the game.

```swift
// Call Friend Requests API to present friend request view from a view controller, when player click on Add Friends Button in your game
let error = GKLocalPlayer.local
.presentFriendRequestCreatorFromViewController(using: navigationController)

if error != nil {
    print("Fail to send friend request with error: \(error!.localizedDescription).")
}
```

## API

* Privacyfriendly access to friends list
* Player must approve access
* Only bidirectional friends returned.  Only friends who have also granted access to the game
* Permission synced across devices

"show what level my friends are on"
"launch with my friends"
"show friends on recurring leaderboard"
etc

1.  Call API to ask for permissions ot access friends
2.  Player gets prompt (info plist text as usual)
3.  Return the list of friends

e.g. show player location in your UI

1.  First, edit your Info.plist `NSGKFriendListUsageDescription`

May want to call this in `didFinishLaunching` and delay loading until there is a good moment to prompt:

```swift
// Checking authorization
GKLocalPlayer.local.loadFriendsAuthorizationStatus { (authorizationStatus, error) in
    guard error == nil else {
       // Error handling
       print(“Fail to load friends list with error: \(error!.localizedDescription).”)
       return
    }

    // Handle GKFriendsAuthorizationStatus
    switch authorizationStatus {
      case .notDetermined:
         // Player have not made a choice on friends list sharing
      case .denied:
         // Player have denied your request to access their friends list
      case .restricted:
         // You should delete collected player data from your end
      case .authorized:
         // Player have authorized your request to access their friends list
    }
}
```

If denied or restricted, you should delete the player data.  If you choose to grant access, will be able to update friends list.

```swift
func loadFriendsOnProgressionMap() async {
    do {
        let friends = try await GKLocalPlayer.local.loadFriends()
        if friends.count > 0 {
            let leaderboards = try await GKLeaderboard.loadLeaderboards(IDs: [“progress"])
            if let leaderboard = leaderboards.first {
                let entries = try await leaderboard.loadEntries(for: friends, timeScope: .allTime)
               for entry in entries.1 {
                    let avatar = try await entry.player.loadPhoto(for: .normal)
                    let name = entry.player.displayName
                    let friendLevel = entry.score
                    // Display player on progression map
                }
            }
        }
    } catch {
        print("Error: \(error.localizedDescription).")
    }
}
```

Here we use `async`.

# Multiplayer

Many different ways for players to find people to connect/play with.  

* Auto-matching
* Game center friends
* Contacts
* Nearby players
* Phone number / email
* GC groups
* message groups

Dramatically reduces efforts to show different people from different sources.  Auto-matching is the mechanism.

* Suggestions shelf, heterogeneous collection
	* Game Center Group – group of players you recently played with in real-time or turnb-ased matches
		* easier for players to play repeatedly with a certain group of people
	* Message group
* Send button
* Adding/removing players after initial invitation, if they have not responded
* Individually remove with X.  As long as they have not accepted or ?
* Players can invite more players to the game
* Greatly simplifies/streamlines experience

Increase game's discoverability.  Leveraging GC multiplayer UI you get these features for free.

## Fast start
* New API
* Get players in match even faster
* Players can still join after the game starts
* Auto-matching continues in the background

Players will be able to join your game.  If your game supports players joining at different times...

### Ex

1.  GKMatchMakerViewController
2.  Game starts when 2 players are present
3.  Invite additional players
4.  Start game
5.  Gameplay begins as soon as VC is dismissed
6.  GC is still connecting in the background
7.  Game notified of 3rd player
8.  GC is still automatching in the bg.  After matchmaking is finished, two other players join.
9.  Now we have a full set of 5 players.

Byadopting fast start, different players will be able to join the same game session at different times.

```swift
// Set canStartWithMinimumPlayers to true to enable Fast Start mode
let request = GKMatchRequest()
request.minPlayers = 2
request.maxPlayers = 6
request.playerGroup = 2021

let vc = GKMatchmakerViewController(matchRequest: request)
vc.canStartWithMinimumPlayers = true
vc.delegate = self
self.present(vc, animated: true, completion: nil)
```

```swift
// On the invitee side, present GKMatchmakerViewController with the invite
func player(_ player: GKPlayer, didAccept invite: GKInvite) {
    if let vc = GKMatchmakerViewController(invite: invite) {
        vc.matchmakerDelegate = self
        self.present(vc, animated: true, completion: nil)
    }
}
```

# Wrap up
* Enhanced discoverability
* Friends
* Multiplayer

[[Bring recurring leaderboards to your game]]
[[Tap into virtual and physical game controllers]]


