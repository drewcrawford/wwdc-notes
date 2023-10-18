#l4s #networking 
Streaming video, multiplayer games, and other real-time experiences depend on responsive, low latency networking. Learn how Low Latency, Low Loss, Scalable throughput (L4S) can reduce network delays and improve the overall experience in your app. We'll show you how to set up and test your app, network, and server with L4S.

# Explore L4S

Today, we're going to talk about reducing app delays in your app.  with l4s.

Low latency low loss, scalable throughput.  Dramatically improves the performane of apps where network latency can impact UX.

Reduce content loadtime, improve video quality , and bring more repsonsibve collaboration between users.

Cause high latency, packet loss.  For example, you might have experienced delays or video stalls during a video call, when someone else is using the network.

We made a sample app.  I will show you two video calls, with/without l4s.  In both cases, we use low bandwidth.  Therea re also multiple other devices actively using the same network.

In this demo, l4s allowed us to achieve significant improvements across all key networking metrics.

rTT distribution is much tighter.  up to 45ms.  But, with l4s turned on, even the worst-case latency was cut by 50%.  And reduced to less than 25ms.

Let's look at packet loss rate.  Lower loss represents a more reliable connection between devices.  With legacy, packet loss rate is 40%.  With l4s, packet loss is nearly eliminated.

Video rendering metrics.  Legacy: stall percentage 100%.  l4s: very low.

Recieved video fps.  Much higher and more consistent via l4s.  

How a queue builds up.
* sender
* hop 1
* hop N
* server

When a packet is sent, it flows through various hops.  This set of hops, the packet traverses is called a network path.  On a path, you can only deliver data at the rate that is supported by the slowest hop.  That's the bottleneck. For many people, it's their ISP.  

Packets can be dropped, etc.  L4s solves this problem through collaboration between client, server, and bottleneck.  With L4s, a client/server needs to cooperate, etc.

L4s capable sender uses a technology called ECN (explicit congestion notification).  When transmitting a packet, the sender indicates support for L4s using the ECN bits in the header.

When an L4s hop receives this, it applies L4s queue management.  When a queue starts to build up at the bottleneck, L4s can indicate congestion by setting a different ECN_LABEL.  Network has experienced congestion before forwarding to the next hop.  Receiver counts the number of packets marked with congestion label, and provides feedback.  Sender uses this feedback to understand the congestion on the network.  Sender adjusts its sending rate.

What you need to do to enable this collaboration ?




# Prepare your app
Adopt URLSEsison and Network framework for L4S
Use HTTP/3 or QUIC
Alternatively, use HTTP/2 or TCP
Use zero code changes!

If your app uses a custom protocol, there are a few things you need to implement:
* RFC 9330 is a good starting point to learn about requirements.
* Scalable congestion control algorithm
* ECN (Explicit Congestion Notification) validation
* Relay ECN feedback
Use ECN in Network framework
* ECN property on `NWProtocolIP.Metadata`
* can use setsockopt in BSD sockets.
# Set up your server
Consult your QUIC server provider about enabling ECN and L4S
Configure L4S support for TCP in your server
Linux: https://github.com/L4STeam/linux

# Configure test network
L4S capable network
* pass-through ECN bits
* L4S queue management at bottleneck hop

Test L4S with Internet Sharing
In sonoma, the mac because an additional network hop.  If you configure the mac to be the bottleneck, it will apply l4s queue management, allowing you to view the complete l4s network.

To enable internet sharing,

system settings -> general? -> sharing.  Click the info button next to "internet sharing".  

### Throttle Internet Sharing - 14:38
```bash
sudo ifconfig en1 tbr 10Mbps
```

replace the interface name with the interface you are using.  

Can enable in macOS like this:


### Turn on L4S - 15:36
```bash
sudo defaults write -g network_enable_l4s -bool true
```

in iOS: developer settings?  

# next steps
* test your app with L4S enabled
* ON l4s-capable network
* on legacy networks
* ask server provider to enable L4s
* Provide feedback!


# Resources
https://developer.apple.com/documentation/network/testing_and_debugging_l4s_in_your_app
