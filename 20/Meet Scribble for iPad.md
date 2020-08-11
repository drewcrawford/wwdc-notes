#pencil

# Overview
Write directly into any text field, not into a separate writing area.

Pencil system experience.
Any text input
Fast, accurate, private recognition
English, chinese, cantonese
# Writing experience
Fluid and effortless.  Non-modal, direct, ephemeral.  You shouldn't have to tap on the field.

Effortlessly jump from field to field.  

Consisntency.  Works everywhere, editing gestures, familiar interaction.  

Some people find it natural to write in the blank area at the bottom of the list, so reminders adopted the API to allow writing in this area.

Gestures: Draw a horizontal line to select text.  Scratch out some text that you want to delete.

Pencil friendliness.  Reduce distractions, stable layout, space to write.

When you start writing into a textfield, the placeholder disappears.  

Writing on a search field that moves is annoying.  Wait for you to pause.  Search controller does this for free.  You can request delaying focusing of the field.

Messages detects that you're using scribble and it adjusts the size of the field.
# API
Standard Text Controls
UIKit Text Input
`UIScribbleInteraction`
`UIIndirectScribbleInteraction`

## Standard text controls

It just works!  
Customize look and feel
Password fields are not supported by scribble, because we recommend using autofill.

## UIKit Text Input
Custom text editing.

`UITextInput`, `UIKeyInput`, `UITextInputTraits`.  
`UITextInteraction` - cursor and selection UI

[[Keyboard input in iOS - 12]]
[[The keys to a better input experience - 17]]

## customization
### `UIScribbleInteraction`

Add interaction to views.  Interaction has a delegate.
* Disable scribble
* Delay first responder
* Scribble begin/end

* Reduce distractions -> can hide completion text
* Space to write -> `.isPencilInputExpected` use to show more space if true
* Drawing vs writing -> disable scribble to allow drawing.  `shouldBeginAt:` delegate method

### `UIIndirectScribbleInteraction`

* Enable writing in non-text-inputs
* Single or multi-field


Provide identifier, frame ,etc. via delegate implementation.
