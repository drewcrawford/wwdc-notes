#CloudKit #coredata 

NSPersistentCloudKitContainer

Development water-cycle
* exploration
* analysis
* feedback

The goal of this cycle is to
* durably capture learning
* Use platform tools
* collect actionable feedback

Core Data
* NSPersistentCloudKitContainer
* Sample application

[[Build apps that share data through CloudKit and Core Data]]
[[Using Core Data with CloudKit - 19]]

# Related content
* xcode
* instruments
* device organizer

[[getting started with instruments]]
[[Diagnose Performance Issues with the Xcode Organizer]]


# Exploration
* facilitate learning
* Verify assumptions
	* what does this button do?
	* will it sync?
	* how much data is too much?

Posts
attachment
image data

shape
structure 
variance

generate 1000 posts button.  Wehn tapped, it generates a smample dataset of 1k posts with a short title.  Post tableview easily handles this level of data.

How can I explore a dataset of a different shape or size?

* algorithmic (data) generator
	* Insert 1k objects
	* Every field has a value
	* No fielsd have values
* You are a data generator
	* use code, sql
	* interact with an application

```swift
class LargeDataGenerator {
    func generateData(context: NSManagedObjectContext) throws {
        try context.performAndWait {
            for postCount in 1...60 {
                //add a post               
                for attachmentCount in 1...11 {
                    //add an attachment with an image
                    let imageFileData = NSData(contentsOf: url!)!
               }
            }
        }
    }
}
```

660 images in total.  At an average size of 10-20mb per image.  Generated dataset consumes over 10gb of data.

```swift
class TestLargeDataGenerator: CoreDataCloudKitDemoUnitTestCase {
    func testGenerateData() throws {
        let context = self.coreDataStack.persistentContainer.newBackgroundContext()
        try self.generator.generateData(context: context)
        try context.performAndWait {                     
            let posts = try context.fetch(Post.fetchRequest())
            for post in posts {
                self.verify(post: post, has: 11, matching: imageDatas)
            }
        }
    }
}
```

Generates over 10gb of representative data.

We can vbuild validation methods in the test methods that verify the generator behaves correctly.
```swift
func testExportThenImport() throws {
    let exportContainer = newContainer(role: "export", postLoadEventType: .setup)
    try self.generator.generateData(context: exportContainer.newBackgroundContext())
    self.expectation(for: .export, from: exportContainer)
    self.waitForExpectations(timeout: 1200)
}
```

I use the large data generator to popualte the container with the desired dataset.

I wait for the contianer to finish exporting the data.  In this specific test, I wait for up to 20 minutes to give the dataset time to upload.

maybe you noticed that this test appears to be doing a lot of waiting.
expectation helper method
```swift
func expectation(for eventType: NSPersistentCloudKitContainer.EventType,
                 from container: NSPersistentCloudKitContainer) -> [XCTestExpectation] {
    var expectations = [XCTestExpectation]()
    for store in container.persistentStoreCoordinator.persistentStores {
        let expectation = self.expectation(
            forNotification: NSPersistentCloudKitContainer.eventChangedNotification,
            object: container
        ) { notification in
            let userInfoKey = NSPersistentCloudKitContainer.eventNotificationUserInfoKey
            let event = notification.userInfo![userInfoKey]               
            return (event.type == eventType) &&
                (event.storeIdentifier == store.identifier) &&
                (event.endDate != nil)
        }
        expectations.append(expectation)
    }
    return expectations
}
```

 ```swift
func testExportThenImport() throws {
    let exportContainer = newContainer(role: "export", postLoadEventType: .setup)
    try self.generator.generateData(context: exportContainer.newBackgroundContext())
    self.expectation(for: .export, from: exportContainer)
    self.waitForExpectations(timeout: 1200)
    
    let importContainer = newContainer(role: "import", postLoadEventType: .import)
    self.waitForExpectations(timeout: 1200)
}
```

We created a new NSPersistentCloudKitContainer with empty store files.  Explore what happens when all this data is downloaded to a device.

How to feel how a dataset behaves in an application?  Sample application.

