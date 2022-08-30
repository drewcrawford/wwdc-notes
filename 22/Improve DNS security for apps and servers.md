* https://developer.apple.com/forums/tags/wwdc2022-10079
* https://developer.apple.com/forums/create/question?&tag1=51&tag2=176&tag3=394030

# DNS is not secure
Translates domain names which are ? and easy to remember into other addresses for machines.

TCP, TLS, QUIC, rely on having an IP address.  So everything starts with DNS.

TLS is widely used to secure internet communication.  
DNS, the foundational layer, has some security issues.

**DNS is historically not secure**.  Designed in 1983 with few security considerations.  In the years since, many DNS attacks have been created.

Cache poisoning attack.  Attacker exploits flaws of DNS resolvers and make them cache incorrect IP addresses.  Causing clients to connect to malicious hosts.  This reveals one vulnerability of DNS: it's not authenticated.

No way to validate answers, can be spoofed, etc.

DNS Sniffing attack.  Attacker watches DNS tracfic between client and DNS server, collecting the client's history.  Serious problem for user privacy.

DNS Traffic was originally unencrypted.  

DNS needs to be both authenticated and encrypted.  When we use DNSSEC to sign DNS response, it provides authentication.  Whenw e use TLS and HTTPS to encrypt DNS resolution, it ensures privacy.  

# DNSSEC
* data integerity
* denial of existence
* Cryptographic authentication

## Data integrity
Attaches signature in response.  If response is altered, siganture won't match.

## Denial of existence
asserts existence and non-existence.  By using special recordsl ike NSEC, tells you what the next record name is securely in alphabetical order.  Any name not listed does not exist.  

## Authentication chain
Device wants to resolve www.example.org with DNSSEC enabled.  Sends queries asking for IP address, signatures, keys, etc.  

If hash matches preinstalled anchor, trust chain can be established securely.  With the trust chain, the IP Addresses of www.example.org are now authenticated.

## requirements
* support ipv6
* sign your domain
	* if you fail to do this, there's no benefit but there is additional traffic and time
* adopt APIs

```swift
let configuration = URLSessionConfiguration.default
configuration.requiresDNSSECValidation = true
let session = URLSession(configuration: configuration)
```

If you don't want it for the full session, you can do it at the request level.
```swift
var request = URLRequest(url: URL(string: "https://www.example.org")!)
request.requiresDNSSECValidation = true
let (data, response) = try await URLSession.shared.data(for: request)
```

On Network.framework, #networking 

```swift
let parameters = NWParameters.tls
parameters.requiresDNSSECValidation = true
let connection = NWConnection(host: "www.example.org", port: .https, using: parameters)
```

Validation behavior
* only ocnnect to validated addresses
* Response that fails validation is discarded
* No error returnd for the validation failure
* Validation result is re-evaluated automatically

## Validation behavior
* siganture mismatch
* Unable to establish a trust chain
* No DNS over TCP or EDNS0 option support
* IPv4-only signed domain on IPv6-only network

# Encrypted DNS with DDR
DNSSEC only does signatures, not encryption.

Last year,
[[Enable Encypted DNS]]

Manual
* NEDNSSettingsManager or DNSSEttings profile
* `NWParameters.PrivacyContext`

For more information, please watch [[Enable Encypted DNS]]

Now can be used automatiaclly if your network supports DDR (discovery of desginated resolvers)

https://1.1.1.1/dns-query

Common mechanisms such as DHCP or Router Advertisements only provide a plain ip address.  DDR is a new protocol developed in the IETF by apple.  Provides a way for DNS clients to learn this necessary information by using a special DNS query.

When you join a new network, it will issue XVCB to \_dns.resolver.arpa.  Reply with one or more configurations.  Device uses this information to set up an encrypted connection to the designated resolver.

DDR applies to a single network at a time.  Device will use encrypted DNS automatically only if the current network supports it.
Not compatible with private IP addresses.  Such IP addresses are not allowed in TLS certificates.  

## Client authentication
configure certificates using `ideneityReference` in `NEDNSSettings`
Applies to both
* DNS over TLS
* DNS over HTTPS

This allows encrypted DNS servers in enterprise environments where servers need to authenticate clients.


# Next steps
* sign your domain with DNSSEC
* Enable DDR on your network
* Use client authentication in enterprises
* 







* https://developer.apple.com/forums/tags/wwdc2022-10079
* https://developer.apple.com/forums/create/question?&tag1=51&tag2=176&tag3=394030

