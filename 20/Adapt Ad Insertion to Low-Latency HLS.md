#hls 
In a regular live HLS stream, new segment added every Target Duration
Client reloads Playlist every Target Duration
Playlist devices current window of content

* first segment is earliest seekable point
* End of last segment is live edge of presentation

# Live HLS
At some point, segments slide out of the playlist.

Note that if segments are 6 seconds, then each segment has a publishing delay of 6s.

## Low-latency HLS and partial segments
Like looking through a microscope at the live edge of the presentation.
Each regular segment has a series of shorter partial segments.  Same media, just lower latency.

# Ad insertion (regular HLS)
* program source feed marks ad avails
* Packager splits segments at ad avail boundaries, so that the ad starts exactly on a boundary
* Ad decisioning chooses ad from inventory
* Packager replaces program segments with ad segments
* seprarated by DISCONTINUITY tag
* New ad segment added to Playlist every Target Duration
* Until end of ad/program/resumption

## LL HLS
similar, but with partial segments as well.
Packager needs to maintain same LL partial segment behavior from program content.

* mostly the same
* Biggest difference: ads are spooled out as partial segments
* Parent segment follows Partial Segments
* Same segmentation otherwises
* Blocking Playlist Reload for both program and ad Partial Segments

[[Reduce latency with HLS Blocking Playlist Reload]]

## Preload Hinting for ads

[[Discover HLS Blocking Preload Hints]]

Every Playlist update must provide URL of upcoming Partial Segment
`EXT-X-PRELOAD-HINT`
Ad content is no exception
Hint requests for live content must block until Partial segment is ready
* but blocking not required for prerecorded content like ads

## Early return from ad
Usually, you see this when something really exciting happens from live broadcast during an ad break, and the producer wants to cut back.

* Program content returns before end of ad
* Ad ends at last published Partial Segment
* Current ad Parent Segment ends at same place
* Followed by DISCONTINUITY, followed by program content
* `EXT-X-PRELOAD-HINT` switches from next ad part to next program part

# wrap up
* One big difference for server-side ad insertion in LL HLS
* Ads are spooled out as Partial Segments every Part Target Duration
* Blocking Playlist Reload for both ad and program content
* Preload hinting for both ad and program content
* But hint requests for pre-loaded content do not need to block
* 