```swift
UIAlertAction(title: "Generator: Large Data", 
              style: .default) {_ in
    let generator = LargeDataGenerator()
    try generator.generateData(context: context)
    self.dismiss(animated: true)
}
```

Clear, concise, and easily understood.  

In this section, we explored the behavior of an application

* generators make it easy to explore datasets
	* tests
	* custom UI
	* CLI arguments
	* many more

# Analysis
In this section, we'll learn about tools and techniques to analyze how an application behaves with large dataset. 
* instruments
	* time profiler instrument
	* allocation instrument
* logs
	* application
	* cloudKit
	* scheduling
	* push notifications

In my test case, right click, profile.  Xcode will build the test and automatically launch instruments.

This test is taking awhile to run.  Let's skip ahead and see why.

I can see large data generator is spenidng time generating thumbnails.  Bug or feature?

```swift
func generateData(context: NSManagedObjectContext) throws {
    try context.performAndWait {
        for postCount in 1...60 {
            for attachmentCount in 1...11 {
                let attachment = Attachment(context: context)
                let imageData = ImageData(context: context)
                imageData.attachment = attachment
                imageData.data = autoreleasepool {
                    let imageFileData = NSData(contentsOf: url!)!
                    attachment.thumbnail = Attachment.thumbnail(from: imageFileData,        
                                                                thumbnailPixelSize: 80)
                    return imageFileData
                }
            }
        }
    }
}
```

Thumbnails are special.  They're computed on demand from the related image data.  This line is unnecessary.  I can just remove it.


```swift
func generateData(context: NSManagedObjectContext) throws {
    try context.performAndWait {
        for postCount in 1...60 {
            for attachmentCount in 1...11 {
                let attachment = Attachment(context: context)
                let imageData = ImageData(context: context)
                imageData.attachment = attachment
                imageData.data = autoreleasepool {
                    return NSData(contentsOf: url!)!
                }
            }
        }
    }
}
```

Rerun in instruments.  I don't see much ofa  change.  But after a few more seconds, the test completes.  It's faster than the previous run.  Let's see where test spent time.

We now spend 1/10th of the time.  Analyzing tests doesn't always uncover bugs, sometimes we just learn more about where the app spends time.  Either way it's valuable learning.

I'm curious to see how much meomry the test uses.  So let's try the Allocations instrument.

Even though this test executes quickly, it uses a lot of memory – over 10gb.  Tells me that nearly the entire dataset is being kept in meomry during the test run.

Expand the stacktrace on the right.  Usually an indication that the object was retained bya  fetch, autorelease pool, or object in test.

```swift
func verifyPosts(in context: NSManagedObjectContext) throws {
    try context.performAndWait {
        let fetchRequest = Post.fetchRequest()
        let posts = try context.fetch(fetchRequest)

        for post in posts {
            // verify post

            let attachments = post.attachments as! Set<Attachment>
            for attachment in attachments {

                XCTAssertNotNil(attachment.imageData)
                //verify image
            }
        }
    }
}
```

Note that this keeps the attachment and associated image data registered with the MOC.  A number of ways to resolve this, for example, in a tableview we could use a batched fetch to free images as we scroll over the post.  This test is too quickly for that to be effective.

Let's verify attachments instead, and only load object IDs.
```swift
func verifyPosts(in context: NSManagedObjectContext) throws {
    try context.performAndWait {
        let fetchRequest = Attachment.fetchRequest()
        fetchRequest.resultType = .managedObjectIDResultType
        let attachments = try context.fetch(fetchRequest) as! [NSManagedObjectID]

        for index in 0...attachments.count - 1 {
            let attachment = context.object(with: attachments[index]) as! Attachment

            //verify attachment
            let post = attachment.post!
            //verify post

            if 0 == (index % 10) {
                context.reset()
            }
        }
    }
}
```

this wont' capture any of the loaded objects uintil I ask.  And use `object(width id)` to load attachments as needed.

For every 10 attachments I validate, I reset the context, freeing cache state and associated memory.

 We use the "Created & destroyed" filter at the bottom to see objects that were created and subsequently destroyed.  We want to drill down into a specific allocation like that.
