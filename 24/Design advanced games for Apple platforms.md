Learn how to adapt your high-end game so it feels at home on Mac, iPad, and iPhone. We'll go over how to make your game look stunning on different displays, tailor your input and controls to be intuitive on each device, and take advantage of Apple technologies that deliver great player experiences.

[[Port advanced games to Apple platforms]]

Easier than ever to make games for Mac, iPad, and iPhone

# Design for the device

## Jump into gameplay

Additional download is probably not the launch experience you wanted.  **let people play as soon as installation finishes**.

Make sure the initial stages of your game are downloaded at launch.  Get around 15 minutes of play bundled into appstore download.  From there, start loading the next chapters of your game in the bg.  If they do progress far enough to cache miss, that's when to show the UI.

Put download progress somewhere in your UI like a level selection screen.  Consider allowing players to revisit earlier chapters or levels as they wait.  Take advantage of either On-Demand Resources (appstore hosted) or Background Assets (developer hosted).  See the website.

Aim for an installation experience that blends with gameplay and is invisible to the player.

## set great defaults

Set onboarding options automatically.  ex look for places to remove choices that aren't relevant.  Screen size or aspect ratio can just get from device.

Automatically detect whether a controller is connected and get it sprofile from GCF.  Match physical buttons to UI from launch.

When choosing defaults across platforms, consider adjusting initial perf settings to match device capabilities.  

Remember that players just want to pick up and go.  Look for opportunities to reduce granularity of settings compared to desktop.

## create flexible layouts

With a unified gaming platform, your game can ship across a variety of screen sizes.  Design an adaptive game interface that scales.  Save time designing across screens.

Ensure your layout reacts to different aspect ratio.  Device models can have different aspect ratios.  Scale your interface up/down to fit into other devices.  

As you move between device shapes, keep each section a consistent size/distance.  Keep controls at physically comfortable sizes for mobile players while making the best use of screen space across devices.  

## use the full screen

Safe areas help you avoid home indicator, dynamic island, etc.  On mac, the safe area is inset from the top to help you avoid rounded corners, and camera area.  

Ensures every part of your app is visible.

Try building your game to the simulator app within Xcode.  

Guides for positioning UI, but not the actual margins for your entire game.  Take advantage of every pixel for your game's environment.  If content gets clipped, try adjusting your game's camera.  To see if you can fit that content in.

With prerendered content, make use of letterboxing.  To make it still feel like a full screen experience, either fill in with custom game artwork, or tint it black to blend in with hardware bezels.

Only use letterboxing if you have to.  Your game will feel more immersive if it i makes use of the entire screen.

## make it legible

Especially important if you're bringing a console or PC game to devices with small displays.

iPhoen and iPad, aim for 17pt or higher.  For body text. 
For less essential information, try not to go lower than 11pt typesize.

Mac type sizes
Default: 13 pt
minimum: 10pt


Default tap target size on iPad/Iphone, 44 pt.
For less critical UI, can push it down to 28pt.  But keep in mind that buttons at this size are hard to select accurately.  Especially if they have attention on gameplay.

mac: hit area should be 28x28 pts.
minimum: 20pt.

Aim for the defaults rather than the minimum.  

Take advantage of scroll views to show more UI!

If you're in doubt, test it on a real device.  Try on as many devices as you can.  
# Design for input

## Understand input methods

We support a wide range of controllers.  Use GCF to get keymaps, etc.

Consider that modifier keys are arranged in a different order than PC keyboards.  Validate that your control mappings are comfortable.

Apple is especially unique in our support for high-end games on touch-first platforms.  While some players may play your game on an iPad or controller, most players won't have a controller available.  To maximize the reach of your game, go the extra mile by designing custom touch controls.

Touch is different than controller.  On touch screens, your input surface is also the output.  ex, controllers cluttering the screen.  

## Adapt movement and camera
For a first or 3rd person game, create a virtual replacement for left thumbstick control.  Dynamic nature of touchscreen means I can hide when not in use.

while thumbsticks traditionally have fixed physical dimensino, you're not bound by those constraints.  Since players can't feel where their hand is, expand the input area to be as broad as possible.

For camera, I could use another virtual thumbstick.  Not the best use of the touch screen.  Use direct touch input to pan the camera.  As I drag my finger, I'm directly manipulating the camera position.  Very past and precise movement, more like a mouse input.

Input area should be as broad as possible.  In a game with an overhead or isometric perspective.  Consider letting a player tap to move.

While camera panning transfers well, consider adding inertia for large distances.

Two-finger pinch to zoom in/out.

## place controls intuitively

Be mindful of safe area insets.  Avoid covering my character.  

This leaves the regions around the thumbs.  And top of screen.  

Consider which controls might need to be used at the same time to place on appropriate side of the screen.

ex in controllers we use l2 for aim, where r2 is an ability.  Doesn't translate well to touch, can't aim/move at the same time, or aim with r2.

If we swap controls, it works better.




## respond to gameplay

Action-oriented control glyphs.
When a control changes its behavior based on context, update the symbol.

When an action isn't available, remove it entirely to avoid cluttering the screen.

Reflect details about game state.  here, I'm using the fire button to visually represent when ability is available for use again.  Hide controls in menus, etc.
## provide feedback

Because touchscreen doesn't have tactile feedback, provide feedback in other ways.  Any touch contorl you create should have a press state.  Confidence that they pressed the button, without them controls feel unresponsive.  

Consider e.g. a glow outside the bounds of the control.  Theme into the visual style of your game.

Sound and haptics are another great form of feedback and go beyond simple button press.  Can provide haptics on touchdown/up. 

see hig for even more.

# Wrap up
* provide a great first launch experience
* look beautiful on every display
* consider each input type
* Design great touch controls
[[Port advanced games to Apple platforms]]

# Resources
* https://developer.apple.com/games/game-porting-toolkit
* https://developer.apple.com/design/human-interface-guidelines/designing-for-games
* https://developer.apple.com/design/human-interface-guidelines/game-controls
