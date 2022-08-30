https://developer.apple.com/security/
# What is developer mode
A nwe mode for iOS 16 and watchOS 9 that enables developer workflows
Disabled by default and requires explicit enrollment
Persists acrossr eboots and system updates
setup can be automated

Further improves security for  aast majority of users that aren't develoeprs
developer featuers aare being abused in targeted attacks
most people od not need to use

most common distribution flows will not require
* testflight
* enterprise
* app store
developer mode is only required for local development


# Using developer mode
* run and install developments igned applications
	* including personal team
* debug and instrument your apps
* automate testing

You need to connect your device to xcode for the menu item to appear
IOS 16 beta releases have this visible at all times
installing an application without xcode (configurator) willa lso enable

Settings=>privacy and security
for automation you can use `devmodectl`.

## Demo
Requires you restart.

# Automation flows
Can be time-consuming if you're dealing with multiple devices.  Automation flows have one limitation
* only devices without passcodes can be automatically enrolled
* due to restart requirement
macOS ventura ships with `devmodectl`
* single device one-off mode
* streaming mode

## Demo
```bash
devmodectl streaming
```

You will get a notification on the device.

# Wrap up
* starting with IOS 16 and watchOS 9 you'll need to enable developer mode to perform common development activities
* For automation, use `devmodectl`

[[What's new in notarization for Mac apps]]
 