Allocation stacktrace.  Deallocation stacktrace.

Select the deallocation event.  I happen to know that this stacktrace means that NSManagedObjectCotnext is asynchronously deallocating the object that contained this blob.  Establish a high watermark for the test, so ti can run on systems with less memory.

## Logs
* application
* cloudkit
* push notifications

Lifecycle of a change
1.  CoreDAtaCloudKitDemo
2. asks dasd to export that data to cloudkit
3. if it's a good time, it tells the cloud kit container to run an activity
4. conatiner schedules work with cloudd.
5. We can observe logs from each process with console.

(or log stream)
```bash
# Application
log stream --predicate 'process = "CoreDataCloudKitDemo" AND 
                        (sender = "CoreData" OR sender = "CloudKit")'

# CloudKit 
log stream --predicate 'process = "cloudd" AND
                        message contains[cd] "iCloud.com.example.CloudKitCoreDataDemo"'

# Push
log stream --predicate 'process = "apsd" AND message contains[cd] "CoreDataCloudKitDemo"'

# Scheduling
log stream --predicate 'process = "dasd" AND 
                        message contains[cd] "com.apple.coredata.cloudkit.activity" AND
                        message contains[cd] "CEF8F02F-81DC-48E6-B293-6FCD357EF80F"'
```

dasd logs have a specific format.
identifier `com.apple.coredata.cloudkit.activity.export.{STORE ID}`
how the service decides if a policy can run.  policies that affected the decision are listed along with the final decision.
`cloudd` logs information from cloudkit.  Filter by contianer identifier.

One additional process to observe.  `apsd` will receive push notifications and forward to the application.  Causes `NSPersistentCloudKitContainer` to intiate a series of activities.  It asks dasd for time to perform, and works with `cloudd` to fetch updated objects and import to the local store.
apns payload
1.  `cid` container identifier
2. `sid` `zid` subscription id, zone id

challenge is knowing what tools to use to find/analyze
# Feedback
collect diagnostics
* actionable
* specific
any device

3 steps:
1.  Install CloudKit profile
2. Collect a sysdiagnose
3. Collect store files from Xcode

sysdiagnose.  For an iPhone we hold the volume buttons and the side button for a couple of seconds.  After a short while, a sysdiagnose will be available in settings.

For example, I like to airdrop to my mac for analysis.

Download container via xcode.
Both system logs and store files are now available for analysis.

reading logs from sysdiagnose:
```bash
log show --info --debug
    --predicate 'process = "apsd" AND
                 message contains[cd] "iCloud.com.example.CloudKitCoreDataDemo"'
    system_logs.logarchive

log show --info --debug
    --start "2022-06-04 09:40:00"
    --end "2022-06-04 09:42:00"
    --predicate 'process = "apsd" AND 
                 message contains[cd] "iCloud.com.example.CloudKitCoreDataDemo"'
    system_logs.logarchive
```

```bash
log show --info --debug
    --start "2022-06-04 09:40:00" --end "2022-06-04 09:42:00"
    --predicate '(process = "CoreDataCloudKitDemo" AND
                      (sender = "CoreData" or sender = "CloudKit")) OR
                 (process = "cloudd" AND
                      message contains[cd] "iCloud.com.example.CloudKitCoreDataDemo") OR
                 (process = "apsd" AND message contains[cd] "CoreDataCloudKitDemo") OR 
                 (process = "dasd" AND
                     message contains[cd] "com.apple.coredata.cloudkit.activity" AND
                     message contains[cd] "CEF8F02F-81DC-48E6-B293-6FCD357EF80F")'
    system_logs.logarchive
```

# Next steps
* explore with data generators
* analyze with instruments and logs
* provide actionable feedback


* https://developer.apple.com/forums/tags/wwdc2022-10119
* https://developer.apple.com/forums/create/question?&tag1=71&tag2=54&tag3=417030
* https://developer.apple.com/documentation/coredata/synchronizing_a_local_store_to_the_cloud
* https://developer.apple.com/documentation/coredata/mirroring_a_core_data_store_with_cloudkit
* https://developer.apple.com/documentation/coredata

