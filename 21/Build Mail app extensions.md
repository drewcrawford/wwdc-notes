Walk you through how to build great mail app extensions.

#mailkit

* Security and privacy built-in
* Official APIs
* Notarized Mac app
* Can be bundled
* App store distribution

*Plugins will stop functioning in a future macOS release.*

# Compose
Full suite of mail extensions.  

In macOS monterey, there are 4 ways.
* Annotate recipient email addresses
* Present view controller
* Set additional headers (outgoing)
* Validate message before sending

Add new target to existing app.  New mail extension template.

Wizard lets you choose the extension type. Info.plist, you must specify an icon and tooltip in `MEComposeSession` dict.  We use this to display a toolbar button in the compose window.

Principal class
`MEExtension`
Protocol adopted by `NSExtensionPrincipalClass`
Provides handler for extension type

`handler(for session:)`
Return an instance.

Life cycle methods for compose window
`mailComposeSessionDidBegin`
Methods called based on user actions

All methods in `MEComposeSession` have a unique instance for every compose window.

`MEMessage` exposes various message details.  

`annotateAddressesForSession`.  

Provide a subclass of `MEExtensionViewcontroller` (required).
Return an instance in `viewControllerForSession:` method.

# Actions
Perform actions in incoming messages to help users manage their inbox.

* Mark message as read/flag messages
* Move to other standard mailboxes
* Apply color in message list

Message action compatibility
For action extensions, principal class must return a message action handler.

`MEMessageActionHandler`.  Must implement `decideAction(for message:)`.  

Few tings to note
* mail calls you for every new mail message received, before it's visible in the inbox
* The first time, it has only a subset of the message headers.  You can provide a decision based on available headers
* Once applied
* Visible in inboxes

In some cases, you will need the complete body and headers of the message.  In this case, you can call `invokeAgainWithBody`.  This will cause mail to fetch the complete message body before invoking your handler.

# Content blocking
Hooks into WebKit.  Allows extensions to block content based on HTML criteria.  e.g., URL.

Select "Content Blocker" in wizard.  

`MEContentBlocker`

Same syntax as safari content blockers.  Can use same rule-list.  See WebKit blog

return `contentRulesJSON() -> Data`.


# Message security

* Encode/decode encrypted messages
* Sign messages
* Viewing signatures

`MEMessageSecurityHandler`.

## Message encoding
* Composing messages
* Sending messages

As the message is composed, mail sends the message to the extension.  Extension can determine if it has the ability to sign/encrypt the message.

Mail highlights the lock/certificate icons.

When sender/recipient changes, mail calls `getEncodingStatus`.  

## Message sending

Mail takes RFC822 message data and pass it to the extension.  Extension signs/encrypts the message as needed, and returns RFC822 data back to mail.

Message security handler returns the encoded message.  When the message is viewed, mail will send encoded message data to the extension.  Extension decodes the message

Calls `decodeMessage(for messageData)`.  Extension returns `MEDecodedMessage`.  If not needed, return `nil`.

## Signatures and certificates

Mail uses some UX.  Mail allows extensions to provide own VC to render certificate information.

Extension can return `MEMessageSigner` with a label and `context` which is I guess passed to the VC.

# Mail app extensions
mail-app-extensions@apple.com
Developer forum

