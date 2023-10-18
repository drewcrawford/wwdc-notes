#shareplay 
Discover how to work with files and attachments in a SharePlay activity. We'll explain how to use the GroupSessionJournal API to sync large amounts of data faster and show you how to adopt it in a demo of the sample app DrawTogether.

# Transfer attachments
now transfer attachments as well.  Now when you drop an attachment, it transfers faster than ever.  Multiple layers to minimize the data needed to be transferred.  As fast as possible.  At a system layer.

User privacy in mind.  All this is done while making sure the data you're transferring is e2e encrypted.  We don't know about the content. 

For anything you want.  PDF, voice recording, annotation, etc.  Any content that is being collaborated on can be faster than ever.
# GroupSessionJournal
API design.

Actiosn you take on the journal affect everyone, etc.

observe attachment smutations via async sequence.  And add attachments.  Just make type conform to Transferable.

Likewise, if you remove the attachment, everyone's attachments will be updated.  That's pretty much it.  Before we dive into implementing this, let's talk about some big considerations.

100MB limit!!  To ensure that we think carefully about the UX.  Need to be fast, devices can only upload/download data so fast.  Always minimize the size of attachments being sent.

Larger attachments that aren't user generated should be served from your own servers.

Anythign you put on the journal wills tay around for as long as one person is still in the session.  Attachment will stick around even after the person leaves.
# Late joiners
Typically when a new person joins, your application will have to retransmit the state of the world to that person.  This is managed by application adopting `GroupSessionMessenger`.  Each device sends a message.  Pretty expensive.

Late joiners will receive the attachments with only one reupload?  Late joiners will receive the updated attachments without any (explicit?) reupload from others
# Adopt in DrawTogether


demo

# Wrap up
* explore implementing GroupActivities in your app
* Learn more about shareplay in [[Add SharePlay to your app]] and [[Build custom experiences with Group Activities]]



# Resources
* https://developer.apple.com/design/human-interface-guidelines/shareplay/overview/
* https://developer.apple.com/shareplay/
