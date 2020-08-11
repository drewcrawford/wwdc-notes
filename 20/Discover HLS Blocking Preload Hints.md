# What are Preload Hints?
Recall Blocking Playlist Reload
[[Reduce latency with HLS Blocking Playlist Reload]]
* client asks server for next generation of Playlist in advance
* Server unblocks request when Playlist updates
* Provides immediate discovery of new segments

Preload hinting is a similar approach for loading media
* make segment request in advance to lower response time.

# Benefits of Preload Hints
* Media data begins flowing to client from server sooner
* No RTT bubble between updating playlist and requesting segment
* Client-issued requests also drive HTTP cache fill
* Significant win for global CDN deployments

# Preload hinting in low-latency HLS
Every LL-HLS playlist contains an `EXT-X-PRELOAD-HINT` tag
* carries URL of the next Partial Segment expected to appear

When it receives the Playlist, the client immediately requests the hint URL
* Server holds hint URL request (blocks) until segment is completely available
* Server generally releases hint response...

Client generally sees both arrive at the same tests.  Also used to signal upcoming MAP tags.

## request flow

1.  Client requests playlist with part 2.  Blocks
2.  Client issues request hinted URL 2.  Blocks
3.  Eventually, server finishes encoding.  Then returns hint request
4.  At tsame time, add new partial segment to playlist...
5.  ...and update playlist request

## syntax

# Hinting byte-range partial segments
A Partial Segment can be a byte-range of a larger resource
`EXT-X-PRELOAD-HINT` can include byte-range start and length
* length is omitted if it is unknown

`EXT-X-PRELOAD-HINT:TYPE=PART,URI="23.MP4",BYTERANGE-START=0,BYTERANGE-LENGTH=4044`

# CMAF chunks
essentially fMP4 fragments
Build Segments by appending CMAF chunks as they are produced
Requires Chunked Transfer Encoding between server and client
Generally requries a specialized CDN
Serve same media to LL-HLS and LL-DASH

# using CMAF chunks with LL-HLS
Each resource (URL) corresponds to a Parent Segment
EAch Parent Segment contains multiple CMAF chunks
Each CMAF Chunk is a Partial Segment

* specified as a parent segment URL + byte-range
* `#EXT-X-PART:DURATION=1.0,URL="segment 1.m4s",BYTERANGE=5000@3000`

# Preload hinting of CMAF chunks
`EXt-X-PRELOAD-HINT` contains UR of parent segment
* URL set to the current Parent Segment being produced
* `BYTERANGE-START` set to the start of the current CMAF chunk being produced
* BYTERANGE-LENGTH is unspecified since we don't have it
* Client issues blocking GET request for every new hint URL
* Server progressively unblocks a new byte-range every time a CMAF chunk is produced

# Furhter notes
A server can change its plan after publishing a hint
* early return from ad
* If Playlist update no longer references URL, client cancels its request
* Hints request for pre-recorded content are not required to block

[[Adapt ad insertion to low-latency HLS]]

# Wrap up
Preload hint front-loads request for next Partial Segment
* part starts flowing to client (and CDN) without delay
* Can hint either entire resources or byte ranges
* Supports Chunked Transfer delivery of CMAF segments
* Low-latency HLS requires Preload Hints

