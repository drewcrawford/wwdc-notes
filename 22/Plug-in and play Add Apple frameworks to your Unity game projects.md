Add new features to your unity-based games.
Many of you are using tools such as Unity.  Our goal is to bring you the latest features no matter what tool your'e using.  Starting today, enhance your games with a new set of unity plugins.
* Game center
* Game controller
* Accessibility
* core haptics
* PHASE
* apple core - build settings, bulid process

Add new gameplay mechanics, etc.

# Design principles
* Map to single framework
* Expose the framework as C# scripts
* By learning the plugin you'll be implicitly learning hte underlying framework
* Exist as a native plugin

Each plug-in is a Unity package
Extend Editor functionality where applicable
Include documentation, references, and samples
https://github.com/apple/unityplugins

Once you've cloned the repository, build the plugins.  Build script: `build.py`
Builds, packs, and cleans

Simplest invocation for most common workflow
`python3 build.py`
Requries: xcode, python3, npm, and Unity.

Build script documentation in the repository.



# Project organization
# Project workflow
# Plug-in deep dive

## Apple.Core
Common build settings UI
Native library management
Build post-process
Native code runtime inter-op utilities
Dependency for all Apple Unity plugins

Demo.
The editor will then load the package and compile the scripts.  Editor => Project Settings...
When you improt an apple unity plugin, all its available build options will be available here.

Set deployment target
Disable post-process build step for any plugin.
Configure common security settings to propogate to intermediate xcode projects.

## Game Center
* Player authentication
achievements
Leaderboards
Challenges
Matchmaking

Game Center home screen widget
app store widget

1.  Add plugins
2. GKLocalPlayer object.
3. Query for player restrictions.  
	4. Limiting acces to adult or explicit content
	5. isMultiplayerGamingRestricted
	6. isPersonalizedCommunicationRestricted

Game manager component.
```c#
using Apple.GameKit;

public class GameManager : MonoBehaviour
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

Once the local palyer has authenticated, check their player restrictions.  Series of boolean checks.

```c#
try
{
    _localPlayer = await GKLocalPlayer.Authenticate();

    if (_localPlayer.IsUnderage)
    {
        // Hide explicit game content.
    }

    if (_localPlayer.IsMultiplayerGamingRestricted)
    {
        // Disable multiplayer game features.
    }

    if (_localPlayer.IsPersonalizedCommunicationRestricted)
    {
        // Disable in-game communication UI.
    }
}
```

Additional steps.
1.  Add game center capbility to your intermediate xcode project
2. Add game center features in ASC.  See portal for more information.

Only scratches the surface in gamecenter.  To learn more about improving discoverabilty in your game

[[Reach new players with Game Center dashboard]]
[[What's new in Game Center Widgets, friends, and multiplayer improvements]]


## Game Controller
Handful of features.
* game controller customizations
* Button glyphs
* Support for MFi, Sony, and Microsoft controllers

1.  Initialialize GCControllerService.
2. Query GCControllerService for connected controllers
3. Represented as GCController objects
4. Poll for updated state.

Test for input on each controller, such as buttons, thumb sticks, etc.
Callbacks to handle connect/disconnect.

Create a simple input menu component.
1.  GCController collection
2. Start method for initialization
3. Update method for handling polling, testing for inputs.

```c#
using Apple.GameController;

