# Content Steering review
Pre steering, there was a wild west.  e.g. multivariant playlist ordering.  If steering provider wants to specify fallback CDNs, they list every variant and order them in then playlist.  In case the client palyer enocuntered a failure, move on tot he next variant, with the failed variant penalized.

Even though the authoring server can control authoring, it onyl happens at the point of the client requesting multivariant playlist.  Once playlist is handed out, no way to change the fallback algorithm.  This is where content steering comes in.

Pathways are ordered by preference.  Streaming provider also hosts a steering server, generating manifests for each client player.  manifest defines rules of pathway priorities so the player can see rules.

Client will first exhaust variants within the current pathway, and then fall over to an additional pathway.  

content steering => send some regiosn to cdn1, other to cdn2.  But because of daylight shift, this becomes unbalanced.  So we steer the eruope region to the other cdn.

* `eXT-X-CONTENT_STEERING`.  Server-URI specifying where to request steering manifest updates.  Pathway-id: initial pathway to choose.
* each variant stream is given a PATHWAY-ID.  Contain the same streams, etc.

example manifest.
* `PATHWAY-PRIORITY` => list of pathways in preference order

[[Improve global streaming availability with HLS Content Steering]]
# Pathway cloning
Dynamically spawning fleets of CDN server in sreal time.  ex, new fleet of CDN3.  Advertise to existing clients.  Challenge is that the dynamically spawned CDN did not exist.

How do we tell clients about the emergence of a new CDN?  Pathway cloning
* compatible with content steering 1.2
* Announce new CDN to existing players
* Compact manifest definition
* ASsume pathways are structure-identical
* New pathway can be created by copying and modifying existing Pathway.

We clone not only its variant streams, but also reerenced media groups if any.  
Modify URIs.  

Most flexible way is to replace URI.  We ened to store a full set of URIs in steering manifest.  But we can usually do better.  Common to deploy multiple CDNs with same URI structure.  Only need to store the replacement of host and query parameters.

`PATHWAY-CLONES` field.  Defines a enw pathway clone from an existing pathway.
1.  BASE-ID => original pathway
2. `ID` new id
3. `URI-REPLACEMENT`
	4. `HOST` => new host
	5. `PARAMS` => new parameters.  These are merged with prior parameters.

`PER_VARIANT-URIs`
`PER-RENDITION-URIs`

# Steering server
Manage and orchestrate.  Apply rules at the buckets level.  Without maintaining client session states.  When client requests , it makes a GET request.
Server can generate a uniform random number out of e.g. 12 buckets.

When returning manifest, it adds bucket number to RELOAD-URI attribute.  So that appears in next request.

It carres the bucket number in its request parameter, server can apply steering rules.  

Suppose we want to steer 50/50.  Can write in terms of the bucket number, e.g. first 6 buckets go one way, etc.

Suppose we have new CDN3 dynamically.  Steering server can advertise using variant cloning.  One common question is, what to include?  Pathways that are not in the variant playlist.  But how can steering server know the set of pathways in the client's multivariant playlist?

One way is to include the set of pathways in the initial steering server URI as a query parameter.  Here we say `?pathways=CDN!+CDN2`.  It includes them in the SERVER-URI attribute as query parameter.

Steer to 3 pathways.  buckets 0-4: CDN 1, 4-8: CDN2, etc.  

# Wrap up
* adopt content steering
	* more versatile fallback
	* dynamic traffic steering
* Takea dvantage of pathway cloning
* Refer to latest IETF HSL spec
* Validate using apple HTTP live streaming tools
* hls-interest@ietf.org

[[Improve global streaming availability with HLS Content Steering]]


* https://developer.apple.com/forums/tags/wwdc2022-10144
* https://developer.apple.com/forums/create/question?&tag1=125&tag2=442030
* https://datatracker.ietf.org/doc/draft-pantos-hls-rfc8216bis/
* https://developer.apple.com/documentation/http_live_streaming/using_apple_s_http_live_streaming_hls_tools
