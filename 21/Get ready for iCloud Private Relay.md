Exciting new feature in internet privacy.

New service that prevents networks and servers from monitoring user activity across the internet.

# What is private relay?
Built into iOS and macOS.
Wont' always be affecting your app.  Only apply when a user is an iCloud+ subscriber and enables it.

What does it do?

Currently, people can inspect DNS.  Can fingerprint a user.

When connections reach the server, they can see a user's IP address.  Allows servers to determine user location without permission.

Servers can fingerprint user identity.

Proxies run by separate entitites.  Both apple and a content provider.  Only client IP address is visible to network provider and apple proxy.

Second proxy only sees the name the user is requesting.

No one in this chain, not even apple, can see both client IP address and what the user is accessing.

* QUIC
* HTTP/#
* Blind signatures
* Oblivious DoH


[[Apple's privacy pillars in focus]]

Private relay is focused on securing the most sensitive system traffic, without impacting user experience.

* Safari browsing
* DNS queries
* Subset of app traffic
	* Specifically, insecure HTTP traffic such as TCP port 80

## Compatibility
Filters and parental controls still work
Not applicable traffic
* Local network connections
* Private domains
* Network Extensions and VPNs
* Proxy configurations

[[Meet the Screen Time API]]

Not all networking occurs over the public internet.

Traffic taht uses a proxy is exempt.


# Prepare your app

Usually don't need to do anything.

Best practices

For several years, we've recommended URLSession and NWConnection.  Now you have one more reason to adopt them.  These APIs provide the best tools to understand how private relay is applying

* Analyze connections in URLSession and NWConnection
* Network Instrument in Xcode
* `URLSessionTaskTransactionMetrics`
* `NWConnection.EstablishmentReport`

[[Analyze HTTP traffic in Instruments]]

Use the Metrics APIs to understand when your connections use private relay.

Using task transaction metrics, can check if your task used a proxy.  Can also inspect timings and DNS name resolution in each stage of the proxy connection establishment.

Private relay takes a big step in independent/secure HTTP connections.
Now's the time to move off port 80.

* Private relay adds encryption to the egress proxy
* Prefer end-to-end security
* "Fine tune your App Transport Security settings" doc.

## Accessing location
* Prefer CoreLocation to IP address geolocation
	* Accuracy
	* Permission-based

[[Apple's privacy pillars in focus]]

## How to test
* iCloud+ subscription
* Ensure private relay is enabled
* Provide feedback


# Prepare your server

* Proxy IP address pools
* Shared across users
* Regional or city-level address mappings

Proxy IP addresses may be shared.  Each address is mapped to a specific city or region.

Detials of how to identify proxies are in resources.

Device uses ingress proxy to forward network requests using ingress proxy IP.

Ingress proxy forwards request to destination servers by choosing IP.

## Best practices
* Request user permission for location
* Transparency for user identification

To identify users, request a login or some other form of explicit identification.


# Manage your network

QUIC (HTTP/3) connections
Fewer DNS queries

Making sure your routers or network are tuned to handle this well.

1.  Device first sends DNS query for hostname.
2.  Device connects to IP address at TCP
3.  3-way handshake
4.  TLS exchange

Hostname and device are visible though.

With private relay, device sets up connection using QUIC.  Notice no hostname info is leaked.

On the server end, there's no protocol change.  Similar to traffic without private relay.  Server sees incoming connection from a proxy.

Most networks don't monitor user traffic.  However, enterprise or school may require intercepting all traffic.

* Block DNS resolution of ingress proxy names
* Disable Private Relay per network

Parental controls
* Prefer on-device content filter

[[Meet the Screen Time API]]
[[Network extensions for the modern mac]]

# Wrap up
Use modern, secure networking APIs
Test your apps with private relay
Prepare your servers


