# Review Schoolwork
* Share anything in assignments
* ASsign specific activities within an app
* View student progress

New features
# Introduce new API
More than 190k education apps.

Added a new file-based API.  

* Apps that interact with files
* Enables teacher insights
* Implement open-in-place.

`fetchActivity(for url: URL)`.  

If new to ClassKit, these classes are currently being used on CLSContext.

CLSActivity used to encapsulate progress on a particular file

* Duration (start, stop)
* progress.  Some value between 0 and 1, represents progress through the file
* primaryActivityItem.  File that can be edited.  Identifies which is primary
* additionalActivityItems.  

1.  CLSBinaryItem.  Any binary datatype.  Question on a quiz.
2.  CLSQuantityItem.  Any generic numerical value.  Numer of pages, wordcount
3.  CLSScoreItem.  Anything that can be a part out of a total.


# Walk through code sample
1.  Grab primary activity item if exists
2.  Create a new CLSQuanittyItem if not
3.  Update its value
4.  Set as primary activity
5.  Add progress
6.  Call stop
7.  Call save.  If we dont' call save on CLSDataStore, then no changes will persist.


# Test using developer mode
`com.apple.developer.ClassKit-environment` set to `development` and not `production`.  production is default.

Important to make sure to set this back to production after we're done testing.

Now we will open schoolwork app.  When this opens, we are presented with teacher UI.  

Add a file for the assignment in the teacher mode.  

Settings=>Developer=>ClassKit API=>Student

Notification is an indication that save was successful and time was started.

1.  Check if time is reported correctly
2.  Wordcount, this was the primary item.

Teacher mode
1.  Avg time and avg wordcount
2.  Row for every student on the assignment

don't forget to set entitlement back to production!

# Wrap up
* Adopt ClassKit API
* Give us feedback

[[What's new in ClassKit]]
[[Introducing ClassKit - 18]]

* https://developer.apple.com/documentation/classkit/clsdatastore/3727268-fetchactivity
* https://developer.apple.com/documentation/classkit/incorporating_classkit_into_an_educational_app
* https://developer.apple.com/documentation/classkit

