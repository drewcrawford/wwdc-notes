#watchOS 

[[Streaming audio on watchos 6]]

# New codec

You could use AVPlayer or your own custom protocols.  This year, we're bringing support for a new audio codec.

xHE-AAC (MPEG-D USAC)
Equivalent quality at lower bitrate
Higher quality at equivalent bitrate

Also include at least one of
* AAC-LC
* HE-AAC
* HE-AACv2

for interoperability

[[Deliver a better HLS Audio Experience]]

# Fairplay Streaming on watchOS
Play protected HLS content 

`AVContentKeySessionDelegate`.  AVFoundation asks to load your key and you get the key.

Decouples key loading from media loading and playback.  Allows applications to optimize key delivery.

[[AVContentKeySession Best Practices]]

# Best practices
* avoid unnecessary round trips
* minimize http redirects
* Pre-fetch resources including certificates and keys
* Use AVContentKeySession to prefetch content keys
* Cache certificate on device - use expiry from HTTP responses to decide how long to cache.
* We recommend using a higher cache duration of 20s.


[[Creating Audio Apps for watchOS]]
