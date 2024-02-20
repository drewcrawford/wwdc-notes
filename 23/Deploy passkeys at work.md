Discover how you can take advantage of passkeys in managed environments at work. We'll explore how passkeys can work well in enterprise environments through Managed Apple ID support for iCloud Keychain. We'll also share how administrators can manage passkeys for specific devices using Access Management controls in Apple Business Manager and Apple School Manager.

# Passkeys

Replacements for passwords
faster, easier to sign in
secure and phishing resistant
available on all devices via secure syncing
industry standard

[[Meet passkeys]]


# Benefits of paskseys at work

How passkeys uniquely protect employee accounts at work.

common attacks on enterprise
* phishing
* credential theft
* 2fa bypass

2.9% of people click on phishing links

3 of 100 people click on phishing links
15 of 500 people
29 of 1k people, etc.

Recall, a successful phishing attack is the initial entrypoint for a full breach.  With passkeys, nothing to type.  No more phishing.  \

40% of breaches involve stolen credentials.  Passwords have a hash on the server.  Stolen and potentially cracked, etc.  Use the password from breach A to log into B,C, etc.  This makes server breaches a cycle for attackers.

With passkeys, there is no more credential theft from servers.  

You might think to use 2fa, but they're increasingly tricking users.

| type of 2fa        | attack   |
| ------------------ | -------- |
| sms                | phishing |
| totp               | phishing |
| push notifications | push fatigue         |

passkeys are not just another bandaid, they replace the broken primary factor with a new one.

With passkeys, the primary factor is no longer broken.  layering on a vulnerable 2fa adds no extra security.

compare password-creation experience vs passkey-creation experience.  it's faster and easier.  Let's look at the experience of signing back in.  

benefits:
* no phishing
* no credential theft from servers
* great UX. 

  
# Manage passkeys at work
While they want to deliver same great experience at work, you have new requirements.

* manage the apple ids used with iCloud Keychain and passkeys
* ensure passkeys only sync to managed devices
* Store passkeys created for work in iCloud keychain of managed accounts
* Prove to relying parties that passkey creation happens on managed evices
* Turn off sharing of passkeys between employees.

## Use managed aple ids

created in abm/asm.  These now support iCloud Keychain in sonoma, etc.

Users get all the benefits of using passkeys with all devices.  You manage accounts.  Passkeys stored in managed apple ids cannot be shared.

[[Do more with Managed Apple IDs]]

IT admins also wnat to control which devices passkeys are synced to.  ABM/ASM have new access management functionality.  Two different controls admins can use.

## control syncing

any device
managed devices only
supervised devices only

can ensure that passkeys used for work only sync to managed devices.  Must implement support.  Works out of the box with apple business essentials, etc.

How to make sure certain passkeys stay on work devices only?
* store passkeys created for work in iCloud keychain of managed accounts
* Prove to relying parties that passkey creation happens on managed devices.

[[Explore advances in declarative device management]]

Declarative device management has new enterprise passkey attestation information.  

## require creation on apple devices
You can restrict passkey creation to only managed devices, to pair with syncing controls.

available on iOS, iPadOS, macOS.  

how an organization would configure device?  MDM server sends configuration and identity asset to device and ensures it activates.

identity certificate provisioned from corporate CA server.
Later, user visits a website.  Website rqeuirests a passkey.  They can indicate they only create passkeys with enterprise attestation.  Device generates a new passkey, attests to it with certificat,e and returns to website.  Website can verify attestation.

Relying party only validates attestation at the time of passkey creation.  Otherwise, used without need for further attestation.  

[[23/What's new in managing apple devices|What's new in managing apple devices]]
### Example passkey attestation configuration - 11:07
```json
// Example configuration: com.apple.configuration.security.passkey.attestation

{
    "Type": "com.apple.configuration.security.passkey.attestation",
    "Identifier": "B1DC0125-D380-433C-913A-89D98D68BA9C",
    "ServerToken": "8EAB1785-6FC4-4B4D-BD63-1D1D2A085106",
    "Payload": {
        "AttestationIdentityAssetReference": "88999A94-B8D6-481A-8323-BF2F029F4EF9",
        "RelyingParties": [
            "www.example.com"
        ]
    }
}
```

### WebAuthn Packed Attestation Statement Format - 13:12
```json
// WebAuthn Packed Attestation Statement Format

attestationObject: {
    "fmt": "packed",
    "attStmt": {
        "alg": -7, // for ES256
        "sig": bytes,
        "x5c": [ attestnCert: bytes, * (caCert: bytes) ]
    }
    "authData": {
        "attestedCredentialData": {
            "aaguid": “dd4ec289-e01d-41c9-bb89-70fa845d4bf2”, // for Apple devices
            <…>
        }
        <…>
    }
    <…>
}
```

Look at packed attestation statement.

1.  Algorithm number.  Always set to -7 for ES256.
2. bytestring containing signature.
3. array with x5c certificate chain.  Each encoding in x509 format.

see standard docs.  Now administrators can create new attestations for work, etc.  Ensure passkeys get stored in iCloud keychain, not the user's personal apple id.

## Next steps
* use passkeys with managed apple ids
* manage what devices they sync to
* control creation with attestation

# Resources
* https://developer.apple.com/passkeys/
