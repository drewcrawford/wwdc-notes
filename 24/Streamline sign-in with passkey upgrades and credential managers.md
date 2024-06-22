Learn how to automatically upgrade existing, password-based accounts to use passkeys. We'll share why and how to improve account security and ease of sign-in, information about new features available for credential manager apps, and how to make your app information shine in the new Passwords app.
### Offering a passkey upsell - 0:01
```swift
// Offering a passkey upsell
func signIn() async throws {
    let userInfo = try await signInWithPassword()
    guard !userInfo.hasPasskey else { return }
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(relyingPartyIdentifier: "example.com")
    guard try offerPasskeyUpsell() else { return }
    let request = provider.createCredentialRegistrationRequest(
        challenge: try await fetchChallenge(),
        name: userInfo.user,
        userID: userInfo.accountID)
    do {
        let passkey = try await authorizationController.performRequest(request)
        // Save new passkey to the backend
    } catch {
        // ...
    }
}
```

### Automatic passkey upgrade - 0:02
```swift
// Automatic passkey upgrade
func signIn() async throws {
    let userInfo = try await signInWithPassword()
    guard !userInfo.hasPasskey else { return }
    let provider = ASAuthorizationPlatformPublicKeyCredentialProvider(relyingPartyIdentifier: "example.com")
    let request = provider.createCredentialRegistrationRequest(
        challenge: try await fetchChallenge(),
        name: userInfo.user,
        userID: userInfo.accountID,
        requestStyle: .conditional)
    do {
        let passkey = try await authorizationController.performRequest(request)
        // Save new passkey to the backend
    } catch {
        // ...
    }
}
```

### Modal passkey creation (web) - 0:03
```swift
// Modal passkey creation
const options = {
    "publicKey": {
        "rp": { ... },
        "user": {
            "name": userInfo.user,
            ...
        },
        "challenge": ...,
        "pubKeyCredParams": [ ... ]
    },
};
await navigator.credentials.create(options);
```

### Automatic passkey creation (web) - 0:04
```swift
// Automatic passkey creation
let capabilities = await PublicKeyCredential.getClientCapabilities();
if (!capabilities.conditionalCreate) {
    return;
}
const options = {
    "publicKey": {
        "rp": { ... },
        "user": {
            "name": userInfo.user,
            ...
        },
        "challenge": ...,
        "pubKeyCredParams": [ ... ]
    },
    "mediation": "conditional"
};
await navigator.credentials.create(options);
```

### New Credential provider Info.plist keys - 0:05
```xml
<dict>
    <key>NSExtensionAttributes</key>
    <dict>
        <key>ASCredentialProviderExtensionCapabilities</key>
        <dict>
            <key>ProvidesPasswords</key>
            <true/>
            <key>ProvidesPasskeys</key>
            <true/>
            <key>SupportsConditionalPasskeyRegistration</key>
            <true/>
            <key>ProvidesOneTimeCodes</key>
            <true/>
            <key>ProvidesTextToInsert</key>
            <true/>
        </dict>
    </dict>
</dict>
```
# Resources
* https://support.apple.com/en-us/HT213305
* https://developer.apple.com/documentation/bundleresources/information_property_list/nsextension/nsextensionattributes/ascredentialproviderextensioncapabilities
* https://developer.apple.com/documentation/authenticationservices
* https://developer.apple.com/documentation/authenticationservices/connecting_to_a_service_with_passkeys
* https://developer.apple.com/passkeys/
* https://developer.apple.com/documentation/authenticationservices/public-private_key_authentication
* https://developer.apple.com/documentation/authenticationservices/public-private_key_authentication/supporting_passkeys
