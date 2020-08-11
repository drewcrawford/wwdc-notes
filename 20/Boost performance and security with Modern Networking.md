Use URLSession and Network.framework

# Performance

## ipv6
IPv6.  Lower latency.  Test with NAT64 networks.
IPv6 has 1.4x faster connection setup.

Make sure you've enabled ipv6 on your server.

## http2
multiplexing to a single connection.
connection coalescing -> across apps, e.g. at os level
header compression

[[optimizing your app for today's internet - 18]]

79% of requests on safari use http2
1.8x faster requests

# Security
## tls 1.3
Faster handshake
Improved security 
Enabled by default

49% of all connections.  1.3x faster connection setup.

# Mobility
## Multipath TCP
Seamlessly switch between networks.

`multipathServiceType` on `URLSessionConfiguration`

[[Advances in networking, Pt 1]]

13% reduction in music stalls.  22% reduction in music stall duration

# Privacy

## Local network privacy
[[Support Local Network Privacy in Your App]]

Multicast, broadcast.

## DNS privacy

DNS-over-TLS
DNS-over-https

[[Enable encrypted DNS]]

This is at the OS level (devs can write extensions), but can be required by apps.

# Looking forward
* Encrypted SNI
* HTTP3 / QUIC.  Experimental preview which you can enable in developer settings.



