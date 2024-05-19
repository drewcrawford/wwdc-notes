Get to know the new heads-up display panel built to help you analyze graphics performance in real time. Metal Performance HUD displays key graphics statistics so you can monitor, log, and identify tough-to-spot performance problems.

When enabled, it will appear as an overlay in upper righthand corner.  

* show and log key performance statistics
* identify and provide evidence of issues
* complement to instruments and xcode

# Device and display details
* GPU, display.  Drawable resolution
* scaling state.  Direct or composited path.  Device refresh rate.  (also minimum if variable)
* realtime data.  First column is instantaneous.  all measured in MS.
* middle column, lowest values.
* last column, high values in last 1.5 second.  Unusual numbers highlighted in red.
* process and gpu memory.  
* Present intervals timeline, and gpu time spent.  
# Enable the HUD.

enable in xcode.  Under diagnostics, 'show graphics overview'.  'log' will log.
on iOS/tvOS, it's in developer settings.  hud will only show for your own apps.

on macOS, enable `MTL_HUD_ENABLED=1` or `MTL_HUD_LOGGING_ENABLED=1`.

Enable with NSUserDefaults.  On macOS, iOS, tvOS, set `MetalForceHudEnabled` to true.

On macos, set globally `defaults write -g MetalForceHudEnabled -bool yes`

`MetalHudEnabled` info plist.  

Can also do it programmatically in CAMetalLayer.developerHUDProperties

"mode":"default",
"logging":"default"

default -> show.  Clear -> hide.

# Understanding logging

once per second, we write data to system logs, summarizing the data it's collecting.  Launch console, filter for `metal-hud`.

Each like starts with `metal-HUD`.  first frame number, estimated frame misses, process memory usage (MB), frame present interval, GPU time in ms.  

# Wrap up

* visualize your game's performance
* easy to enable on macOS, iOS, or tvOS
* Capture detailed logs for your own analysis.
send us feedback