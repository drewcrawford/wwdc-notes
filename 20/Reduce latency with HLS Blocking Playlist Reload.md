#hls 
# What is blocking Playlist Reload?
* HLS clients discover new segments by reloading the playlist
* Original HLS approach polling delays discovery.  If a client misses a playlist update, it will take another target duration before it notices it.
* Low-latency HLS introduced server Delivery Directives.  
* Client asks server to reload request until new segment appears
* Response unblocks when playlist updates

# Request flow
Server advertises blocking playlist reload `CAN-BLOCK-RELOAD=YES`
Clietn loads playlist first time with no query parameters
Reload request is made as soon as Playlist is received
Client uses `_HLS_msn` to get Playlist with next segment
Adds `_HLS_part` to specify Partial Segment

# Example
# With low-latency HLS

Server interprets part+1 -> next segment, part 0 if necessary

`_HLS_msn` and `_HLS_part` are ignored if playlist contains `EXT-X-ENDLIST`
Server unblocks immediately if playlist is newre than requested
Requested segment/part is not the most recent
It might even have rolled out of the playlist

Origin always returns the current version of the playlist, as long as it's new enough.
Server may time out if it spends too long being blocked.

# CDN support
Each request with a different MSN or part is cached separately.  
Different query parameter for each request prevents stale cache.
Each playlist response has a longer useful life in cache
Recommend lifetime six target durations

Configure your CDNs to coalesce duplicate blocking requests into a single request to the origin.  

# summary
* blocking playlist reloads enable efficient update discovery
* Triggered using delivery directives
* Improves performance across CDNs
	* positive caching: can be cached for longer
	* Negative caching: easy to detect cache miss
* Works with regular HLS
* Required for low latency HLS