public class InputManager : MonoBehaviour
{
    void Start()
    {
        // Initialize the Game Controller service
        GCControllerService.Initialize();

        // Check for connected controllers
        var controllers = GCControllerService.GetConnectedControllers();
        foreach (GCController controller in controllers)
        {
            // Handle controllers
        }

        // Set up callbacks to handle connected/disconnected controllers
        GCControllerService.ControllerConnected    += _onControllerConnected;
        GCControllerService.ControllerDisconnected += _onControllerDisconnected;
    }
}
```

Update controller state, repsond to state.

```c#
foreach (GCController controller in _myConnectedControllers)
{
    controller.Poll();

    // Check the 'South' button ('A' button on most controllers)
    if (controller.GetButton(GCControllerInputName.ButtonSouth))
    {
        //Handle button pressed
    }

    // Check other controller inputs…
}
```

### Resources
[[Advancements in Game Controllers]]
[[Supporting new game controllers - 19]]

## Accessibility
making technologies available for everyone.  Use accessibility plugin to integrae a wide range of assistive technologies.

Ability to add 
* voiceover
* Switch Control
* Dynamic Type
* UI accommodations
* Learn more about the accessibility plugin
[[Add accessibility to your Unity games]]

## Core Haptics
Increase immersion and enhance the gameplay experience.  Use corehaptics to build custom haptic patterns.
Tightly synchronize audio content
Program haptic patterns
Define file-based haptic patterns
Tune with in-Editor pattern editing

4 fundamental elements
* CHHapticEngine
* CHHapticPatternPlayer - start, stop, pause, resume
* CHHapticPattern - logical grouping of events
* CHHapticEvents - building blocks

Data-driven API which allows programmatic deifnitions, or by leveraging AHAP files.

Haptics component.

Assets=>create=>apple=>corehaptics=>AHAP.
Core haptics comes with editor extensions that let me tune my pattern from unity.

By default, there's a transient event, but I can add a continuous event as well.  Import, Export, Reset.

With the pattern defined, all that remains is to add some logic to the haptics component that can play a haptic pattern.
1.  PrepareHaptics - haptic engine is initialized, player is created
2. Play => call the player's start method

```c#
using Apple.CoreHaptics;

public class Haptics : MonoBehaviour
{
    private CHHapticEngine _hapticEngine;
    private CHHapticPatternPlayer _hapticPlayer;
    [SerializeField] private AHAPAsset _hapticAsset;

    private void PrepareHaptics()
    {
        _hapticEngine = new CHHapticEngine();
        _hapticEngine.Start();
        _hapticPlayer = _hapticEngine.MakePlayer(_hapticAsset.GetPattern());
    }

    private void Play()
    {
        _hapticPlayer.Start();
    }
}
```

Add a serialized field attribute to allow the AHAPAsset to be set in the UI.
CoreHaptics unity plugin gives you the tools you need to add more immersion to your games.  Create magical game moments that look, sound, and feel real.

[[Introducing core haptics - 19]]
[[Designing Audio-Haptic Experiences - 19]]
[[Practice audio haptic design]]

## PHASE
Lush soundscapes into your game worlds.
Provide complex, dynamic audio experiences to your games.
* Geometry-aware audio
* Reverberation and reflections for environmental effects
* Complex runtime sound events

Attach them to your game objects, etc.  
1.  PHASEListener => ears of your game, and processes audio based on transform and reverb preset.
2. PHASEOccluder => Transform, mesh filter.  Dampen audio when they come between sources and the listener.
3. PHASESource => Use the object's transform to make sounds in the world.  SoundEvent => describe audio playback events.

Add yer plugins

Create a sound event.  Assets=>create=>apple=>PHASE=>Sound Event.
Phase plugin will imeediately open the copmoser window.
Popup allows me to add a node to the event.  Sampler node.
Route it to a mixer.

Audio in the scene is routed and configured for playback.

Learn more about PHASE at apple developer site
[[Discover geometry-aware audio with the Physical Audio Spatialization Engine (PHASE)]]

# Wrap up
* Explore the repository
* Experiment with the plug-in samples
[[Add accessibility to your Unity games]]
[[Reach new players with Game Center dashboard]]

https://github.com/apple/unityplugins
https://developer.apple.com/documentation/corehaptics/delivering_rich_app_experiences_with_haptics
https://developer.apple.com/documentation/swiftui/view-accessibility
https://developer.apple.com/documentation/phase
https://developer.apple.com/design/human-interface-guidelines/game-center/overview/introduction/
https://developer.apple.com/documentation/gamecontroller

