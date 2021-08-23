#shortcuts  #design 
Focus on experiences that people can have with your app when it isn't open.  

# Actions
Way of representing a task that peopel can complete with your app.

Mix and match different actions together to form a shortcut that does something even more powerful.  

If you donate actions to the system, we'll suggest your actions.



# What makes an action great?
1.  Useful
2.  modular
3.  Multimodal
4.  Clear
5.  discoverable

## Useful
A useful action performs a task
* Perform a task that can be done inside your app
* Avoid opening the app, unless necessary
* Ask follow-up questions instead, where possible

what to do if you need more information?  When the shortcut is run, action can ask a followup question.  When possible, handle ambiguity by asking a folowup question rather than opening your app.

A useful action is **not too immersive**.  e.g. creating custom watch face is not a good idea for a shortcut.  Compare with switching between watch faces.

Lets me change my watch face in one tap.  

A useful action is **repeatable**.  Used on a recurring basis, frequently, quickly.

## modular
Goal: clean up old todo tasks

One task?  Or many?  Many.

Modular actions can be useful in other situations.  

By combining your actions, other actions, etc., people can create entirely new shortcuts and usecases with your app.

By separating your output into pieces, they show up as a varaible name in the shortcut.  

Consider designing (around your custom objects)
* Create
* Edit
* Delete
* Get
* Thing
* Open

* Composeable
* sueful standalone
* parameters

## multimodal
When you're designing actions, keep in mind that they can be used in 3 ways
1.  Run from UI
2.  Run with Siri
3.  Edit in shortcuts editor

Note how the same prompt is used in the compact UI, and on the homepod.  The prompt is phrased as a question and appropriate in both contexts.

`Deadline:` is fine on the iPhoen but doesn't make sense when spoken by Siri.  **Phrase your prompts as questions**

[[Designing Great Shortcuts - 19]]
[[Evaluate and Optimize Voice Interaction For Your App]]

In shortcut editor, same parameter are displayed as blue rectangles that can be edited.  Floating tip shows the date.  Always aim to make all parameters useable in both.

Pick a type for action parameters.  If your action needs a unique entity that isn't well-represented, define a custom type.

To help people understand the devices they can choose from, shortcut editor shows visuals.  Because there isn't a builtin paramete,r the action defines a custom parameter and a UI to go with it.

1.  Configuration => setting up your action in the shortcut editor
2.  Resolution => When action is run

For example, when we run "order coffee" a preview is shown.  for actions that are permanent, strongly consider showing a different snippet before completed.

## Clear
In iOS 13, we had parameter summaries.  This year, we redesigned editor to focus on the summary.  Notably, app names have been removed.

Parameter summary is different from the action title.  Title is in the action list.  Begin with the same verb and share as many words as possible.

Parameter summaries always begin with a verb.  Written like a sentence, without the period.

Don't include optional parameters or punctuation.  People can still use this from options UI.

If required parameter is left empty, action should ask for a value.  Consider providing a default value.  

People can set any parameter as "Ask Each Time".  Shortcut will prompt for the parameter value.  Always include a prompt or visual list for all parameters.

## Discoverable
New sharing methods this year.

You can place link to multistep shortcuts on your website, app, etc.  Also new, when someone taps "Add to Siri", the shortcut is added instantly using your suggested phrase.  
# Wrap up
* https://developer.apple.com/documentation/sirikit/offering_actions_in_the_shortcuts_app
* https://developer.apple.com/videos/play/wwdc2020/10084/
* https://developer.apple.com/design/human-interface-guidelines/siri/overview/shortcuts-and-suggestions/

