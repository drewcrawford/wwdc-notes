#appstore

# Automating common tasks
* setting up your team
* provisioning your app
* managing your beta testers
* financial reports

# more open

* Downloadable OpenAPI specification file
* Swagger-UI
* Improved documentation

# App Metadata

## create a new version

Each app has a relationship to "app store versions".

1.  Can look up the app id.
2.  Create a new app store version
3.  Include a relationship to the app, using the app id we just looked up.

## set price tier
App price has a price tier.
Each tier has a price point, one for each app store territory.

1.  get the current price schedule for an app `?include=priceTier`.
2.  Make a `patch` against the app to make an atomic operation on multiple prices.
3.  We use temporary ids with `${new-price-1}` to indicate the resource does not exist.  We will define them in the same request.
4.  Define each price.  These are linked to the tier.
5.  Put the new prices into the `included` array.  Order doesn't matter, but temporary ids should match 1:1.

If your app is released, these changes will take place immediately.

## Edit app and version metadata.

web: "App Information"
Also version-level information.

Note that each app info has a reference to the app info localization, for each locale.
similar for versions, we have an "app store version localization".

Lots of screenshot stuff.

1.  Get the app store version localization
2.  Get preview sets in a localization
3.  Post to `appPreviewSets`
4.  `previewType` 
5.  Link to localization by id

## asset uploads
Various stuff, geojson, screenshots, etc.

1.  Create an app preview reservation.  Include name of file, and the filesize.  Link to preview you've created
2.   Will create a set of operations.  For small assets, this is simple, just upload to our servers.
3.   For large assets, we will send back multiple operations.  You take these operations and split your asset into multiple parts, one part for each operation.  Use the length and offset to determine the range of bytes in the file associated with each part.
4.   Always be prepared to send multiple operations.
5.   Retry parts that failed.
6.   Patch the preview URL and indicate `uploaded:true` as well as the md5 checksum.

Asset processing is async, you can use get requests to poll status.

## Associate build with version
1.  Look up a build ID
2.  Patch build relationship to add to the version

## submit to review
1.  Post to v1/appStoreReviewDetails with phone number, etc.
2.  create an "app store version submission".  


# Power and Performance Metrics and Diagnostics

## power/performance
High-level view of how to acces this data.  We have a whole session devoted to this feature.
[[Identify Trends with the Power and Performance API]]

`perfPowerMetrics` off the app url.  Need a custom `Accept` header.

Can fetch metrics for a particular build as well.  

## disk writes / diagnostic signatures
disk writes: we have a type for this.  Sorted by % contribution to total disk writes.  Seems to be called `diagnosticSignatures`.




https://developer.apple.com/app-store-connect/api/
https://developer.apple.com/documentation/appstoreconnectapi
