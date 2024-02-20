Discover how you can integrate CarPlay into modern vehicle systems. We'll show you how to adjust CarPlay for any high-resolution display — regardless of configuration or size. Learn how you can use CarPlay-supplied metadata and video streams to show information on additional displays, and find out how advances in wireless connectivity, audio, and video encoding can help prepare your vehicle system for the next generation of CarPlay.

How your vehicles can deliver the best CP experience.

#carplay 


# Visual integration
Modern vehicles feature large main dispaly with increasing varieties of shapes/layouts.  Wide, tall.  Nonrectangular.  Mix vehicle UI.

View area defines the boundaries where CP draws its UI.  Great in fullscreen, however...

support dynamic screen resizing.  If your system displays a widget next to CP, we may shrink the area.  You don't need to inset the view area.  Simply give it as much space as possible and CP will create the right amount of padding.

Nonrectangular?  View area is smallest rectangle enclosing the whole display
safe area - inner rect

draws interactive content inside the safe area.  Outside the safe area, pixels are black.  Now you have the option to allow drawing outside the safe area, bg edge to edge.  `drawUIOutsideSafeArea`.  Only available for the main display.

status bar.
Always displayed
Vertical position on driver's side
horizontal psoition at bottom of CP UI.
If needed, use `viewAreaStatusBarEdge` to override position

Rounded corners drawn on black background
In windowed configurations, support corner clipping masks with `cornerMasks`
iPhone provides transparency mask
Cannot be used in combination with `drawUIOutsideSafeArea`.

## focus transfer
some systems support rotary knob or touchpad.  We may show focus highlight on selected element.  maybe you have your own focus highlight for builtin UI.

Transfer between CP elements and car system.  We arbitrate focus between CP and your system.  Support `focusTransfer` for windowed configurations.  

* coordinate ownership with CP
* give CarPlay focus with `accessoryGiveFocus`
* Acquire focus with `accessoryAcquireFocus`.

## CarPlay appearance modei
light/dark themes.  Synchronize cp's appearance with builtin UI appearance.  Appearance mode can change based on vehicicle state, time of day, etc.

Separate controsl for CarPlay UI and maps
`uiAppearanceMode` vs `mapAppearanceMode`.

Specify appearance for each display, in this case main and instrument cluster are in dark UI.Can inform carplay of difference appearance modes per-display.  

multiple displays.
* main display
	* main UI stream.  Built-in UIs can get iap2 route guidance, phone call, now playing, etc.
	* carplay navigation UI stream
* instrument cluster display
	* navigation UI streams.  
	* iap2 route guidance, phone call, now playing, etc.
* HUD
	* iap2 metadata, so people can view tbt, etc.


# Connectivity

Begins when driver pairs for the first time.

* out of band pairing with USB
* people expect wireless cp to support oob pairing over usb.  if your system supports digital car keys, we've got something new for you
* pair carplay when adding a car key.

Simplfiied connection flow
Start CarPlay with iap2 CarPlayAvailability and CarPlayStartSession
Supports WPA3 only networks

Initiate wireless CarPlay reconnections earlier with car keys

Design your system to handle wireless interference
* Detect interferers
* switch to a free channel or band
	* we recommend 5ghz
* suppress short link disconnects


# Audio

enhanced buffering.  Apps can support airplay enhanced audio buffering.

Support enhanced buffering in vehicle systems with `mainBuffered`.  
Apps stream audio faster than realtime
Improved responsiveness and playback

[[Tune up your AirPlay audio experience]]

Support mixing these audio output streams:
* main audio
* main buffered audio
* alternate audio
* auxiliary audio

enhanced siri.  People expect the familiar siri experience in carplay
support `enhancedSiri` for
* voice activation
* media continues during siri
* instant button activation
[[Advances in CarPlay systems]]
# Video encoding
HEVC now supported
* bandwidth efficient
* enables high resolutions
* continue to support h264 for compatibility

# EV routing

Plan trips with stops for charging in apple maps
Preferred charging networks
Real-time charging station availability
Provide vehicle characteristics
Support state of charge information
Implement SiriKit intents in your automaker app 'list cars' and 'get car power level status'
Send iap2 `VehicleInformationComponent` and `VehicleStatusUpdate`

prepare for next-gen carplay
* pair and connect with car keys
* simplified connection flow
* handle wireless interference
* enhanced buffering for audio
* enhanced siri
* hevc

# Next steps
* make the most of what carplay offers today!

# Resources
* https://developer.apple.com/carplay
