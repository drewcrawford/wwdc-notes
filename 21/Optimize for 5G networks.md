#networking 

# Path to 5G
Each step in evolution has brought dramatically new capabilities.

2G Networks had simultaneous data and voice.
Text messages
Pictures messages
Multimedia
All at 384 kbit/s

Meant the performance of the network didn't matter as much.  But once we have access to a network, we push it to its limits.

3G => "data connectivity"
video calls, mobile TV, internet

4G/LTE.  VoIP, high-quality video, data-intensive UX

Exploring shared augmented reality experiences, realtime games, etc.

fantastic new experiences were made possible with a dramatic increase in network performance.

Always looking to push the envelope, faster speeds, etc.

## 5G
Increasing need for speed

Look at performance over generations, each new generation had increase in speed.  Each jump in speed leads to a leap in possibilities.

Richer, previously-impossible experiences.

Faster and more synchronized transactions.  More devices concurrently.

Data-related services in congested environments.   Remember thsoe?

Each leap in performance meant increased opportunities.

# Built-in intelligence
Ensure your app will perform optimally regardless of network.

Easy to imagine that the previous network disappeared.  But portions of the past in each generation.

Varieties
* Non-standalone (NSA)
* Standalone (SA)

## NSA
Existing LTE core
Uses LTE and 5G links
Operates below 7ghz, mmwave

## SA
* Built in new 5G core
* Operates below 7ghz, mmwave
* Provides lower latency

## 4gbit/s

3gb/sec.  20x faster than LTE.  With a ping latency of 7ms, that's pretty fast.

While these provide lower latency and fast data speeds, they are power hungry.

Super-fast experience, but not necessarily at the expense of battery life.

There's a range of network types with varied performance characteristics.

We pick the best network using 2 techniques based on various techniques.

* Battery
* Security (public wifi, etc)
* speed
* foreground/bg


* Automatic switch to 5G
* Smart data mode
	* Detect network congestion on any network.

# Practical guidance

Stop considering the network type.

Using network type to drive behavior is guaranted to prevent accessing benefits

Don't assume if wifi is available, cellular isn't.  We suport multiple paths

Use high-level frameworks.  AVFoundation for media.   CallKit for VOIP.

Tune for constrained and expensive paths.

In most cases, expensive/constrained are derived by the system based on scellular plan.  Howeve,r user can affect as well.

* Allow More Data on 5G
* Standard (expensive)
* Low Data Mode (constrained)

## Expensive and constrained
high-level networking frameworks.  Only consideration when determining the type of networking services available.

URLSession lets you take advantage to benefit from these requests.  To benefit from inexpensive networking over 5g, consider `.allowsExpensiveNetworkAccess`.

If your app implements policy based on `expensive`, provide a way for users to influence the policy.

In network.framework you can check for `constrained`.

AVFoundation provides similar options.

# Wrap up
* 5G is here
* Keep it high-level
* Give us feedback




