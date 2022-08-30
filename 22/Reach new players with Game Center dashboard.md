Players set upa  profile and connect with their friends.

* Identity
* Leaderboards
* Achievements
* Multiplayer

* accesspoint
* achievements/leaderboards
* friends API
* multiplayer experience
* activity to more places including surfaces what you and friends are playing in the app store, and new widgetsl ike continue playing, etc.

We know a lot of game developers are leveraging Unity.  Unity plugin for GameKit.  Entire gamekit API.  Don't ahve to choose between building your game iwth unity and taking full oadvantage of the first class gamecenter features.

You will see examples in swift **and** C#.

# Activity
Ex, when they get a new achievement, or jump up a leaderboard.  We've redesigned the GC dashboard so that it will now include activuity from a player's friends in your game, in one place.

Players will see recent activity in your game, achievements earned, etc.  

## Getting started
Your games already appear in Activity if you use GC.

* Enable GC in xcode
	* signin and capabilities
* Authenticate the local player

```swift
// Authenticate the local player
import GameKit

class TitleScreenViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Authenticate the local player
        GKLocalPlayer.local.authenticateHandler = { viewController, error in
            if let viewController = viewController {
                // Present the view controller from Game Center.
                return
            }
        }
    }
}
```

Place this as early as possible.  Even on the title screen.

For unity developers,

```c#
// Authenticate the local player
using Apple.GameKit;

public class MyGameManager : MonoBehaviour
{
    private GKLocalPlayer _localPlayer;

    private async Task Start()
    {
        try
        {
            _localPlayer = await GKLocalPlayer.Authenticate();
        }
        catch (Exception exception)
        {
            // Handle exception...
        }
    }
}
```

Players will see this welcome banner when they launch your game.  Activity will appear in player feed.

Now provide easy access to the dashboard.  Best way is through GC access point.
* convenient way to launch the GC dashboard

```swift
// Show the Access Point
import GameKit

class MenuScreenViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        GKAccessPoint.shared.location = .topLeading
        GKAccessPoint.shared.isActive = true
    }
}
```

Consider waht makes the most sense, ideally on your menu page.  Set a location for its appearance.  Then set `isActive` to `true`.  Now your AP will appear.

Unity:

```swift
// Show the Access Point
GKAccessPoint.Shared.Location = 
    GKAcessPoint.GKAccessPointLocation.TopLeading;

GKAccessPoint.Shared.IsActive = true;
```

When players interact with the AP, system brings up the dashboard.  Provides a familiar place for players to learna bout your game and check in on recent activity.

# Leaderboards

Leaderboards are a powerufl way to increase your game's expsure in activity.  Gives playeers more reason to jump back into your game.

After setting up a leaderboard, players see new activity.  here my friend placed in the top 25% of a leaderboard.
Activity highlights when ap layer beats recent score.  For this activity, players will get a notification.  Sent from GC, you don't need to worry about asking the user to opt into notifications for your game.  If your game provides leaderboards, these activities will appear automatically.

Consider expanding leaderboard sets to provide mroe moments of ocpmetition for players and their friends.  recurring leaderboards provide timeliness and a reason to re-engage on an ongoing basis.

# Achievements
How achievements are featured in activity.  Reflected in activity.  When player completes every achievement, we recognize that with special celebratory activity.  Sense of progress and accomplishment, tella  story of how far a player has made it in your game.

Wider visibility throughout GC social network.  Players have more reasont ojump into you rgame and play together.

See all recent activity in one place.  What you see or don't see is based on profile privacy option in GC settings.

# Wrap up
Increased reach and distribution with Activity
Tighter integration with Unity

[[Tap into Game Center Leaderboards, Achievements, and Multiplayer]]
[[Bring Recurring Leaderboards to your game]]
[[Plug-in and play Add Apple frameworks to your Unity game projects]]



* https://developer.apple.com/documentation/gamekit/gklocalplayer
* https://developer.apple.com/design/human-interface-guidelines/game-center/overview/introduction/