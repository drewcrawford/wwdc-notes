#applemusic

# Search enhancements
`/search/suggestions` replaces hints.
Access to same terms as hints.  However, response is different.

* Access top suggested results
	* `kinds=topResults`

Results you'll get might be noticeably different from search, as this response prioritizes speed.  Not a replacement for regular search.
# Relating resources
## What's a resource?
identifier => minimal amount of information to look a resource up in the API.

## relating
Returns a collection of resource identifiers
Faster than `include`
`relate[{resourcetype}] = {relationshipOne},...`

# Extending attributes
Every resource has a default set of attributes
Extended attributes are more expensive, less needed
`extend[{resourceType}]={attributeName},...`


# Using relationship views
* More loosely coupled than relationships
* Ideal for driving product pages
* May have `attributes` as well as data
* Only request-able on direct resource fetches

`?views={viewOne}`
or
`/view/{viewName}`

Evidently these are fixed.  Full list in the documentation.

# Storefront and city charts
Now have playlists for specific cities.  Can add playlist to libraries which add automatically.

# Wrap up
* Search and charts enhancements
* Relate, extend, views
* developer.apple.com/musickit

[[Cross reference content on the Apple Music API]]
[[Introducing MusicKit - 17]]

* https://developer.apple.com/musickit/


