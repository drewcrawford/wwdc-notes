# Differences in UI
On iPad, code completion is at the bottom.

On Mac, we have a new expandable area for code completion with quickhelp.

Also displayed on iPad in  popover and over the code completion bar.
# Customizing content for each platform
`supportedDevices`
`requiredCapabilities`

`SupportedDevices` is an array in the manifest.plist and can be "iPad" and/or "Mac".
Put this in the `feed.json` as well, but as `supportedDevices` case.


`requiredCapabilities` can be anything in `UIRequiredDeviceCapabilities`.  Similarly, this goes in both places.

Maybe check `#targetEnvironment(macCatalyst)`.

# respecting platform settings
* system colors
* accent colors
* dark mode

`liveViewSafeaAreaGuide`.  We recommend using `safeAreaLayoutGuide` instead.


# bringing it all together
