 #networking 
 
 "Responsiveness"
 Main factor that affects resonsiveness is the network response in the working condition.
 
 Typical internet speed test idle tells you about when you're *not* using it.  But how about when you *are* using it?
 
 RPM -> milliseconds are an abstract concept.  People are more familiar with metrics where higher is better.
 
 a few hundred RPM => few thousand RPM
 
 


 
# How network delays affect your app

CLI `/usr/bin/networkQuality`.  Sometimes repsonsiveness becomes a *lot* worse when in use.
 
 e.g. video conferencing.  Why is network behavior not reliable?  We think we'll fix it by upgrading our internet.  megabits vs gigabits.  A few megabits should be plenty for video conferencing.
 
 * Higher throughput does not mean higher responsiveness
 * Today's internet speed tests measure capacity, not speed
 * Bufferbloat severely degrades resonsiveness

[[Your app and next generation networks]]

Good news: modern queue management algorithms like CoDel?  That eliminate bufferbloat.

Controlled delay queuing algorithm

https://www.iab.org/activities/workshops/network-quality

High-throughput, low-delay.  Not an either/or choice. 

## Components of delay
* Processing time
* Transmission time
* Buffering delay
* Speed-of-light propagation delay (not going to change)

Whcih apps are hurt by high-speed network delay?  All apps

We should put equal effort into reducing the time waiting.  Techniques you can use to reduce those wait times.

RTT * number of round trips

Not a lot you can do about RTT, but you can control # of RTs.

# Adopt modern networking APIs

Responsiveness is inversely proportional to RTs.  

* HTTP/3 and QUIC
* TCP FO
* TLS 1.3
* Multipath TCP

Server-side support is required for all of these.

All of these technologies are available on iOS and macOS.  

## HTTP/3 and QUIC
* Faster connection setup
* Reduce delay in data delivery
* URLSession for HTTP/3
* Create NWConnection with QUIC parameters

[[Accelerate networking with HTTP3 and QUIC]]

By reducing HOL blocking, QUIC can reduce delays.

## TCP Fast Open
Send app data with TCP handshake

Allowing fast open, requires app to provide initial data.

```swift
parameters.allowFastOpen = true
connection.send() //do this first
connection.start() //do this second
```
Network.framework and sockets.

You have to be careful that you *only send idempodent requests with the handshake*.  

```swift
let tcpOptions = NWProtocolTCP.Options()
tcpOptions.enableFastOpen = true
```

Can send TLS handshake as initial data

[[Introducing Network.framework - 18]]

Recommended way to use FO is via Network.framework.  But if your app is built over sockets, use `connectx(...,CONNECT_DATA_IDEMPOTENT | CONNECT_RESUME_ON_READ_WRITE`


## idempodent?
Replay-safe operation.  No additoin effect if performed more than once.  

e.g., GET request is sent with HTTP handshake.  If acknowledgment is delayed or dropped, device will resend the HTTP get request which may get routed to a different server.

Acknowledgment may arise along with HTTP response.

Since GET request has no additional effect, it is idempotent.

## TLS 1.3
* Faster handshake
* better security
* enabled by default
* TLS 1.3 early data allows idempodent requests to be sent with handshake

[[Your apps and evolving network security standards - 17]]

## Multipath TCP

Save RTs when switching networks

On URLSessionConfiguration
`configuration.multipathServiceType`
or NWParameters
`parameters.multipathServiceType = .interactive`

[[Advances in networking, Pt 1]]

## How many RTs can you save?


| Protocol            | RTs to first byte |
|---------------------|-------------------|
| TLS 1.2 over TCP    | 4                 |
| TLS 1.3 over TCP    | 3                 |
| TLS 1.3 over TCP FO | 1-2               |
| HTTP3/QUIC          |                   |

Common to see RT times as high as **600 ms**.

4 RTTS = 2.4s ttfb

Huge time to wait and stare at network spinner.  By adopting modern network protocols, can reduce tftp to 0.6s.

Pay attention to number of roundtrips

## Real-world network conditions
* Hard to test all network conditions
* Developer settings => Network Link Conditioner to the rescue
* [[Advances in networking, Pt 2 - 19]]
* [[Designing for Adverse Network and Temperature Conditions]]

Reliable and repeatable way to test your app in different conditions that users may experience.


# inform the system about your traffic

How to reduce RTT?  Well there's a special way.

Most apps have a mix of traffic that's sent/received.  Lot of data that is exchanged from user's device running apps.  In real-world networks, a number of devices share the same network.  These devices simultaneously send/receive significant amount of data while using apps.

To avoid building up long queues in this shared network, it's crucial to classify app data appropriately so system manages traffic efficiently.

Make your foreground traffic faster.

Many apps prefetch content etc.  When app prefetches substantial data, network might have a full queue.

If at this point, the user initiates a network activity like viewing their profile page, the response fo rthis request will be queued at the end.  May take awhile before it is dequeued.

consider what happens when we mark tasks as bg. 

Dramatically reduce size of network queue, which becomes available for foreground data.

In new OS, background service type as improved dramatically with new congestion algorithms.

New algos reduce delay significantly but ensure that the bg transfers finish in the same time as other traffic.

How to take advantage?

## Foreground
* Use default `URLSession`, set background on `URLRequest`
* NWConnection => `serviceclass.background`

If your app starts a long-running transfer, create a *background* URLSession to continue running after suspending.

Can set `isDiscretionary` to `true`, allowing the system to wait for optimum conditions

## Minimize queuing at the source
 * Automati for URLSession and NWConnection
 * BSD socket option TCP_NOTSENT_LOWAT

[[Your app and next generation networks]]

Consider setting this option on your server to ensure that tehy're using this flag.

# Next steps
Adopt modern networking protocols
Use bg mode for prefetching assets, bulk transfers, etc
Test under different network conditions
