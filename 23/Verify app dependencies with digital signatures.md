Discover how you can help secure your app's dependencies. We'll show you how Xcode can automatically verify any signed XCFrameworks you include within a project. Learn how code signatures work, the benefits they provide to help protect your software supply chain, and how SDK developers can sign their XCFrameworks to help keep your apps secure.

Apps are developed using a range of SDKs.  

supply chain attacks, etc.

# Dependency signatures
We first generate a CD hash ("code directory")
we then sign this hash with your developer ID/cert.

made up of a private key that is used for code signing and apublic key taht is dstributed as part of a public signature.  Secure timestamp can be used to verify that it was signed at a point in time.

ex xcframework
[[Get started with privacy manifests]]

* verify dependencies
* suply chain integrity
* signature status

xcode now shows signature inspector.  This appears to be in the righthand panel, first tab?

seems like this is trust-on-first-use?

|                                      | apple developer program | self-signed | unsigned |
| ------------------------------------ | ----------------------- | ----------- | -------- |
| one identity per developer           | yes                     | no          | no       |
| automatic trust for new certificates | yes                     | yes         | no       |
| signature verification at build      | yes                     | yes         | no         |

these alerts will be rare,and will ensure you can build your app until you resolve the issue. xcode will offer to let you remove the xcframework from the project.

for self-signed, we will still compare sha-256 against the one that was previously used.  Alert if the signature has changed.


# App developers
xcode ensures that if this change happens... you will be notified automatically.

If you know the change is legitimate.  Then you can accept the change.
# SDK authors
Important for authors... to sign, for building trust, and distributing SDKs.

two types of identities that you can use.  ADP certs, and self-signed.

* valid certificate types
	* apple distribution and development certificates
* enterprise program
	* iOS distribution and app development certificates

attestation by apple
convenience
```bash
codesign --timestamp -v --sign "Apple Distribution: Truck to Table (UA527FUGW7)" BirdFeeder.xcframework
```
Codesigning recap.
secure timestamp, attested to by apple.
developers including your sdk in your app, will be able to confirm that the SDK is signed by you.
if you always sign the sdks you distribute, this new feature will help your sdk clients get more confident in their supply chain.

* for existing and new versions
* apple developer program or self-signed
* Inclusion in build scripts

app developers
* use signed SDKs
SDK authors
* sign your SDKs

# Resources
* https://developer.apple.com/documentation/Xcode/verifying-the-origin-of-your-xcframeworks
