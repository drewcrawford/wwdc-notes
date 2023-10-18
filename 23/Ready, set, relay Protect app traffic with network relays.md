Learn how relays can make your app's network traffic more private and secure without the overhead of a VPN. We'll show you how to integrate relay servers in your own app and explore how enterprise networks can use relays to securely access internal resources.

# Discover relays

great features, such as iCloud private relay, mail privacy protection, hiding IP addresses, etc.

your app may also handle sensitive information to keep private.  Or ensure that your own servers can't associate with client IP.

Your app can use relays that you select to provide strong privacy protections.  Special type of proxy that is optimized for performance, uses latest transport/security, and has a modern network stack.

two standard protocols defined with IETF

MASQUE, Oblivious HTTP

## Masque
* proxy any TCP or UDP connections
* Add privacy with multiple hops
	* iCloud private relay
* Access enterprise resources without VPN
* Built on TLS 1.3, QUIC, and HTTP/3
* Fallback to HTTP/2

## oblivious HTTP
* each message separately encrypted and proxied
* requires servers that support oblivious HTTP

[[23/What's new in Privacy|What's new in Privacy]]

# configure relays in your app
can choose specific relay servers.

new proxy configuration class allows you to define 
* network.framework
* urlsession
* webkit

`ProxyConfiguration`.

supported proxy types
* masque relays
* oblivious http
* secure HTTP connect
* HTTP connect
* socksv5
* 
### 4:52 - Configuring a relay
```swift
import Network

let relayEndpoint = NWEndpoint.url(URL(string: "https://relay.example.com")!)
let relayServer = ProxyConfiguration.RelayHop(http3RelayEndpoint: relayEndpoint)

let relayConfig = ProxyConfiguration(relayHops: [relayServer])
```

http2 server used as a backup in case http3 is blocked.


### 5:40 - Configuring a relay in Network framework
```swift
import Network

let relayEndpoint = NWEndpoint.url(URL(string: "https://relay.example.com")!)
let relayServer = ProxyConfiguration.RelayHop(http3RelayEndpoint: relayEndpoint)

let relayConfig = ProxyConfiguration(relayHops: [relayServer])

var context = NWParameters.PrivacyContext(description: "my relay")
context.proxyConfigurations = [relayConfig]

let parameters = NWParameters.tls
parameters.setPrivacyContext(context)

let connection = NWConnection(host: "www.example.com", port: 443, using: parameters)
connection.start(queue: .main)
```

now connection will send all traffic through the proxy.

Can also use the same proxy config defined directly in URLSession

### 6:07 - Configuring a relay in URLSession
```swift
import Network

let relayEndpoint = NWEndpoint.url(URL(string: "https://relay.example.com")!)
let relayServer = ProxyConfiguration.RelayHop(http3RelayEndpoint: relayEndpoint)

let relayConfig = ProxyConfiguration(relayHops: [relayServer])

let config = URLSessionConfiguration.default
config.proxyConfigurations = [relayConfig]

let mySession = URLSession(configuration: config)
let url = URL(string: "https://www.example.com/api/v1/employees")!
let (data, response) = try await mySession.data(from: url)
```

also use in webkit

### 6:30 - Configuring a relay in WebKit
```swift
import Network

let relayEndpoint = NWEndpoint.url(URL(string: "https://relay.example.com")!)
let relayServer = ProxyConfiguration.RelayHop(http3RelayEndpoint: relayEndpoint)

let relayConfig = ProxyConfiguration(relayHops: [relayServer])

let webkitConfig = WKWebViewConfiguration()
webkitConfig.websiteDataStore = WKWebsiteDataStore.nonPersistent()
webkitConfig.websiteDataStore.proxyConfigurations = [relayConfig]
let webView = WKWebView(frame: .zero, configuration: webkitConfig)

let url = URL(string: "https://www.example.com/api/v1/employees")!
webView.load(URLRequest(url: url))
```


# Access enterprise resources

Can configure relays for the whole device.  Great wya to use relays to provide access to private enterprise network uses.

Modern access to enterprise resources
No complex session negotiation
fewer roundtrips before data is transferred
Does not require IP assignment or tunnels
Multiple per-domain and per-app relays

cisco -> provides an enterprise relay service for "cisco secure edge"
two ways to install relays
* `com.apple.relay.managed` profile payload via MDM.  Per-app, per-domain, or default route
* `NERelaymanager` from your app.  Per domain or default route
* available in new OS releases.  VPNs now supported on tvOS as well.

### 9:15 - Configuring a relay on the device with a configuration profile
```xml
<dict>
    <key>PayloadType</key>
    <string>com.apple.relay.managed</string>
    <key>Relays</key>
    <array>
        <dict>
            <key>HTTP3RelayURL</key>
            <string>https://relay.example.com</string>
            <key>PayloadCertificateUUID</key>
            <string>5AB702EC-32F3-48A9-94FE-8EA1C67ACF46</string>
        </dict>
    </array>
    <key>MatchDomains</key>
    <array>
        <string>internal.example.com</string>
    </array>
</dict>
```

app can use NERelayManager to add relays programmatically.
### 9:42 - Configuring a relay on the device with NetworkExtension
```swift
import NetworkExtension

let newRelay = NERelay()
let relayURL = URL(string: "https://relay.example.com:443/")
newRelay.http3RelayURL = relayURL
newRelay.http2RelayURL = relayURL

newRelay.additionalHTTPHeaderFields = ["Authorization" : "PrivateToken=123"]

let manager = NERelayManager.shared()
manager.relays = [newRelay]
manager.matchDomains = ["internal.example.com"]

manager.isEnabled = true
do {
    try await manager.saveToPreferences()
} catch let saveError {
    // Handle error
}
```

Demo

# Next steps
* review how you can use relays
* adopt custom relays in your apps
* migrate from VPNs to relays
* 
# Resources
* https://developer.apple.com/documentation/networkextension/relays
