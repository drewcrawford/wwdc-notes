# Overview
Allows an app to read a tag in a session up to 60 seconds.
* TAg must be presented within 1 minute
* Avilable since iphone 7

Background tag reading uses universal links
* requires iphone xs

# Protocols supported
NDEF
Native protocols
* ISO 7816
* FeliCa
* MIFARe
* ISO15693

# Syntax Changes
Returns a `Result<NFCISO7816ResponseAPDU,Error>` 

## More descriptive enums

# ISO15693 additions

Supports extended block operations
Includes security framework related operations
Supports raw packet sending
