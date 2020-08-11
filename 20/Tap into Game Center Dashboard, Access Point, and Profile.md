#gamecenter

# What is game center?
* social gaming network
* single player identity
* instantly play with GC friends
* Leaderboards, achievements, and challenges
* Multiplayer functionality

GC "access point" once highlights finish displaying.  Meant to remain on title screen as a persistent way to get to GC functionality.
API lets you choose what corner to display this in
Can link to a paritcular section, but the dashboard overview is new.

Player profile.  Friends, achievements, games, etc.
[[Design for game center]]

Friends-only leaderboard is now the default.  Global/recent still availabile.
Player cards are accessible in many new places.  

# Native experience – multiplayer
Colors underneath show through, meant to make the design feel at home in any game.
Add additional players for multiplayer

# New
## app store
Now, when browsing popular areas of the store, players can better discover what friends are playing.
Also integrated into product page.

?

# Dashboard
* Consistent, one-stop UI for GC features
* Access player profile, leaderboards, achievements, challenges
* Provided via `GKGameCenterViewController`


```swift
//presenting the main dashboard
// GKGameCenterViewController

public init(state:)
[...]

// Example: Display Main Dashboard
//specify `.dashboard` state.

let vc = GKGameCenterViewController(state: .dashboard)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil) 

[...]
enum GKGameCenterViewControllerState : Int {
   case `default`
   case leaderboards
   case achievements
   case challenges
   case localPlayerProfile
   case dashboard
}
```

could also display a specific state with a different `state:` argument.


```swift
// Display scores for a specific leaderboard
let vc = GKGameCenterViewController(
                leaderboardID: "grp.xyz.laketahoe",
                playerScope: .global,
                timeScope: .allTime)
vc.gameCenterDelegate = self
present(vc, animated: true, completion: nil)
```

# access point
* entry point to in-game dashboard
* visible in corner of your choice
* player icon, optional highlights
* number of achievements earned
* rank on default leaderboard

## configuration
By defualt, AP is placed in top-leading.  However, it's flexible, so you can place in any corner.
Positioning will depend on language settings.
Highlights -> just avatar, or achievements, rank

```swift
// Configure and show Access Point
func showMainMenu() {
    // Call your code to setup the main menu
    self.setupMainMenu()

    // Place access point on top left   
    GKAccessPoint.shared.location = .topLeading    

    // Show highlights
    GKAccessPoint.shared.showHighlights = true
 
    // Show it!
    GKAccessPoint.shared.isActive = true
}
```


## activation
showing/hiding access point
show
* in main menu

hide
* cinematic intros
* during gameplay
* settings screen

[[Design for game center]]
## observable properties
### `isPresentingGameCenter`
Use to pause
* introduction movie
* gameplay
```swift
let observation = GKAccessPoint.shared.observe(
           \.isPresentingGameCenter
    ) { [weak self] _,_ in
    self.paused = GKAccessPoint.shared.isPresentingGameCenter
}
```

### `frameInScreenCoordinates`
```swift
// Observable properties
// frameInScreenCoordinates

let observation = GKAccessPoint.shared.observe(
           \.frameInScreenCoordinates
    ) { [weak self] _,_ in
    let screenFrame = GKAccessPoint.shared.frameInScreenCoordinates
    let accessPointFrame = myView.convert(screenFrame, from: nil)
    // adjust your layout
}
```

## apple tv and game controllers

May want to use this property to draw custom focus around the access point, if your game draws that for other controls on the title screen.

```swift
// Apple TV and controllers

// track and update focus
func trackController(position: CGPoint) {
  let screenFrame = GKAccessPoint.shared.frameInScreenCoordinates
  let accessFrame = myView.convert(screenFrame, from: nil)
  // if the point is in the access point turn on feedback
  accessPointElement.focusFeedback = CGRectContainsPoint(accessFrame, position)
}
// Apple TV and controllers

// Handle selection
func accessPointSelected() {
  GKAccessPoint.shared.triggerAccessPoint {}
}
```




# players and friends
Players represent users within game center
* scores
* achievements
* leaderboard
* multiplayer gaming
consistency across game center games
single account, single signin

|                          | `GKLocalPlayer` | `GKPlayer`        |
|--------------------------|-----------------|-------------------|
| Avatar                   | ✔️               | ✔️                 |
| Nickname                 | ✔️               | ✔️                 |
| Scoped playerID          | ✅ (persistent)  | ✅ (session-based) |
| Player game restrictions | ✔️               | no                |
| Saved game state         | ✔️               | no                |

## getting started
1.  Need to enable GC as a capability in xcode.  Also in ASC.
2.  Authenticate the local player.  
3.  Receive notifications: game invites, challenge invites, user changes

### authentication
* as early as possible

happy path

```swift
// Local player restrictions

GKLocalPlayer.local.authenticateHandler = { viewController, error in
    let isGameCenterReady = (viewController == nil) && (error == nil)

    if isGameCenterReady {
        //enable GC features 
    }
}
```

Error path.  Note that we don't actually use the error, although I think it's available
Just present the VC.
The authentication handler will be called again when the user resolves the issue.
```swift
// Local player restrictions

GKLocalPlayer.local.authenticateHandler = { viewController, error in
    if let viewController = viewController {
		present(viewController, animated: true)
	}
}
```

Now let's say the user doesn't sign in, the authentication handler will be called yet again, this time with an error.  Disable GC and start your game.

## player profiles
Can now present inside your game.
Players can see achievements, find friends, etc.
```swift
// Local player profile

let profileVC = GKGameCenterViewController(state: .localPlayerProfile)
profileVC.gameCenterDelegate = self

present(profileVC, animated: true, completion: nil)
```

## restrictions
There are cases where the local user may have various restrictions on their account.
Your app has to check these restrictions
```swift
// Local player restrictions

GKLocalPlayer.local.authenticateHandler = { viewController, error in
    let isGameCenterReady = (viewController == nil) && (error == nil)

    if isGameCenterReady {
        if GKLocalPlayer.local.isUnderage {
            // Hide explicit game content
        }

        if GKLocalPlayer.local.isMultiplayerGamingRestricted {
            // Disable multiplayer game features
        } 
		//new in iOS 14, user cannot use voice or multiplayer features
		//will always be true if the user is underage
        if GKLocalPlayer.local.isPersonalizedCommunicationRestricted {
            // Disable in game communication UI
        }    
    }
}
```

tvOS user management
* support for multiple GC accounts
* transparent to your application
* easy opt-in
* [[Support multiple users in your tvOS app]]

## friends
* leaderboards
* multiplayer game invites
* achievement challenges
* friend profiles -> see what friends are playing etc

### privacy
* everyone
* friends only
* no one

In pt 2, we will cover

[[Tap into Game Center: Leaderboards, Achievements, and Multiplayer]]
# leaderboards
# achievements
# multiplayer
