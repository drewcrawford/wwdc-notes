#swiftui 

# Cross-device connectivity
Fitness and meditation apps often work best on a large screen where a coach can demonstrate moves.  People benefit from sieeing their data collected from apple watch.  Gaming experiences.  Input/actions from a connected iPhone or augmented by extending a second-screen experience.

All share a need for reliable and easy-to-use cross-device connectivity.
# Introducing DeviceDiscoveryUI
#devicediscoveryui
Secure, privacy-preserving discovery of nearby devices.  Pairs with network framework to infer device-to-device connectivity.  Easy discovery of
 devices
 No longer implement key exchange.  
On the right is the list of discovered devices.  As some apps may be available on specific platforms, can filter by platform.

System prompts for permissiont o create a connection.  With this euser consent, you no longer need to request access to the whole network.  Once permission has been granted, your app will be launched to handle connections.  

No longer needs to be running on both devices for a connection to be established.  System immediately launches your app.

System takes people to the appstore.  
# Adopting DeviceDiscoveryUI
Updating TicTacToe
* Universal purchase
* declare application service in Info.plist
* Declare browsers on tvOS
* Declare advertisers on iOS, iPadOS, and watchOS
* Create and show device picker
[[Advances in networking, Pt 2 - 19]]
[[Support local network privacy in your app]]


Our app needs to tell the system what services to discover and what platforms they supoprt.  Add our new application services Info.plist.

Will map to one of two things.

Declare Browsers array that contains all application services that our app discovers.

First item represents our application service.  Service identifier, usage description, platforms.
Service identifier => name of our service
usage description => displayed in the device picker
Platform support => platforms supported by the service.  System filters discovered devices accordingly.

Since DeviceDiscoveryUI will launch our app when it's not running,   The ysstem uses this array to know which services to advertise.  Make srue this is the same for every platform.

# Discover devices
We're using a connected device as a controller to play the game.

Becuase this replaces the need to browse for nearby devices, we can remove the PeerBrowser file and our passcode extension to NWParameters as they're no longer necessary

Use the new convenience initializer, applicationService, on NWParametedrs.  Everything we need for this local connectivity.

Use our existing framework for communicating gameplay actions, just add it to the protocol stack.

Device picker is how our app discovers nearby iPhones, aiPads, and apple watches.  
1.  Check if supported.
2. Create picker
3. Present.  Always as a full-screen modal view.
4. access async endpoint.  
# Open connections
Once our app has been launched, we need to fulfill the promsie we made to the system by creating an NWListener.  Created as soon as the app is launched to accept incoming connections.  Same parameters as before, application service should use identifier from the plist.

When TV opens the connection, the listener will receive the connection here.  Now that the connection has been properly established, handle application state transitions.  When our app is backgrounded, transition to the failed state, ECONABORTED.  Establish a new connection from the TV to the same endpoint.  This new connection stays in the preparing state and moves to the ready state when the app is resumed.  New connection will be delivered to the same listener.

# Next steps
* Explore TicTacToe example
* Adopt DeviceDiscoveryUI
* Provide feedback

Please file reports in feedback sassistant.  We're excited to improve this technology together.


* https://developer.apple.com/forums/tags/wwdc2022-110339
* https://developer.apple.com/forums/create/question?&tag1=176&tag2=477030
* https://developer.apple.com/documentation/network/building_a_custom_peer-to-peer_protocol
