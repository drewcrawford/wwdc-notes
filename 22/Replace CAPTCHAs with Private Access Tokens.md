How your apps and websites can work with fraud prevention providers to reduce the need for captchas.
# meet Private Access Tokens
If you signed up for a new account on a website or tried tos ign in with an existing account, you've encountered captchas at some point.

The reason these exist is to prevent fraudulent activity.
You don't want a server to be overwhelmed by fraud.  Accounts/products come from legitimate users, but other attempts may be attackers or bots.
The common tools to prevent fraud often make it harder for people to use your app or website.  Finding the right balance between ux and fraud is a challenge.

problems
* user experience
* IP address tracking
* accessibility

There's a better way.  Even if someone interacts with your app for the first time, they've already performed many actions hard for a bot to imitate.

they've launched a codesigned app, they have an apple id, etc.  private access tokens allow your servers to automatically trust clients.  new in iOS 16 and macos ventura.  Before explaining how these work, let's show them in action.

Privacy pass protocol
* privacy pass standard from IETF
* PrivateToken HTTP authentication
* RSA blind signatures
* Servers cannot track client identity

1.  Client accesses the server, the server sends back a challenge.  This specifies a token issuer that is trusted.
2. Client contacts the iCloud attester.  And sends a token request.  This request is blinded, so it can't be linked to the server challenge.  Tester performs deviced attestation using certificates stored in the secure enclave.  Verifies the account is in good standing.
3. Attester can perform rate limiting to determine if compromised, farm, etc.
4. Sends request to the issuer.  When token issuer gets the request, it doesn't know anything.  But it signs the token from the attester.  
5. Client receives the signed token and transforms it in a process called unblinding, so the original server can verify it.
6. Client presents the signed token to the server.  Server can check that the token is signed by the issuer, but can't identify/recognize the client.

# Prepare your servers
3 steps.
1.  Select a token issuer
2. Send PrivateToken challenges
3. Validate client token responses


## Selecting an issuer
* token issuers offered by CDNS
* issuers sign up at register.apple.com
* issuers work with 100+ servers

use fastly or cloudflare today.  Other capture providers, CDNs, etc, will also be able to run token issuers that work with apple devices.  

That the user of a specific issuer doesn't become a way to identify what websites a client is accessing.  So, each token issuer needs to be a large service that will work with at least hundreds of servers.

## send auth challenges
* ProviateToken HTTP authentication scheme
* Supoprted by CAPTCHA provider
* Sent directly by server
Requires a first-party domain (subdomain of your main URL)
Not a separate third-party domain embedded on your site.

## Validate responses
* check tokens with issuer public key
* prevent replay attacks

## Compatibliity
* support legacy clients
* Avoid blocking page load

# Test your app
## REquirements
* iOS 16 or macOS ventura
* sign in with an apple ID
* Use WebKit or URLSession
* Foreground apps only

## URLSession support
PrivateToken handled automatically
token failures passed as 401 responses

# Next steps
* Replace CAPTCHAs
* Enable token challenges on your server
* Use URLSession or WebKit in your app
* 



* https://developer.apple.com/forums/tags/wwdc2022-10077
* https://developer.apple.com/forums/create/question?&tag1=311&tag2=389030