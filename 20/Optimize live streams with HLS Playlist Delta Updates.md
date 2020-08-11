#hls 

# What are playlist delta updates for?
An HLS client reloads its media playlist to discover new segments.
OK when playlist is small.  But what if event is long?

Server sends only most recent changes to client
Client combines to reconstruct server version

# Delta update flow
Server advertises Playlist Delta Updates with `CAN-SKIP-UNTIL=<SL>`
`<SL>` is skip limit, how old something must be to get skipped
Always at least 6 Target Durations

Client downloads full playlist at least once
Next time asks for delta using `_HLS_skip`.
Client merges delta with previous Playlist

# Structure
`EXT-X-VERSION:9` or higher
Current `EXT-X-MEDIA_SEQUENCE` tag shows which segments have been removed since the last update.
`EXT-X-SKIP` tag, which replaces
* segment URL lines added to Playlist before skip limit
* Every Media Segment Tag applied to a skipped URL line
* `SKIPPED-SEGMENTS` indicates # segments skipped
* After `EXT-X-SKIP` is the rest of the playlist

`GET playlist.m3u8?_HLS_skip=YES`

# Other
This was introduced in last year's os.  However, this year...

# Skipping `EXT-X-DATERANGE` tags
* Server can offer to skip older `EXT-X-DATEGRANGE` tags.
	* `CAN-SKIP-DATERANGES=YES`
* Client asks to skip DATERANGE with `_HLS_skip=v2`
* Update skips any DATERANGE added before skip limit
* `EXT-X-SKIP: RECENTLY-REMOVED-DATERANGES`
	* Lists DATERANGE tags removed within last `CAN-SKIP-UNTIL`
* `EXT-X-VERSION:10`

# Wrap up
* Provide Playlist Delta Updates for livestreams with large windows
* Latest OS offers v2 update that include `EXT-X-DATERANGE` tags
* Improves playlist reload performance, which increases reliability


