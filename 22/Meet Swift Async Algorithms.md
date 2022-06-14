#swift #async 

alongside collections and algorithms.

# AsyncSequence recap
* just like sequence
* iteration using swift ocncurrenyc
* iteration can throw
[[Meet AsyncSequence]]

You alreayd have map, filter, reduce, and more.
github.com/apple/swift-async-algorithms has more advanced algorithms.

[[Meet the Swift Algorithms and Collections packages]]

```swift
struct Account {
  var messages: AsyncStream<Message>
}  

actor AccountManager {
  var primaryAccount: Account
  var secondaryAccount: Account? 
}

protocol MessagePreview {
  func displayPreviews(_ manager: AccountManager) async }
```
# Multi-input algorithms
zip
* combines values produced into tuples
* iterates concurrently
* rethrows failures

```swift
// upload attachments of videos and previews such that every video has a preview that are created concurrently so that neither blocks each other. 

for try await (vid, preview) in zip(videos, previews) {
  try await upload(vid, preview)
}
```

zip itself has no preference on which value comes first or not.
sends a complete tuple.

Merge.
* combines multipel asyncsequences into one asyncsequence
* element types must be the same
* awaits elements concurrently and rethrows failures

```swift
// Display previews of messages from either the primary or secondary account

for try await message in merge(primaryAccount.messages, secondaryAccount.messages) {
  displayPreview(message)
}
```

# Clock, instant, duration

## Clock
* protocol for defining time
* defines aw ay to wake up after a given instant
* defines a concept of now

Migrate from existing callback events to lock sleep function.  create a deadline by adding a duration value to delay.
```swift
// Sleep until a given deadline

let clock = SuspendingClock()
var deadline = clock.now + .seconds(3)
try await clock.sleep(until: deadline)
```
measure elapsed duration of work.
```swift
let clock = SuspendingClock()
let elapsed = await clock.measure {
  await someLongRunningWork()
}
//Elapsed time reads 00:05.40

let clock = ContinuousClock()
let elapsed = await clock.measure {
  await someLongRunningWork()
}
//Elapsed time reads 00:19.54
```
Key difference is behavior asleep.
For long-running work, the work can be paused just as we did here.  But, when we resume the execution,t he continuous clock has progressed while the machine was asleep while the suspending clock did not.
Use the suspending clock for
* measuring device time
* delays for animation
* any "machine" type of time

Use the continuous clock for
* measuring human time
* delays by an absolute duration

## Algorithms using time
Rate limit interactions, efficiently buffer messages.
```swift
// Control searching messages

class SearchController {
  let searchResults = AsyncChannel<SearchResult>()

  func search<SearchValues: AsyncSequence>(_ searchValues: SearchValues) 
    where SearchValues.Element == String 
}
```
wanted ot make sure we rate-limit search emssages on the server.

### Debounce
awaits a quiescence period
rethrows failures immediately

```swift
let queries = searchValues
   .debounce(for: .milliseconds(300))

for await query in queries {
  let results = try await performSearch(query)
  await channel.send(results) }
```

by default, debounce uses the continuous clock.

## chunks
Groups elements into collections
* by count
* by time
* by content


```swift
let batches = outboundMessages.chunked(
  by: .repeating(every: .milliseconds(500))
)

let encoder = JSONEncoder() 
for await batch in batches {
  let data = try encoder.encode(batch)
  try await postToServer(data) 
}
```

# collections
Just like lazy algorithms, oftentimes we need to move back into the world of collections.
* Initialize from AsyncSequence

```swift
// Create a message with awaiting attachments to be encoded
init<Attachments: AsyncSequence>(_ attachments: Attachments) async rethrows {
  self.attachments = try await Array(attachments)
}
```

* combine
* ratelimit
* chunks
* just the highlights that we ended up using in our app.  This pakcage has many more!
* buffering
* reducing
* joining
* injecting values intermittently
* and more!

try it out
