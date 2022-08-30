#localauthentication

# Authentication and authorization
Distinct but related.

authentication => is this the right user?
authorization => is the user allowed to perform a specific operation on a concrete resource?

Authentication enables authorization.

Authorize access to Secure Enclave keys
You never actually handle the key.  SE keys can be associated with an ACL.  Specifies the requirements for signing and decrypting.

Specify when a... is available.  After device unlock, etc.  For this example, let's say you want biometric authentication, and ensure the device has been unlocked.

1.  Request operation
2. Identify operation requirements
3. Satisfy requirements
4. Perform operation

First, we ahve a resource.  Secure enclave key.
Operation.  Sign.
Third: requriements.  Unlocked device, and biometric auth.


# Use local authorization APIs
How a flow like this can be implemented usign the current APIs.

## LAContext
Authentication and autohrization
* can be used to evaluate the user's identity
* Handles user interaction
* Interfaces with Secure Enclave

Can be used in combination with other frameworks.
Can be used to evaluate ACLs.

Authorize a signature operation
```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassKey,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecAttrApplicationTag as String: "com.example.app.key",
    kSecReturnAttributes as String: true,
]

var item: CFTypeRef? = nil
guard SecItemCopyMatching(query as CFDictionary, &item) == errSecSuccess, let attrs = item as? NSDictionary, let accessControl = attrs[kSecAttrAccessControl] else {
    throw .aclNotFound
}
```

```swift
let context = LAContext()
try await context.evaluateAccessControl(accessControl as! SecAccessControl, 
                      operation: .useKeySign, 
                       localizedReason: "Authentication is required to proceed")
```
Present at the appropriate time.  Once the ACL has been authorized, we can use it as part of our query for obtaining the reference for the key.  Append to the query.
```swift
let query: [String: Any] = [
    kSecClass as String: kSecClassKey,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecAttrApplicationTag as String: "com.example.app.key",
    kSecReturnRef as String: true,
    kSecUseAuthenticationContext as String: context
]

var item: CFTypeRef? = nil
guard SecItemCopyMatching(query as CFDictionary, &item) == errSecSuccess, item != nil else { 
    throw .keyNotFound
}
```
By binding to our private key reference, we ensure that executing will not trigger the operation.
```swift
let privateKey = item as! SecKey

var error: Unmanaged<CFError>?
guard let sgt = SecKeyCreateSignature(privateKey, self.algorithm, blob, &error) as Data? else {
    throw .signatureFailure
}
```

* offers a great deal of flexibility
* Can be used in combination with other frameworks e.g. #security 
* Increases the complexity of your code
* Requires careful orchestration
It can be the right toolf or your app

If all you need is a way to authorize access, then you may want to trade out some flexibility for the simpler API.
# Streamline your app
This year we're adding new API.
* Higher level API focused on authorization
* Builds on top of existing abstractions
* Supports common authorization flows
* Introduces the `LARight`.
Authorize operations on application defined resources.  ex, use it to gate access  to the user profile section of your application.
By default, rights aer protected by authentication requirements
Can choose to associate your rights with more general requirements.


```swift
// LARight: Basic usage

func login() async {
   self.loginRight = LARight(requirement: .biometry(fallback: .devicePasscode))
   do {
       try await loginRight.checkCanAuthorize()
   } catch {
       navigateTo(section: .public)
       return
   }
   do {
      try await self.loginRight.authorize(localizedReason: self.localizedReason)
 navigateTo(section: .protected)
   } catch {
      showError(.authenticationRequired)
   }
}
```
Lifecycle.
1.  `.unknown`
2. authorize() => `.authorizing`
3. success => `.authorized`
4. failure => `.notAuthorized`
5. CAn move from authorized to `.notAuthorized`, when your application explicitly deauthorizes, or when the instance is deallocated.

**Keep a strong reference to your LARight instance to preserve its authorized state**.

## Observing the authorization state
* state can be queried directly
* Transitions can be observed
* Through the state property using KVO/Combine
* Through the NotificationCenter
* `.didBecomeAuthorizedNotification`
* `didBecomeUnauthorizedNotification`

```swift
// LARight: Basic usage

func login() async {
   self.loginRight = LARight(requirement: .biometry(fallback: .devicePasscode))
   // ...
   do {
       try await self.loginRight.authorize(localizedReason: self.localizedReason)
  navigateTo(section: .protected)
   } catch {
       showError(.authenticationRequired)
   }
}

func logout() async {   
   await self.loginRight.deauthorize()
}
```

* authorize operations on application defined resources
* Have a lifecycle that is ultimately tied to the runtime
* Need to be created and configured on every session

## Persistence
`LARightStore`.  When persisted, rights are activated by the secure enclave key.  Thisi helps us ensure that authentication requirements remain immutable.

You can access the private key that backs your write and use it to perform protected cryptographic operations such as encryption, signature, key exchanges.
Public key is accessible and can be used for encryption, verify.  This key can be exported.

Private key operations are only allowed after the right was authorized.  Public key operations are always allowed.

Can store a single mutable secret.  Secret is associated with an ACL matching the authentication requirements.  Only accessible once right is authorzed.

## summary
* obtained using LARightStore
* Configured only once (immutable)
* can be used across application sessions
* Bound to a specific device
* Backed by a secure enclave key (EC P256)
* available to you
* Can be used to protect a single immutable secret
```swift
// LAPersistedRight: Retrieval and private key usage

func generateClientKeys() async throws -> Data {
    let login2FA = LARight(requirement: .biometryCurrentSet)
    let persisted2FA = try await LARightStore.shared.saveRight(login2FA, identifier: "2fa")
    return try await persisted2FA.key.publicKey.bytes
}

func signChallenge(_ challenge: Data, algorithm: SecKeyAlgorithm) async throws -> Data {
    let persisted2FA = try await LARightStore.shared.right(forIdentifier: "2fa")
    let localizedReason = "Biometric authentication is required to proceed"
    try await persisted2FA.authorize(localizedReason: localizedReason)
    return try await persisted2FA.key.sign(challenge, algorithm: algorithm)
}
```

Provide a unique identifier.  Useful for the next time i need to fetch the right, etc.

After right is authorized, use the private key for protected operations.

# Wrap up
* authentication enables authorization
* LAContext support for authorization flows
* Streamline your authorization code with `LARights`.



* https://developer.apple.com/forums/tags/wwdc2022-10108
* https://developer.apple.com/forums/create/question?&tag1=151&tag2=398030
* https://developer.apple.com/documentation/localauthentication
* https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys/storing_keys_in_the_secure_enclave
