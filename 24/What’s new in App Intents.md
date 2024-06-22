Learn about improvements and refinements to App Intents, and discover how this framework can help you expose your app's functionality to Siri and all-new features. We'll show you how to make your entities more meaningful to the platform with the Transferable API, File Representations, new IntentFile APIs, and Spotlight Indexing, opening up powerful functionality in Siri and the Shortcuts app. Empower your intents to take people deep into your app with URL Representable Entities. Explore new techniques to model your entities and intents with new APIs for error handling and union values

[[Bring your app’s core features to users with App Intents]]


# Spotlight integration
Suggestions.  Suggested actions.
App shortcuts

IndexedEntity protocol.
Index via CSSearchableItem.  AttributeSet to extent information you provide

IndexedEntity is a new api?  

Default implementation only uses display representation to populate the attribute set.  But you can provide more information by implementing attributeSet yourself.


# Entities and files
Making entities meaningful.  Define and expose concepts.  Other apps can't understand these concepts. But we can represent e.g. a PDF, with a UTI.

Transferable AppEntity.  e.g. export pdf, png, rtf.  

order of transferable representations is important.  Use high fidelity first.  

Transferable restrictions
* xcode tools read transferable representations at compile time
* compiler feedback if there are issues.
Properties can only be referred to if there's `@Property` decorator.  

[[Meet Transferable]]

## IntentFile

We discussed converting various types.  Earlier, I howed appending my activitySummary to a note.  When we get intent parameter, we check which content type is available.  Use transferable representation to convert to requested content type.

`attachment.availableContentTypes.contains(.png)`.  

## FileEntity
For apps that manage files.  

In some cases, the file is the canonical version of the entity.  Siri/shortcuts can facilitate secure access to your file in other apps, directly access the file.  e.g. in-place edits.

Create with URL, or if file doesn't exist yet, as a draft identifier.  Use URL's bookmark data, so if the file is moved/renamed, path is still valid.

# Universal links
Help people access your content, whether or not app is installed.  

https://trailsapp.example/trail/1/details

URLRepresentableEntity
URLRepresentableEnum
URLRepresentableIntent

can use entity's id, or any property with the `@Property` for an interpolable string.  



# Developer improvements

## UnionValue

add supports for tagged enums to `@Parameter`.  Each case in the enumeration must have exactly 1 associated value.  No values is not valid.  Each value must be distinct.  

## Generated titles
Building with xcode 16, titles are no longer required.  We generate titles automatically based on the name of the property.  

## Framework improvements

In xcode 16, the limitation is gone.  Allowing you to define app entities in a framework and reference from extension targets.

Only frameworks.  Libraries outside of a framework are not supported.

# Wrap up
* index your app entities in spotlight
* Provide meaningful representations of your app entities
* Try out URLRepresentable

[[Bring your app to Siri]]
[[Build a great Lock Screen camera capture experience]]
[[Extend your app’s controls across the system]]


# Resources

* https://developer.apple.com/documentation/AppIntents/AcceleratingAppInteractionsWithAppIntents
* https://developer.apple.com/documentation/AppIntents
* https://developer.apple.com/documentation/AppIntents/Making-actions-and-content-discoverable-and-widely-available
