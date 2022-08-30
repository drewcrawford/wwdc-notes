#signinwithapple 

# Sign in with Apple at Work & School
## Sign in with Apple
* fast and easy way to sign in with an Apple ID
* Create an accoutn with a simple tap
## managed apple IDs
* personalize a device
* Access apps and services
A user can tap "continue with apple"

When using an apple id, they will tehn see a signin in with apple welcome screen.
When using a managed apple id, they will see "sign in with apple" "at work and school"

The user sees thea ccount creation screen where they may edit their name, share/hide email, etc.

In organization context, app should understand who is logging in.  In this screen, the user is signing into slack at work.  In order to know which slack channels they should join, slack needs to know which user is signing in.  So slack has name and email.

When a user uses Sign in with Apple at work and school, name/email fields are always accessible.

If you're workign on a client-side app, here's example code.  credential.user, credential.fullname, credential.email

* works with primary managed apple ID on device
* supported web flow
* Email address may be empty

## Implementation
Alreeady have sign in with apple?
you're done, nothing to do
Check out WWDC sessions [[Get the Most out of Sign in with Apple]] [[Enhance your Sign in with Apple experience]]

# Roster API
students/teachers/staff can use managed apple ID to log in.

Setup users and classes at the start of the year.
Manual data entry and a barrier to adoption

Automatically import resources (e.g. users, classes)
Exposed via REST APIs
Secured through OAuth 2.0
managed by administrator and side manager accounts

* authorize sharing data for an organization
* follow industry standards
* recognizable UI design patterns
* choice of technology
* Oauth 2.0
	* scopes
	* consent screen for IT administrator

UI contains
 * requestor (app)
 * Scopes (what's accessed)

Implementation journey
* register app in developer protal
	* request scopes
* Implement OAuth 2.0 authorization
	* receive access token after IT admin consents
* Query roster API endpoints

certificates, identifiers, profiles => organizational and accoutn sharing => configure

Ruight now we offer 2 scopes: user access and class access.  
Return URLs: used  in oauth 2.0 authorization flow

## Oauth 2.0 workflow
IT admin => logs into your web app
Get /authorize to appleid.apple.com

parameters: client_id, redirect_uri,s tate, response_type, scopes
response: redirect to apple oauth consent screen

IT admin may be prompted to authenticate with their managed apple ID.  Presented with consent screen to confirm share data with your app.  Same content screen that we saw earlier.  

Retrieve your app's access token.  Body has standard oauth parameters, authorization code, etc.
Response will have the access toekn, expiration, refresh token, etc.

refresh flow.  

## Querying roster API endpoints.
Get classes, users, each class and user, etc.
Users endpoint has 3 parameters.  pageToken, limit, role parameter (optional).
Requires bearer (accessToken)

Response has standard user information such as given name, family name, etc.
id: same id you get from sign in with apple.  Email, role, also included.
nextPageToken => gets the next page.

Your app server can now query classes endpoint.  

This request retrieves a list of classes with a maximum of 200 to include in the response.  Page token parameter.  


# Access management
Provide management capabilities to organizations using organizational datassharing.

Which apps to sign into?  When using sign in with apple at work and school, IT admin centrally maanges access on behalf of all organizational users.  IT admin is responsible for entering safety and security of organization data.

Manage which apps can use
* sign in with apple at work & school
* organizational data sharing

configured in 
* asm
* abm

ABM => access management => sign in with apple.  Allow all apps, allow only certain apps.

Same situation in ASM.  

# Wrap up
* Implement sign in with apple
* Adopt roster API
* Give us feedback

[[Get the Most out of Sign in with Apple]]
[[Enhance your Sign in with Apple experience]]



* https://developer.apple.com/documentation/rosterapi
* https://developer.apple.com/forums/tags/wwdc2022-10053
* https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js/configuring_your_webpage_for_sign_in_with_apple
* https://developer.apple.com/documentation/authenticationservices/implementing_user_authentication_with_sign_in_with_apple

