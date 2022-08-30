Fundamentals of apple music API.  #musickit 

# MusicKit overview
announced in 2017.  Since then, we've made improvements.

* set of client framewroks and apple music API
* Discover and fetch Apple Music content
* Search the catalog
* Browse popularity charts
* For apple music subscribers
	* Play available content
	* Library, reocmmendations, recently played

client frameworks
* authenticate subscribers
* start/control playback
* Availability
	* Apple platforms
	* web
	* Android => SDK

On apple platforms, use MusicKit.
* authenticate subscribers
* start and control playback
* Discover and access apple music content
* native support for resources and pagination

[[Meet MusicKit for Swift]]
[[Explore more content with MusicKit]]

on web,
* bring apple music to your website and apps
* discover content
* authenticate subscribers
* start/control playback
* full access to apple music API
* web components
	* full-featured media player
	* easy to add and customize

on android
* integrate apple music into your andoroid apps
* authenticate subscribers
* start/control playback
* full access to apple music API

API
* json web service
* discover the apple music catalog
* personalized features
	* music library
	* recommendations
	* recently played history
# Getting access
* applications on apple platforms
	* Enable MusicKit service for your app
	* app ID section of apple developer portal
* other platforms
	* Obtain developer token on apple developer portal
	* request/download a private key
	* generate and sign a JSON web token

Headers
* alg -> signing algorithm, es256
* kid => key identifier.
Claims
* iss -> use your team ID
* iat -> issued at, seconds since the epoch
* exp -> expiration.  max of 6 months from issue
* origin -> for MusicKit on the web

Supply a valid token in the `authorization` header for all requests.


# Requesting resources
* content is modeled as resources
* resources have different types
* Resources can be discovered via endpoints or fetched using their identifier

content can differ between regions

[[Cross reference content with the Apple Music API]]

Support for localization is possible using language parameter `l`.  see documentation.

Attributes.  
`playParams` indicates if enabled for playback based on subscriber status etc.

ARtist artwork is now available in apple music API.
new and existing apps can add support for images.
Load the same way as other artwork.
Can load of any requested size.

## Extended attributes
* resources have a set of default attributes
* some resources have extended attributes
* appear in the resources attributes map alongside their default attributes.

## including relationships
Request metadata with the `include` parameter.
some resource types have additional relationships
Only include necessary relationships and metadata.

[[Explore the catalog with the Apple Music API]]


# Limits and pagination
Resources appear ina  data array.  Whenr esources are small, all resources will appear in a single response.

May select your own limit from 1-some max size in documentation, varies per relationship.

`next` page will appear as a sibling.  When resource is exhausted, `next` will be missing.

Does not reflect page size for request.  need to use `limit` if you want a different page size.  Tryign to calculate your own offset is not a good idea.
# Search the catalog
* Find content using a search term
* Support for multipel content types
* `meta` for relevancy of heterogeneous objects.
* `next` is for each type?

# Personalized features
* apple music library
	* view and search for content
	* add content and create playlists
* recommendations
	* surface the user's recommended content
* recently played
	* rediscover recently played music

available for users with an active subscription
Require authentication specific to your app
User must grant permission to share access

Music user token.
`music-user-token` for personalize drequests
* specific to application and device
* **must not be shared across devices**
* Can become invalid or expire
* requires refreshing
* managed automatically on apple platforms and web

# Wrap up
* integrate apple music into your apps
* client frameworks available for multiple platforms
* Use apple music API to access and find content
* Personalized features available for subscribers
[[Explore more content with MusicKit]]

* https://developer.apple.com/forums/tags/wwdc2022-10148
* https://developer.apple.com/forums/create/question?&tag1=26&tag2=173&tag3=402030
* https://developer.apple.com/documentation/musickit
