#maps

We empower developers to create beautiful geolocation experiences.  mapkit and mapkit JS offerings.

Very "client-centric".  We listened carefully to your feedback.  To round out our ecosystem, introducing server APIs.

* geocoding
* reverse geocoding
* search
* ETA

Let you tackle a variety of usecases while integrating maps into your application.  Convert an address to geographic coordiantes: latitude and longitude.
reverse geocoding => from coordiantes to an address
search => enter a search string to discover places, POI, etc.
ETA => get a sense of how far your business is from them.

1.  Full stack architecture
2. Network performance
	3. Repetitive and reudndant requests?  Delegating to your server and doing it once on the backend will help your application consume less bandwidth
4. Power efficiency.  Processig is now delegated to your server using apple maps server APIs.

# Comic book store locator
For now, let's focus on building one of these contact cards. 

Since we don't have server APIs in this example, now our client app has to perform various actions on the address.  Client has to make multiple calls to various backend services.  e.g. client talks to apple maps.  This chattiness between a client and backend can adverseily impact performance and scale.  Using individual requests over cellular is inefficient.  Broken connectivity, incompelte requests.  Application must send, wait, and process data for each request, increasing the chance of failure.  Have to merge all responses on the client, spinner to the user.  Client device is using more bandwidth and power etc.

Architecture with server API.  Use your backend as a gateway to reduce chattiness between client and services. 

Your client makes one call to the server.  Server then does the heavy lifting to make appropriate API calls most suited for your user.

Use geocoding and ETA API to get distance to the store.

`GET/v1/geocode?q=123%20Main%20St`

In resopnse, can see latitude and longitude for addresses.  etc.

As you can see, there are more fields in the response, see docs

`/v1/etas?origin=&destination=`

choose to augment or overlay data with your own store information, like store hours.  Leverage different APIs to build your app.

# Authentication
All server APIs are authenticated.  If using mapkit JS, you're already halfway there, we use the same mechanism.

1.  Get private key from developer.apple.com
2. Add private key to your server
3. generate JWT token
4. Exchange for a maps access token from apple server
5. use the maps token from then on
6. refresh every 30 minutes

```
GET /v1/token
authorization: Bearer <maps_auth_token>
```

Keep in mind, maps *auth* token is not maps *access* token.  You get a 30 minutes access token to access the server API.

# quota
Pass maps access token along server API calls.
With great power, comes great responsibility.  DAta cap on how many API calls you can make.  25k service caller per day.
Calling mapkitJS and server APIs use the same quota, if you need more please ask us.

https://maps.developer.apple.com
When quota is exceeded (more than 25k calls), we'll start 429 too many requests.

ensure app experience degrades gracefully.

In rare circumstances, it's possible to get 429 "as well" (without quota?)  Use exponential backoff

# Wrap up
Rich set of apple server APIs
Full stack implementation
Designed for efficiency
25k calls per day, shared with mapkit JS



https://developer.apple.com/documentation/applemapsserverapi
https://maps.developer.apple.com/maps-server-api-playground
https://developer.apple.com/documentation/mapkitjs/creating_and_using_tokens_with_mapkit_js
https://developer.apple.com/documentation/mapkitjs/creating_a_maps_identifier_and_a_private_key
https://developer.apple.com/maps/mapkitjs/
https://developer.apple.com/maps/
