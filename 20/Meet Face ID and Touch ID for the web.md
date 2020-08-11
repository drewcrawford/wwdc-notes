#security

# Web Authentication
* Relying party (RP).  Essentially means your website
* Public key credential
* Authenticator – security key, iPhone, etc.
* Attestation – performed by authenticator

WA provides 
* strong authentication
* Phishing-resistant
	* Website-scoped credential
	* Authenticator-bound credential – can't be exported out of authenticator
# Platform authenticator
* Face ID and touch ID
* secure enclave
* Multi-factor authentication in a single step by combining both

## Attestation: optional
* secure way to ask the device manufacturer that the device is real and really has the features requested.
* Apple attestation does this in a way that you can't track the user as a website

# Adopting Face ID and touch ID

## API Surface
```js
//check if feature is available
PublicKeyCredential.isUserVerifyingPlatformAuthenticatorAvailable()
//create a new credential
navigator.credentials.create()
//use an existing credential
navigator.credentials.get()
```

## Flow
```js
const isAvailable = await PublicKeyCredential.isUserVerifying...()
```

Let your user sign in.  Maybe let your user enable or optin, banner, etc.

```js
const options = {
	publicKey: {
		rp: {name: "example.com"},
		user {
			name: "john.appleseed@example.com",
			id: userIdBuffer,
			displayName: "John Appleseed"
		},
		publicKeyCredParams: [{ type: "public-key", alg -7}],
		challenge: challengeBuffer,
		authenticatorSelection: { authenticatorAttachment: "platform"}, //this is the key!
		attestation: "direct" // optional for attestation
	}
};
const publicKeyCredential = await navigator.credentials.create(options);
```
## Response?

```js
PublicKeyCredential {
	id: "someidstring",
	rawId: ArrayBuffer,
	response: AuthenticatorAttestationResponse {
		clientDataJSON: ArrayBuffer,
		attestationObject: ArrayBuffer
	}
	...
}
```
# Best practices
1.  Validate all metadata
2.  Validate the attesetation (if desired)
3.  Save credential ID and public key data
4.  Set a server-side cookie (optional)

# Sign-in

```js
 const options = {
 	publicKey: {
		challenge: challengeBuffer,
		allowCredentials: [{
			type: "public-key",
			id: credentialIdBuffer,
			transports: ["internal"]
		}]
	}
 }
 ```
 
 ```js
 PublicKeyCredential {
 	id: "str",
	rawId: ArrayBuffer,
	response: AuthenticatorAssertionResponse {
		clienDataJSON: ArrayBuffer,
		authenticatorData: ArrayBuffer,
		userHandle: ARrayBuffer,
		signature: ArrayBuffer,
	}
	...
 }
 ```
 
 ## Processing the sign in response
 1.  Validate the user ID and the credential ID
 2.  Validate all other metadata
 3.  Verify the signature

Not sure I trust this order personally

1.  Feature detection
2.  User onboarding
3.  signin

# Best practices
Provide faceid and touchid as a way to sign in faster, not the only way
Use feature detection instead of UA strings
Call `.create()` and `.get()` within user activated events
Remember enrolled devices
Provide a different user expensirece for platform authenticators and security keys
