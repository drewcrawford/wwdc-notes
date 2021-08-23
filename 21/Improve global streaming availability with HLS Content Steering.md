#hls 

# Today's topic
* Connectivity is global
* Global streaming delivery is challenging
* Let's focus on availability
	* Network congestion mitigation
	* Error recovery

# ex
If a CDN Becomes overloaded, it's difficult to solve.  Existing traffic will continuously overload the current CDN.

Host your own steering server.  Establish a side channel to update them with latest CDN policy.  ex, can redirect CDN users to another node.

What happens when there's a network outage?   With the help of steering server, clients can be updated of new CDN prorities.  Client can move onto the next CDN in the list.

# Integration

`#EXT-X-CONTENT-STEERING:SERVER-URI="/steering" PATHWAY-ID="CN"`

Streams are now grouped with the `PATHWAY-ID`.  Multiple streams are listed.

Pathway ID at toplevel specifies initial pathways to use.  `SERVER-URI` specifies the steering server

Can also duplicate groups for each pathway.  

# Playback startup
* Select Variant Streams belonging to the initial `PATHWAY-ID`
* Load media as usual
* Start making periodic Steering Manifest requests in parallel

JSON document.

* PATHWAY-PRIORITY key => list of pathway IDs ordered by priorities

# Content steering evaluation
* Exclude Variant Streams ineligible for selection
* Penalized variant streams
* Unsupported codecs
* Select a pathway that
	* Eligible variant streams
	* Highest pathway priority
* Switch to new Pathway if different than current Pathway
* Otherwise stay on current Pathway
* New steering request

# Steering manifest
`TTL` => Number of seconds before next steering request

Also worth noting that steering server is able to change this TTL value in each manifest response.

Optional RELOAD-URI tells the client where to grab the next manifest.

# Wrap up
* Content steering
* Fine-tune global streaming availability
* See the links below this video for "HLS Content Steering Specification"
* Validate using Apple's HTTP Live Streaming Tools
* Shout out to the HLS Interest forum
* Try it in WWDC Seed builds

**Act now!** Not available in stores

https://developer.apple.com/streaming/HLSContentSteeringSpecification.pdf
https://developer.apple.com/streaming/

