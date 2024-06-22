Learn how to use App Intents to expose your app's functionality to Siri. Understand which intents are already available for your use, and how to create custom intents to integrate actions from your app into the system. We'll also cover what metadata to provide, making your entities searchable via Spotlight, annotating onscreen references, and much more.

# Introduction

Integrate with siri:
* expose features outside your app
* Quicker, more efficient actions



# What's new
* A more natural voice
* Personal context and on-screen awareness
* Richer language understanding

Take new actions in and across apps

Built on top of app intents framework

mail/photos: available today
rolling out more over the next fews months.
over 100 different actions



# Actions
* Building

schema.  
shapes optimized for models
You write `perform`

Our models are trained to reason over schemas, predict based on user request, etc.

Toolbox -> app intents on your device, grouped by schema.  By conforming to schema, you give the model an opportunity to reason over it.

`@AssistantEntity`.

## xcode demo

Need to ensure associated enums are also exposed.

Compiler is a great tool to help you conform existing app intents to schemas.

Assistant schemas enable additional build-time validation.  
Xcode validates intents and entities
Use xcode snippets to get started

## testing

These automatically appear as actions in shortcuts.
Personal automations, home screen shortcuts, and more.  
Available today!


# Personal context

In-app search.

Built on ShowInAppSearchResultsIntent
Navigates to your search results

To conform to new schema, add a Swift Macro before your app intent.

Semantic search.  This year, we're expanding Siri's capabilities so it can do even more.

Find and understand photos, files, and even your app's content.

For your apps, can provide IndexedEntity to give Siri the ability to search your app's ocntent, making information available in the semantic indexed.

[[What’s new in App Intents]]
# Wrap up

* Make your app capable, flexible, and intelligent with Siri
* Adopt SiriKit or App Intents
* Use Assistant Schemas to integrate your app with Apple Intelligence

[[What’s new in App Intents]]
[[Bring your app’s core features to users with App Intents]]



# Resources
* https://developer.apple.com/documentation/AppIntents/app-intent-domains
* https://developer.apple.com/forums/topics/machine-learning-and-ai?cid=vf-a-0010
* https://developer.apple.com/documentation/AppIntents/Integrating-your-app-with-siri-and-apple-intelligence
* https://developer.apple.com/documentation/AppIntents/Making-actions-and-content-discoverable-and-widely-available
* 