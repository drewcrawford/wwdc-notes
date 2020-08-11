* Unlock, lock, and start
* Secure
* Share with family and friends
* Manage keys remotely
* Agnostic to radio technology
* Offline capable
* Works with power reserve

This sesion is geared toward auto makers.
* owner pairing
* transactions
* server interfaces

## Owner pairing
Establishes a privileged and secure association using a short-range radio channel such as NFC.

1.  Prove ownership of car
2.  Initiate pairing
3.  Place iPhone near car's NFC reader.  As a fallback you can also provide an option to start pairing from within the car.
4.  Car key appears in wallet

## Transactions
Used to lock/unlock the car and authorize engine start.
NFC readers in door handle and dashboard

Place the device on the tray reader in the dashboard to authorize engine start.
Optimized for security and performance.  Express Mode default.  
iPhone and car can be offline.
Apple doesn't know when you use your car.

## Server interfaces
* share keys over Messages
* Car can be offline
* Apple doesn't know who you share your car key with
* Optional access levels

Unlock and drive
Access and drive -> limited.  You can define various levels.

## Key management
Manage owner and shared keys from iPhone or car
Keys removed stop working immediately, even if offline
Manage keys on your iPhone using iCloud
Easy to change owner device

# System architecture
Core features fully integrated into iOS.  Automaker apps are not erquired
Keys created and stored in Secure Element
AES and elliptic-curve cryptography
Offline design based on PKI
Automaker TSM not required

## what is a digital car key?
* elliptic-curve keyapir created in SE
* Private key never leaves SE
* Public key exported as x509

Applet
* key pair, car public key, secure mailboxes
* all car keys hosted in a single applet instance


## Owner pairing flow
1.  PAKE verifier
2.  Pairing password comes from owner app?
3.  Pairing.  PAKE protocol
4.  Cryptographic linking.  iOS gets car's public key, I think car gets device's public key
5.  Key activation/registration.  Phoen talks to apple, apple talks to automaker server
6.  Key attestation.  Device provides attestation to car.  

## Key sharing
1.  Owner sends invitation
2.  Friend creates new key.
3.  Friend cends identitity certificate chain back to owner
4.  Owner verification (todo).  
5.  Owner returns attestation, via IDS.
6.  If the car is offline, friend presents... with attestation
7.  If the car is online, the attestation can also be transmitted to the car during key registration.

## Lifecycle
* created during owner pairing or key sharing
* Transactions lock and unlock the car
* Suspension.  iCloud loss mode temporarily suspends car keys by locking the applet on the secure element
* Revocation.
* Deletion

## Certificates
Car certificate, Owner Key Certificate

Car certificate -> (optional) intermediate cert -> root certificate

This auto root PK is embedded in the owner key cert.
One instance CA per automaker on the secure element.  Subject CN indicates identifier.

When the owner signs out of iCloud or erases their device, instance CA is deleted.  This protects user provacy.

Instance CA is trusted by apple car key root certificate.
Car not require to carry apple's root certificate, the automaker root cert is sufficient to verify by signing apple's key

Owner's iPhone sends invitation with a template for friend's device to create a key.
Friend's key is trusted by its intermediate CA
external CA certificate

I'm not following all of this

## transactions
auth0 (fast), auth1 (standard), exchange.

We recommend using fast trasnactions to unlock a car.

### standard trasnaction
When a key is used the first time, we do a standard trasnaction.  sequence diagram:

car   -> auth 0 car identifier, ephemeral key -> device
 <- ephemeral key from device <-
 car -> auth1: car signature -> device
 <- encrypted: key identifier, device signature <-
 -> encrypted: EXCHANGE: read ->   (read attestation package)
 (derive fast keys...)<- encrypted: friend key attestation
 
 ### transactions
 Fast transaction
 * device authentication
 * No tracking possible
 * Integrity and confidentiality

Standard trasnaction adds
* mutual authentication
* forward secrecy


# radio technologies
## NFC
Based on standard NFC reader
Enhanced Contactless Protocol (ECP)
enables a fully automatic NFC experience
Identifiers available before transaction starts
* reader type
* automaker

efficient reader polling

## ultrawideband (future)
Enables the best possible solution
Use with iPhone in bag or pocket
Common key management architecture
Specification in development

# server integration
Servers are required for remote key management.

1.  Establish connectivity to apple.  Testing, production.
2.  Exchange certificates
3.  Implement server interfaces.  
4.  Provide artwork for Wallet
5.  Connect to automaker app.  
# automaker apps
* provid