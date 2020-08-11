
# A high quality siri experience
Trust level needs to be higher for voice

Play something.  If the user asks for play, and something doesn't play, they won't ask again.

Play something *fast*.  One of the biggest failure cases in SiriKit media apps are timeouts.  We can be more aggressive in certain environments

[[Expand your SiriKit Media Intents to More Platforms]]

Play something fast and *perfect*.  Adopt Siri user vocabulary API.  You can help Siri to understand your app's catalog to do "global vocabulary" API

When you're choosing the perfect thing to play, allow people to ask in different ways.  Support variation.

# Common Utterances
"perfectish" a best guess when you don't know what someone wants.  50% or more follow this kind of pattern.

"play music on app"
"play on app"

This is often the *most important usecase*.  

| Sample utterance(s)                       | Use case                                      | Usage amount |
|-------------------------------------------|-----------------------------------------------|--------------|
| "Play \<myapp>" / "Play music in \<myapp>"  | Empty `INMediaSearch` or `mediaType=music`    | >50%         |
| "Play \<something> in \<myApp>"             | `mediaName=<something>`                       | 30%          |
| "Play \<something> by \<artist> in \<myApp>" | `mediaName=<something>`,`artistName=<artist>` | 5%           |
| "Play the playlist \<playlist> in \<myApp>" | `mediaName=<playlist>`,`mediaType=<playlist>` | 5%           |
	
"Play \<thing>" -> it could be an album, song title, playlist.  Use a broad search.  We don't have a `mediaType`.

The better your siri experience, the more likely it is for users to do more complicated patterns.

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

# Demo
Using text in run options to drive siri

# Vocabulary

## why vocabulary?
Machine-learning system.  Tries to predict someone's intent.
Built-in training for audio features
* genres, media types, media sorts, date
* Generally speaking this is desireable

"Play 70s punk classics" -> may try to parse "punk classics" as a genre and "70s" as a time period.
But if your playlist is named that, this is a problem.

## User vocabulary
* User vocabulary is used for personalized items
* Shared with Siri using `INVocabulary` API

```swift
let vocabulary = INVocabulary.shared()
let playlistNames = NSOrderedSet(objects: "70s punk classics")
vocabulary.setVocabularyStrings(playlistNames, of: .mediaPlaylistTitle)
```

* prioritize user-created content, but can also be used to bias speech and NL recognition towards a taste profile.

## global vocabulary
Used for global catalog items
* shared using AppIntentVocabulary.plist

Only include things that aren't part of Siri's built-in catalog.  Not music or podcast entities.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ParameterVocabularies</key>
	<array>
		<dict>
			<key>ParameterNames</key>
			<array>
				<string>INPlayMediaIntent.playlistTitle</string>
			</array>
			<key>ParameterVocabulary</key>
			<array>
				<dict>
					<key>VocabularyItemSynonyms</key>
					<array>
						<dict>
							<key>VocabularyItemPhrase</key>
							<string>70s punk anthems</string>
						</dict>
					</array>          
					<key>VocabularyItemIdentifier</key>
					<string>70s punk anthems</string>
				</dict>
			</array>
		</dict>
	</array>
</dict>
</plist>
```

Note that the key for user vocabulary, and the plist key for global vocabulary, are differently-named.  He displays a table of their correspondance.  They ultimately land in Siri in the same place.

Vocabulary is an ordered set.  Siri prioritizes items at the beginning of the list.  There are a limit to the numbr of items that Siri will recognize.

In the real world, it may takes some amount of time for vocabulary changes to take effect, power/network issues.

# Now playing support
`MPRemoteCommandCenter`
Register a command, iOS does the rest.
If there's a button in control center, etc., then you want a command

[[Becoming a Now Playable App]]

Siri acts as a voice layer on top of iOS Now Playing support.  Siri sends a command to `MPRemoteCommandCenter`.  

Note that your siri handling and now playing button presses have identical implementations.

`.pauseCommand`
`.playCommand`
`.previousTrackCommand`
`.nextTrackCommand`
`.skipForwardCommand`.  Can tell Siri how far to skip forward and will package as interval
`.skipBackwardCommand`
`.chageRepeatMode` `.changeShuffleMode`
`.changePlaybackRateCommand`.

If you don't implement , Siri gives an error.
Can also temporarily disable a command, in contexts where you might not support in certain circumstances.
Finally, you can fail the command.  Typically you do this in exceptional cases.

Discoverability.  Siri engagement is 10x when there's some education about the feature of the app.  Match your app's look and feel.

