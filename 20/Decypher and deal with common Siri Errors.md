Debugging #siri and #shortcuts 

You can provide Siri intent query from scheme editor in xcode.  In your attachent to your intents extension, can choose between Siri and the shortcuts app.

Both of these extensions are separate processes.  Attach to multiple processes in Debug mode.

# Sorry there was a problem with your app.
* Your intent handler gets 10 seconds to call the completion handler
* make sure to call completion handlers **only once**.
* Your app/extension might have crashed, check the crash logs

`os_log` and console app mgiht help you understand how multiple processes work together.
Can use console app to filter by your keywords.  Check all processes.