 # Passwords
 * Difficult to use correctly
	 * Reuse
	 * Easy to guess
	 * Phishing
 * Additional steps mitigate risk
 * Adding steps reduces the chance someone will be able to access the account.  attacker won't automatically have access to other accounts.
	 * Verification codes
		 * SMS, email, push
		 * Generated (TOTP, ubikey)



 # Verification codes
 * Single use
 * Can still be phished

Usually via SMS.  Autofill made it easier to use.  One tap to fill the code into any text field.

SMS:
* Not very secure
* Depend on good cellular service
* Cost to send and receive

Messages can add up.

## Time-based verification codes
* Defined in RFC 6238
* Generated on-device
* VAlid for short time period
* Lower costs

Requires no communication, happens on-device.

Huge benefit in terms of security and UX.

Share a secret key along with other parameters, e.g. codes etc.

Displaying QR code on one device, etc.  Cumbersome process.

Autofill makes this shine.  Generated codes with a single tap.  Since codes are immediately available, your customers get a streamlined experience.

Securely backed up with keychain.  End-to-end encrypted.

Protected via faceID, touchID, or passcode.  Every device has highest level of security "supported by the operating system"

# iCloud keychain verification codes
Setup
* Add one-tap setup
* Use images for QR codes

Login
* Annotate verification code with text fields

## Add one-tap setup

`optauth://totp/Shiny:user@example.com?secret=foo&...`

iCloud keychain uses "issuer" to suggest account.

Link directly to iCloud keychain password manager with `apple-otpauth://`.

Put in an anchor tag on webpage, etc.  Link attribute to `NSAttributedString` or `open(url:)`.

On previous versions of iOS, hide setup buttons.

## Streamline verification code setup
* Use raster images.
* Safari uses on-device image analysis.
* Safari will offer to setup in right-click menu

## Login

`TextField.textContentType(.oneTimeCode` in swiftUI,
`textContentType`, in UIKit, etc.

 # Future of authentication
 
 Passwords are the traditional baseline.  
 
 Passwords + SMS
 Passwords + TOTP
 
 Also federated signin (delegate entire process).  Federated auth are based on same mechanisms but require tracking passwords.
 
 Can be more secure than traditional mechanisms.
 
 WebAuthn: most secure.  Uses public-key cryptography to keep accounts safe.  iOS 15 and macOS monterey have a preview.
 
 [[Move beyond passwords]]
 
 Domain-bound SMS verification codes.  `@github.com #965751`.  Way of binding to a particular domain.
 
 Autofill will only offer the code for the domain.  Same mechanisms as associated domains or universal links.
 
 # Next steps
 Adopt time-based verification codes
 Add domain bindings to SMS verification codes
 
 
 See article.