If your app provides an experience like this, it uses the local network.
The best way to use the local network is with bonjour.
* Media streaming
* interactive games
* Home and peripheral devices

Lower-level
* Network management
* Custom multicast protocols

With great power, comes great responsibility.

Users shoul dknow when your app uses the local network.

# Location
One of the most privacy-sensitive pieces of information that your app can access.

In iOS 13, your app needs location permissiont o access name and ssid. 

But simply accessing the network can provide a location hint.  

# in iOS 14
Users can control which apps are allowed to access the local network.

| Unrestricted                   | Requires permission              |
|--------------------------------|----------------------------------|
| TCP and UDDP to internet hosts | TCP/UDP to local network hosts   |
| AirPrint, AirPlay, AirDrop     | Bonjour browse/advertise         |
| HomeKit                        | IP Multicast and Broadcast, ICMP |

## Grant permission
* Prompts on first access
* Operations fail until allowed
* Provide usage description

If you notice a prompt you don't expect, you might be using a third-party SDK.

### app requirements
* `NSLocalNetworkUsageDescription` - explains reason
* `NSBonjourServices` lists bonjour service types

If you use any of these APIs
* Network.framework (`NWBrowser` and `NWListener.Service`)
* Foundation (`NetService`)
* MultipeerConnectivity
* dnssd APIs

[[Advances in networking, Pt 2 - 19]]

### custom multicast and broadcast

Custom discovery requires an entitlement
* IP multicast
* IP broadcast
* Bonjour service list `_services._dns-sd._udp`

Request entitlement in developer portal

[[Using multicast in your app]]

## update your app


```
Browser failed with -65555: NoAuth
```

Need to add string to Info.plist.

Want to delay API access until user does an action.


## design a great experience

* Avoid unexpected prompts.  Defer local network access, wait for user interaction
* Provide a clear usage description.
* Handle errors gracefully
	* Bonjour
	* NWConnection
	* URLSession -> makes sure to set `.waitsForConnectivity`.
	* Sockets
* Prefer system-provided frameworks
	* AirPrint
	* AirPlay
	* AirDrop
	* HomeKit

# Next steps
Test your app for local network access
Provide a usage description
Declare bonjour service types

