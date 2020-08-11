#hls #video

# End of low-latency HLS beta
Available to everyone in ios14, tvos14, watchos7, macos11

# Reducing segment delay

Last year we described using HTTP/2 push to send segment with playlist
But people told us it's not compatbiel with how ad-supported content gets delivered.

Replaced push with Blocking Preload Hints

Client requests next part in advance and the server will hold onto request until it can send it.
[[Discover HLS Blocking Preload Hints]]

Actually perform better than HTTP/2 push when you're using a CDN.  Driving the request from the client triggers global CDN cache fill.  

We still require HTTP/2, just not Push.

# Other improvements

Eliminated \_HLS_Report All rendition reports are provided unconditionally to avoid explosion in params?
Defined `EXT-X-DATERANGE` handling for playlist delta updates so the update only carries the most recent one

[[Optimize live streams with HLS playlist delta updates]]

Added gaps in parts and rendition reports.

# tools changes
Reference implementation now produces fmp4/cmaf
Added self-contained ll-hls origin written in go.
Incorporated low-latency HLS tools into the regular HLS tools package.

[[improve stream authoring with hls tools]]


