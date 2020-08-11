# New Platforms
https://developer.apple.com/siri
Many have custom apple tv experience, and we want to open up sirikit to that as well.

## Demo
[[Introducing Siri MediaKit Intents - 19]]
```swift
func resolveMediaItems(for intent: INPlayMediaIntent, with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void) {
}
```
```swift
func handle(intent: INPlayMediaIntent, completion: (INPlayMediaIntentResponse) -> Void) {
  completion(INPlayMediaIntentResponse(code: .continueInApp, userActivity: nil))
}
```

Customers generally only interacting with 1 at a time.  So the foreground app is more likely the preferred interaction, use `.continueInApp`.


# New Features
ControlAudio sample app from last year's talk

"Maybe you wanted" provides alternatives.

## How do we populate the list?

You're probably calling

```swift
INPlayMediaMediaItemResolutionResult.success(with: mediaItems[0])
```

Instead, use plural version
```swift
INPlayMediaMediaItemResolutionResult.successes(with: mediaItems)
```

## How do we handle alternative choices?
It's passed to `handle`

```swift
func handle(intent: INPlayMediaIntent, completion: (INPlayMediaIntentResponse) -> Void) {
  completion(INPlayMediaIntentResponse(code: .handleInApp, userActivity: nil))
}
```

## Demo

```swift
func resolveMediaItems(for intent: INPlayMediaIntent, with completion: @escaping ([INPlayMediaMediaItemResolutionResult]) -> Void) {
    let mediaSearch = intent.mediaSearch
    resolveMediaItems(for: mediaSearch) { optionalMediaItems in
        guard let mediaItems = optionalMediaItems else {
            return
        }
        completion(INPlayMediaMediaItemResolutionResult.successes(with: mediaItems))
    }
}
```


# Performance improvements
* In-app intent handling
	* Moves intent handling into the app
	* Avoids process launch
	* Potentially slower initial Siri response
	* Potentially easier to prepare your audio player

[[Empower Your Intents]]

## App prewarming
1.  resolve
2.  confirm
3.  handle
4.  background app launch

Now we can do the background app launch prior to resolve, so that playback is ready to go.  Does require some additional work in your app.
