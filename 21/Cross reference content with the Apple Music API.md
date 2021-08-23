 #applemusic 

Provides APIs for building apple music experiences across platforms.

# Cross storefront equivalency
Look up "storefront equivalent" IDs.  Storefront (region) scopes the catalog.  
Atlerante versions of content may be available with different IDs between storefronts.

Fetching content by same ID in target storefront may not work.  Sharing links with a user in a different storefront may not work.

`v1/catalog/{targetStoreFront}/{resourceType}?filter[equivalents]={contentID}`

```
GET /v1/catalog/jp/albums?filter[equivalents]=1500951604

{
  "data": [
    {
      "id": "1500954334",
      "type": "albums",
      "href": "/v1/catalog/jp/albums/1500954334",
      "attributes": {
        "artistName": "レディー・ガガ",
        "name": "Chromatica",
        "playParams": { ... },
        ...
      },
      "relationships": { ... }
    }
  ]
}
```


# Explicit/clean mapping
User preference or regional regulations may not allow explicit content.

`GET /v1/catalog/us/{resourceType}?filter[equivalents]={contentID}&restrict=explicit`
albums, song, music video

```
{
  "data": [
    {
      "id": "1541243687",
      "type": "albums",
      "href": "/v1/catalog/us/albums/1541243687",
      "attributes": {
        "artistName": "Megan Thee Stallion",
        "name": "Good News",
        "playParams": { ... },
        "contentRating": "clean",
        ...
      },
      "relationships": { ... }
    }
  ]
}
```



# Library/catalog lookup
Catalog and library have different key space.  Easy way to determine if content exists in the user's library.
`relate=catalog`
```
GET /v1/me/library/songs/{libraryID}?relate=catalog

{
  "data": [
    {
      "id": "i.eoD88xWsk6X5Rl3",
      "type": "library-songs",
      "href": "/v1/me/library/songs/i.eoD88xWsk6X5Rl3",
      "attributes": { ... },
      "relationships": {
        "catalog": {
          "href": "/v1/me/library/songs/i.eoD88xWsk6X5Rl3/catalog",
          "data": [
            {
              "id": "1552791228",
              "type": "songs",
              "href": "/v1/catalog/us/songs/1552791228" } ] } } } ] }
```

vice versa

```
GET /v1/catalog/us/songs/{catalogID}?relate=library

{
  "data": [{
      "id": "1552791228",
      "type": "songs",
      "href": "/v1/catalog/us/songs/1552791228",
      "attributes": { ... },
      "relationships": {
        "library": {
          "href": "/v1/catalog/us/songs/1552791228/library",
          "data": [
            {
              "id": "i.eoD88xWsk6X5Rl3",
              "type": "library-songs",
              "href": "/v1/me/library/songs/i.eoD88xWsk6X5Rl3" } ] } } } ] }
```

* Personalized
* Requires valid user authentication token
* More expensive than catalog fetch


# Lookup content by UPC
* UPC is used to represent music as an entire physical or digital product
* Match Apple Music catalog from other sources

`?filter[upc]={UPC}`

```
GET /v1/catalog/us/albums?filter[upc]=00602435945422

{
  "data": [
    {
      "id": "1556669854",
      "type": "albums",
      "href": "/v1/catalog/us/albums/1556669854",
      "attributes": {
        "artistName": "Drake",
        "name": "Scary Hours 2",
        "upc": "00602435945422",
        "playParams": { ... },
        "contentRating": "explicit",
        ...
      },
      "relationships": { ... }
    }
  ]
}
```

# Wrap up
* Lookup content
	* Across storefronts
	* Different content ratings
	* Between catalog and library
	* Using UPC

[[Explore the catalog with the Apple Music API]]
[[Meet MusicKit for Swift]]

* https://developer.apple.com/musickit/

