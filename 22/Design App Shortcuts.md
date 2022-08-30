Tasks can be easily completed outside their respective apps.  Making them available in siri and spotlight.

Shrotcuts are how you can offer that flexibility with key taps outside teh oS>


Action => A task people can complete with your app.
Shortcuts people can create using one or more actions from apps.  
App shortcuts => Shrotcuts created by app developers containing one action.

In iOS 16, the app shortcuts from your app will be available as soon as your app was installed.  Mkaing it easier than ever to access features from your app throughout the OS.

Create unforgettable app shortcuts.

# Identify a feature
Self-contained
Straightforward.

To give these a bit more context, let's take a look at a sample meditation app.  A great app shotcut is one that doesn't require ytour app in focus at all.  Lightweight task that can be completed via siri/search in its entireity.

Easy to kick this off without opening the app.  Make sure the shortcuts you offer are straightforward.  Effortless to get through.  Not too much input, etc.

Lengthy, multi-step survey is not appropriate for an app shortcut.  Focus on uncomplicated tasks that people can... quickly.  And remember when not usign the app.

The best features are self-contained and straightforward.  Without the app in focus and uncomplicated to get through.  

# Pick a name
This name is really the hero phrase for your shortcut.    See if you can incorporate your app name directly into the phrase.  Can be your official app name, or any alternative name you submitted to the app store.

I'll turn on my transcriptiosn so we can see speech.  The name you choose for app shortcut is what peopel will say to siri.  Provide thoughtful variations for similar phrases.  For this shortcut, I would need to explicitly specify *Start Voicememo* and *New Voice Memo*.

Be thorough.  Capture all alternative phreases people might say.  But don't stray too far from the purpose of your shortcut.  

You will have to go through this exercise for every language where your app is published.  

* Keep it brief and memorable
* Include your app name
* Generate and localize synonyms
* Use a dynamic parameter (optional)

A couple of things to note.  First, you can only have one dynamic parameter in your phrase.  Only used to select from a finite list.  Make sure that other values are predictable.  People are likely familiar with your entities.

Bad example would be something with infinite options like a time value.  But instead prompt for additional info as needed.

This list of parameter values can be updated any time your app is open.  Ensure it contains up to date, relevant values.

Each parameter value creates a unique variant.  Automatically generated and displayed int he shortcuts app.  These parameters are also visible in the shortcuts editor.  Whens omeone taps on a parameter, your options will appear in the menu.

[[Design great actions for Shortcuts, Siri, and Suggestions]]

Start sleep meditation.  If that meditation was required, ask for it in a subsequent step.  Looking back, one theme stands out.  Make it memorable.  Peopel can explore, get drawn itno features, and learn enw flows.  Your mail goal is to create a few focused experiences that peopel can quickly learn, remember , etc.
# Refine your visual
Snippets and live activities provide you with an opportunity to present information, request clarification, etc.  Unlike most of your app tha tuses opaque backgrounds, snippets use a semitranslucent material.  Place leemtnso n this material,d on't use an opaque background.

When you draw your text, use vibrant label colors to guarantee great contrast over the parenet background.

1.  Live activities
2. Custom snippet

Think about whether peopel would benefit from continuous access to th e information, transit progress, timer, etc.  Live activity is continuously glanceable even on the lock screen.  Otehrwise use a custom snippet.

Here, you see the supporting dialog.  This is what siri speaks, and is intended to accopmany, your custom visual.  Together the y communicate all the necessary information.  Your custom visual will consistently appear with supporting dialog.  In this example, you'll see...

Should suppress this dialog in your sourcecode so that it won't be shown.  Presentation goes beyond iOS, and you'll need to learn about other devices in the ecosystem.

airpods => siri reads out full dialog.  Include all info in the full dialog.  Ensure peopel have access to all info regardless of whether they see your visual or not
apple watch => supports custom snippets.  Make sure they look great.

Coffee app repositions some elements on watchOS.

* semi-translucent background
* vibrant label color
* Snippets and live activities
* Device-specific dialog and visuals

In iOS 16, your app shortcuts will also appear in spotlight.  If someone searches for your app name, one shortcut will appear in the siri suggestions below your app as a top hit.  Can slo search directly for shortcut name.

Finally, if your app is a siri suggestion when someone first launches, top shortcut will also appear here, before anything is typed into search.  Provides an opportunity to learn about shortcuts just by looking for your app etc.

Each unique app shortcut has a symbol.  Review SFSymbols and select one for each shortcut.

Know that the order of actions/parameters will influence the order appearing in spotlight.  Order of your parameters is truly dynamic and can be updated any time your app is opened.  Here, you see that reordrered coffee cappuccino is first.  But if they ordered another coffee more recently, you can make it the first option returned by entity query.

As the number of parameters increases, this propritization may become improtant.  Pick a meaningful characteristic so it doesn't feel unpredictable.

**Pick a color for your shortcuts in the shortcuts app.** All your shortcuts will use this color.  Compleements your icon nicely, etc.
# Ask for information
A few different ways to do that.  Sometimes, you can make an informed decision and proceed with the action.  maybe starting a meditation that's in progress?  But you may need to ask for more info.

When possible, try to make a meaningful assumption and present it for confirmation.  These assumptions could be based on remembering a prior selection, etc.  Parameter ocnfirmations are a nice efficient way to ensure you have what you need.

Another option is to provide a short list.  Disambiguation.  Helps people learn values.  Keep in mind that this is best for lists of 5 values or less because in voice-only, siri reads the whole list.  

Maybe you need an open-ended value, like number, location, or string.  You can use an open-ended request.  Less guard rails because peopel can pick anything, so make sure peopel know what type of inormation to expect. 

AppIntent framework provides a set of common options.  Numerical values, dates, time values.  If your parameter aligns with one of these, select it.  Allows you to benefit from certain built-in dialog and visual patterns.  If not, you can use a custom entity.

For more information, see

[[Dive into App Intents]]

Lastly, even if you  have all the information you require, may still want to do one final confirmation.  Only use this for consequential actions.  Destructive action, or an action that may feel high-risk like sending a calendar invite.  Use these appropriately but sparingly.  

One last detail.  Always have a pair of buttons offering to either proceed with or cancel the proposed action.  Buttons should contain a verb re-iterating the action.  

| Default verb | additional verb |
| ------------ | --------------- |
| order        | Book, buy       |
| Start        | navigate        |
| Share        | Post, send      |
| Create       | Add             |
| Download     | Get             |

Can provide a custom string.  Be sure to provide synomyns as well.
# Make it discoverable

First, in place of "add to siri" button is a new tip. Carefully select moments to surface these tips at times people are likely to benefit.  Before or after an action they may want to repeat.

make your tip dismissable.  People may want to remove.

Button to link to shortcuts app.  

# Wrap up

#shortcuts 

* https://developer.apple.com/forums/tags/wwdc2022-10169
* https://developer.apple.com/forums/create/question?&tag1=392030&tag2=507030
* https://developer.apple.com/documentation/AppIntents
* https://developer.apple.com/documentation/sirikit/offering_actions_in_the_shortcuts_app
* https://developer.apple.com/design/human-interface-guidelines/siri/overview/shortcuts-and-suggestions/
