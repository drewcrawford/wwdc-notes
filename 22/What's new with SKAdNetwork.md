#skadnetwork

Attribution data sent back to advertiser while preserving the user's privacy.

* ad networks
* publisher apps
* advertiser apps
# SKAdNetwork
* Impression
	* Ad payload input to SKAdNetwork
* Engagement
	* User interactions with advertiser app
* Conversion
	* Success signal with attribution data

App can repeatedly poll the API to update the conversion value and capture various levels of engagement and return on ads spent.

We send teh postback to the ad network.

# Feature timeline
2.0 => privacy pereserving ad attribution
2.2 => Publisher apps to sshow custom ads
3.0 => Multiple postbacks
15.0 => developer copy

# Web to SKAdnetwork
[[Meet privacy-preserving ad attribution]]

# SKAdNetwork 4.0
## Hierarchical IDs and conversion values
crowd anonymity.  Term we us e to refer to privacy-presrving way in which SKADnetwork delivers attribution data.

We send less data if you have less users, etc.  Hide in the crowd.

In SKAdNetwork 4, we have a way to send more data while retaining privacy.

For this, we are changing the campaign identifier field.
2 digit campaign identifier
4 digit source identifier

Think about it as 3 hierarchical number.  a 2,3,and 4 digit number.

Two digits can represent ad campaign.  Third might represent location of user.  fourth might represent placement.

or, 2 for treatments.  3 for time.  4 for ad display specifics.

Ultimately what we wanted was to open this field up to you.  

Conversion value is changing.  Currently it's a 6 bit value.  We're introducing 2 conversion values.
Fine-grained value => 6 bits, same as current
coarse-grained value => 3 bits

coarse value requires fewer users than fine value.  Only one will be sent to advertiser.


* low => receive 2 digit component of source identifier
* medium => receive 3 digit component
* high => receive 4 digit source id

for conversion level:
* low => no conversion value
* medium => coarse conversion value
* high => fien conversion value

Can set `.sourceIdentifier` on `SKAdImpression`.

For advertised apps, call `.updatePostbackConversion`.  Now takes both coarse and fine-grained values.  

* source identifier is intended to be hierarchical
* conversion values have different granularities
* values can increase or decrease
* SKAdNetwork 4.0 postbacks have new fields


## Multiple conversions
After some time elapses, we send the postback.

As food truck's developer, I want to know the value of my advertising spend.
1.  Starts app
2. Update conversion value due to user action
3. Postback sent
4. Delivers donuts

However when there are further actions the postback has been sent.  Re-engagement (information) is lost.

Moving from a single postback to 3 postbacks.  
* postback 1: 0-2 days
* Postback 2: 3-7 days
* Postback 3: 8-35 days

First postback gets fine value
additional postbacks get coarse value
Winner and developer get additional postbacks

## Web to SKAdNetwork
App store works with SKAdnetwork to attribute installs.

We want to extend the same privacy to apps shown on webpages.

1.  User taps on link in safari for ad
2. Safari opens app store and lands on product page for the advertised app
3. app store fetches ad impression from ad network
4. user installs app
5. everything flows as normal

```html
<a href=""
   attributionDestination=""
   attributionsourceNonce=""
   </a>
```
* can be served on first party sites
* and embedded cross-site iframes

eTLD+1 component from attribution destination.
Add `.well-known/skadnetwork/get-signed-payload`, we post here

Post body will be JSON, will have `source_nonce`.  Use this to identify the impression.
We expect to receive a signed impression response.  `source_domain` is similar to `source_app_id`.
## adoption
* create ad links
* serve signed impressions
* Update postback servers for new source domain
* Publisher webpages
	* embed ad links


## Testing
At a high level, SKAdNetwork deals with impressions and postbacks.

Signing and configuring have been points of friction.  And opstbacks has been an area for improvement.

Unit testing framework within StoreKitTest.  

Create and configure the impression
Provide the public key
Call the validate method

Add test postback
Pay special note to postback URL, since this is where your postback will be sent to.  Remote or local server.
Add this to your test session using the `setPostbacks` method.
`testSession.flushPostbacks`.  

* postback sent over network to server

**Later this year**
* source identifier field
* fine and coarse conversion values
* Testing multiple conversions

# Wrap up
* more data, sooner
*  re-engagement, longer
* Link-driven, privacy preserving, attribution
* Implemented and tested


* https://developer.apple.com/forums/tags/wwdc2022-10038
* https://developer.apple.com/forums/create/question?&tag1=232&tag2=344030
* https://developer.apple.com/app-store/ad-attribution/
* https://developer.apple.com/documentation/storekit/skadnetwork
