#swiftui 

In this talk, we'll learn about a new swiftuI api for handling bg tasks in a consistent way across platforms.

Unified API for watchOS, iOS, tvOS, Mac catalyst, and widgets.
Async/await with Swift concurrency

Good platform citizen by default

# Stormy: A storm photos app
Take photos of the sky when stormy.  Shows a notification at noon.

When taps the notification, they'll upload a photo of the sky.  Upload in bg.  Notifies when complete.

When our app is first launched into the foreground, take our first opportunity to schedule background app refresh task for noon.

When user leaves app, system will know to wake in bg at scheduled time.

We scheduled our time for noon. With this runtime, we need to figure out whether it's stormy outside.  

1.  urlsession request
2. suspend and wait for rqeuest to complete
3. when bg network request completes, our application will be given runtime again with new URLSession background task
4. Send notification
5. system suspends application
# Background on Background Tasks
Let's take a closer look at lifecycle of single refresh task.

1.  System wakes application and sends it an "app refresh background task"
2. we make network request.  Ideally, our network request completes within the alotted time we have.
3. When we get the response, we'd like to post immediately.
4. Now we're done

But what about, when our network request doesn't complete in time?
If an app is running low, system signals the app that time is running low, giving us a chance to handle gracefully.  If applications do not signal before runtime expires, application may be quit by the system and throttled for future requests.

Make sure that our network request is a background request "in this case".
# SwiftUI API in practice
 code not provided.
BGAppRefreshTaskRequest.

Register a handler cooresponding to bg request we scheudled using `.backgroundTask` scene modifier.  Any blocks that match are run.  In this case, we used `.appRefresh` task type.  Scheduled in advance etc.

Using the same identifier for the request as the handler.  This lets the system identify which handler to call when the corresponding task is received by the application.

We call the schedule function again to make it recurring.

make network request to see if it's stormy outside, etc.

If our network request has returned, we await sending the notification to the user, prompting them to upload a photo to the sky.  Underlying bg task is implicitly marked as complete, and the system can suspend our application again.

Many APIs already support swift concurrency, etc.  Here, the `notifyForPhoto` async function can be implemented in a straightforward way.


# Swift Concurrency
Easier than ever to handle bg tasks.

How to write an async network function, etc.

I think the meat of this session is really the routing of events to the swiftUI application main type (window group or whatever).

We had used the *shared* session. Instead, let's createa  background session.  Tells the system that some requests should run even when the application is suspended.  Wake the app for a background task.  Etc.  Note that this is especially important on watchOS, as all requests made my apps in bg on watchOS must be background sessions.

When our runtime expires, system will cancel the async task.  This means that the network request made here, will also be cancelled when our background runtime is expiring.  To resopnd to that, we can use `withTaskCancellation` handler function.

Instead of waiting the result directly, we place our download into a `withTaskCancellationHandler` call.  The data task will be cancelled.  We replace with a download task in the cancellation handler.

**URLSession will deduplicate any in-process requests under the hood.**

`.backgroundTask(.urlSession("isStormy"))`.  By using the same identifier as the URLSession configuration background identifier, we link these together.

# Wrap up
* Unified background task handling in SwiftUI
* completion and cancellation with Swift Concurrency
[[Meet asyncawait in Swift]]
[[Discover concurrency in SwiftUI]]

* https://developer.apple.com/forums/tags/wwdc2022-10142
* https://developer.apple.com/forums/create/question?&tag1=239&tag2=43&tag3=461030
* https://developer.apple.com/documentation/SwiftUI/BackgroundTask
* https://developer.apple.com/documentation/SwiftUI/Scene/backgroundTask(_:action:)

