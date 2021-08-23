MDM protocol
Standard across mobile device management.

Imperative, reactive.
Multiple round-trips.  
Each change is compounded.

**The future of device management is declarative management**.

Bringing policy management to the device.

Allows the device to be autonomous and proactive.

Servers no longer need to poll devices.

Increased performance and scalability.  A new paradigm, but the same protocol.  New functionality is built into existing MDM protocol.

# Declarative data model
* Declarations
	* Represent policy
	* Used for accounts, settings, and restrictions
	* Common or specific use
* Status channel
* Extensibility

Type => which setting
identifier => GUID represented as a string.
ServerToken => unique revision of the declaration based on the identifier key, must be different for each revision.
Payload => Data specific piece of the declaration containing keys and values pertinent to the declaration type


Four types of declarations
* Configuration
	* Policies to be applied to the device.  ex accounts, settings, restrictions.  Similar to MDM profile payloads.
* assets
	* Represent references to ancillary data needed by configurations.  Shared item of large data or personalized one.
	* e.g. URL to server
	* Data can be specific to a user
	* One-to-many relationship with configurations
	* Can update assets without configurations.
* activations
	* Sets of configurations that the device will atomically apply
	* Many-to-many relationship between activations and their configurations
	* Configuration can have multiple activationsr eference it.
	* Predicates determine the activation state (when it's active/inactive)
	* Processed if predicate evaluates to true
	* For example, `device.model.family == 'iPad'`
	* Or `device.operating-system.version > '14'`
	* Predicates are re-evaluated when device state changes
	* Always processed if lacking predicate
* management
	* Represent properties of the overall management state
	* Organization information
	* Server capabilities
	* Conveys static information to the device

## Status channel
Declared state may not match actual device state.  e.g. passcode policy.  If user has to create a new password for the device.

Client reports to server
Server subscribes to status items
Status items are key-paths
`device.operating-system.family` etc.
Can be used as expressions in activation predicates

### Subscribe to status items
* Use status subscription configurations
* Device sends initial and subsequent status reports
* Status reports are incremental
* Declaration status always reported to server on change

ex, Server detects that the device has been updated.


## Extensibility
Essential to maintain compatibility between different versions of MDM.  

Device and server advertise what they support.  
Supported features advertised as well.
Supported payloads.
Server indicates support in management declaration
Client indicates support as specific status item

# Integration with MDM
* Integrated into the MDM protocol for enrollment, transport, and authentication
* Not disruptive
* Declarations and MDM commands and profiles co-exist
* Gradual adoption of declarations
* Unenrollment removes all declarative management
* Does not interfere with existing MDM behavior

## Activate declarative management

New `DeclarativeManamagement` command
Once turned on, can't be turned off
Synchronization of declarations
CheckIn request

Some request,s such as this status report also icnlude base64 encoded data.
When using the checkin request to synchronize declarations, there's a server response
* Response from the server
	* Declaration manifest
	* Single declarations

Migrate to declarative management
* Send profiles as configurations
* Profiles are now declarative
* Adopt now with iOS 15

# Example
* Server sends push notification
* Device sends `Idle` ServerURL request
* Server responds with `DeclarativeManagement` command
* Device activates declarative management
* Device sends `Acknowledged` request
* Server sends empty response
* Device starts declarative management

1.  Device sends `declaration-items` CheckIN request
2.  Server responds with manifest
3.  Device compares manifest to its current declaration list
4.  Device sends declaration fetch CheckIn request (new/changed items)
5.  Server sends response with declaration JSON object
6.  Device applies policy changes
7.  Device sends `status` CheckIn request

# Get started
* iOS 15
* iPadOS 15
* User enrollment (new onboarding flow, or iOS 13 flow)
* Configuration declarations for accounts, profiles, etc.

Activation declarations
* Simple activation

Asset declarations
* User identity
* User credentials

Management
* Organization details
* Server capabilities

Status items
* Declaration state
* Device properties


# Get started

# Wrap up
* Declarative data model
* Integration with MDM
* Example
* Get started

* https://developer.apple.com/documentation/devicemanagement
* 