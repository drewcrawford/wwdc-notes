#CloudKit 

Easy way to create great experiences by allowing users to effortlessly synchronize state.

CKConsole provides tools to work with your apps' schema and data.  Help you understand/debug your applications' schema and data.

# Hidden containers
In CK console, can choose which containers are hidden or visible.
* manage contianer visibility
* STatus reflected in xcode
* Applies to entire developer team

cloud kit console => manage contianers.  Toggle visibility.  

why do you want this?  I dunno?
# Act as iCloud
Understand why certain users hare having trouble with private databases.
* sign in as an iCloud account
* View data using CK console's query tools
* Debug development and production issues

When I sign in, the context of the console will change.  Cannot perform schema operations, etc.  Results of the query from my iCloud account.

* used for records, not schema
	* schema will halt the access session
* Debug development and production
* encrypted fields remain unreadable
* decryption only for original user
* sensitive data stays safe

# Zone sharing
Securely share collection of records
permissions
Public shares
private shares

* all records in the zone are shared
* Zone sharing record
* No sharing hierarchy

Public chared zone makes every record visible to everyone who has the shared zone.  Everyone with the short share can join the share.
World readable.  
Private shared zones have an additional layer of security => members must be a sharing participant.

Public zones have an additional permission option for readonly vs read-write share.

Short unique id that can be sent to join the share.  Shared zones can be joined in the console by using the accept shared record.  Any record created in that zone can be automatically shared.

# Wrap up
* Hide unneded containers
* act as iCloud
* zone sharing

* https://developer.apple.com/forums/tags/wwdc2022-10115
* https://developer.apple.com/forums/create/question?&tag1=107030&tag2=363030
* https://developer.apple.com/documentation/cloudkit/managing_icloud_containers_with_the_cloudkit_database_app
* https://developer.apple.com/documentation/cloudkit
