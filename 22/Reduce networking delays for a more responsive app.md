How to reduce networking delays in your app.
# Latency matters
How quickly ocntent can be delivered to your app.  All apps that use networking can be affected by slow network transactions.

Understand how your app's packets travel in a network.  When your pap or framework requests data, pakcets are sent out by the networking stack.  Assumed that packets go directly to the server with no delays in the nwtwork.  But inr eality, the slowest link of the network usually has a large queue of packets to process.  Packet from your app waits behind a large queue until packets ahead of it are processed.  This queuign increases the duration of each router between your app and your server.

Aggravated with multiple roundtrips.

Duration of each roundtrip multiplied by 4 trips.  Given that each RTT is inflated by queuing in the network, the resulting total time is simply too long.

duration of RT * number of round trips

Page load time as bandwidth increases.  Latency improves up to 3mbps, but not at higher speeds.  After 4mbps, each incremental increase has almost no improvement in page load time.  Apps can be slow even after upgrading to gigabit.

Page load time as latency decreases => every 20ms decrease, there's a linear improvement in page load time.  Down to 20ms.


# Design responsive apps
Actions you acn take to erduce latency and make your apps more responsive.  Reduce latency by adopting modern protocols such as ipv6, tls 1.3, and http/3

[[Reduce network delays for your app]]
[[Accelerate networking with HTTP3 and QUIC]]

20% safari traffic using http/3.

HTTP safari, request completion time normalized for RTT.  56% the time of http1.  Vs 81% for http/2.  Median request completion time as a multiple of RTT.

For http3,
```swift
let configuration = URLSessionConfiguration.default
configuration.multipathServiceType = .handover
```

If you design your own protocol that uses UDP directly, iOS 16 ad macOS ventura introduced a better way to send datagrams.  QUIC datagrams provide many benefits over plain UDP.  The most important thing is that QUIC reacts to congestion in the network, keeping RTT low an reducing packet loss.

For QUIC, enable on NWParameters.
```swift
let parameters = NWParameters.quic(alpn: ["myproto"])
parameters.multipathServiceType = .handover
```

Create QUIC datagram flow

```swift
// Only one datagram flow can be created per connection
let options = NWProtocolQUIC.Options()
options.isDatagram = true
options.maxDatagramFrameSize = 65535
```

Send/receve just like any other QUIC stream.

# Speed up your server
Despite often running on top of the line hardware, it's possible your server becoems a problem.

https://github.com/network-quality/server

```bash
networkQuality -s -C https://myserver.example.com/config
```

Measure bufferbloat in your service provider's network, as well as on your server.  Configure your server to act as a destination for this.  Once you have done that, run `networkQuality`.  First against apple's defualt server, and then against your own configured server.  By comparing you may discover headroom.

## Make streaming more responsive.
Waiting for video to rebuffer.  We investigated this slowness.  We used network quality tool to test behavior of streaming server and we foudn responsiveness was poor.

TCP, TLS, HTTP buffer sizes.

TCP => 4mb
tLS => 256kb
HTTP => 4mb

ram is plentiful.  But just because some buffering is good doesn't mean that more buffering is better.  Our responsiveness highlighted this issue.  A newly-generated packet was queued behind stale data and this created a lot of additional delay in delivering the most recent packet.  We reduced the buffer size to
HTTP => 256kb
TLS => 16kb
TCP => 128kb

```bash
% cat /opt/ats/etc/trafficserver/records.config

# Set not-sent low-water mark trigger threshold to 128 kilobytes
CONFIG proxy.config.net.sock_notsent_lowat INT 131072

# Set Socket Options flag to the sum of the options we want
# TCP_NODELAY(1) + TCP_FASTOPEN(8) + TCP_NOTSENT_LOWAT(64) = 73
CONFIG proxy.config.net.sock_option_flag_in INT 73

...

# Enable Dynamic TLS record sizes
CONFIG proxy.config.ssl.max_record_size INT -1

...

# Reduce low-water mark and buffer block size for HTTP/2
CONFIG proxy.config.http2.default_buffer_water_mark INT  32768
CONFIG proxy.config.http2.write_buffer_block_size   INT 262144
```

We recommend these on apache.  Look for equivalent options on your server.  We reran network quality and this time we got a high RPM score.

By getting rid of unnecesary queuing at the server, we made random access more resopnsive.  Regardless of how your app uses networking, these changes can make your app more repsonsive and delivera  better UX.

# Speed up the network

Measure your network
* responsiveness in iOS 15

bufferbloat tests
* https://apps.apple.com/us/app/speedtest-by-ookla/id300704847
* https://github.com/network-quality/goresponsiveness
* https://www.waveform.com/tools/bufferbloat

```
https://www.waveform.com/tools/bufferbloat
https://github.com/network-quality/goresponsiveness
https://www.speedtest.net/
```

ookla shows RTT in MS, and if you divide 60,000 by that number, you get the RPM.  Use these tools to measure how well your network is preforming.  Best wya to understand delays in a network is with a delay-sensitive application.

Here I logged on to remote machine using screen sharing.  Clicked on different finder menus but the display of each menu was very sluggish.  Show clock on both machines.  Even though time is synchronized, I see the difference.

Presence of a large queue at slowest link of the network. Packets were stuck in the queue.  Apple is working with a networking community on a new technology called L4S.  Beta in iOS16 and ventura.  Reduces queuing latency significantly.

Network explicitly signals congestion instead of dropping packets, sender adjusts sending rate.  Makes it possible to keep low queuing in the network without any packet loss.  

This technology is not just for screen sharing.  L4s improves all of today's apps and opens the door for future apps that wouldn't even be possible today.

Average RTT is much better when traffic is shared with other devices.  massive reduction in RTT.  This is the primary reason for improvement in experience.

enable in
* iOS developer settings
* macOS

```bash
defaults write -g network_enable_l4s -bool true
```

Linux
* accurate ECN
* scalable congestion control algorithm

**To see L4s benefits, network support is needed.**

# Next steps
* Adopt HTTP/3 & QUIC
* Eliminate unnecessary server queuing
* Support and test L4S







* https://apps.apple.com/us/app/speedtest-by-ookla/id300704847
* https://github.com/network-quality/goresponsiveness
* https://www.waveform.com/tools/bufferbloat
* https://github.com/network-quality/server