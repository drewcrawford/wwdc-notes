#design 

But people may not know a feature exists 

How people learn about what your app can do.  If your interface is discoverable, then people should be able to look at a screen and discover what they can/can't do.

How to make your app discoverable?

How will people learn about features?  

You need an interface that stands on its own. **Learning by doing**.

Here are 5 fundamentals.

# Prioritize important features
You don't get a second chance at a first impression. 

Not all pixels are created equal.  

Essential: Immediately visible to people

Non-essential: Ok to require navigation to reach

No way they're all going to fit into one screen.  How to prioritize important features so we're not cramming them all into one screen?

We ordered them by their level of importance.  

A person using your app might want to know the developer to report a problem / make a suggestion, but it's not a top thing.

Try to understand what is important from your user's POV?  

Hamburger menu.  When we tested our interface, we found out that... when it's closed people don't know what's inside.  

Instead, we decided to go with a tab bar nav system at the bottom.  Quickly switch between sections.

Minimal != usable and simple.  People won't know what to do.  When we hide things to make the app look minimal, we increase the risk that people won't find features.

But what if all the features are equally important and I need tons of tabs?  

Have to make hard decisions about what to prioritize.  We picked the top 2 features and put them in the tab bar.

We put the features inside the tabs?
It's ok if ait takes an additional tap to get there.

Make essential features immediately visible.  

# Provide visual cues
Guide people toward a particular piec eof content or action.

* Know your audience
* Who uses this?  Professional chefs?  

People won't want howto instructions that are indepth.  We want ot provide as many visual cues that clue them into how to use the app, while they're using it.  What do people know already?  We assume people can infer.

Icon is tied to the physical world.  

People using iOS associate teh action of tapping a chevron in top left to do back.  We don't need to add extra cues.

Guide with words and visuals for unique design.  Help people understand what to do.

Recognizable icons can be - add labels.  Clarify meaning of each icon.

Camera icon that we're familiar with, can have many possible uses, such as taking a picture, scanning a document, etc.  Adding text labeling – we clarify the button.

Search bar – people are familiar enough with search icon, bu draw a blank about what to write.  We aveh't clued them in about how to search.  We call this the *blank page problem*.  When you create things, presengint people with a blank white screen can lead to indecision.

Provide samples of possible input in placeholder text.

Use animation to teach in context.  Consider what knowledge people are equipped with and when they need helping hand

# Hint at gestures
Gestures are great at short tasks.  Actions, navigation.  But they're invisible and not as immediately discoverable.

* Use familiar gestures.
	* For inventing non-platform gestures, consider real-world actions
	* In non-games, best to use standard gestures
	* tap, swipe, long press, pan, pinch, rotate
	* People expect standard gestures to work across the system
* New experiences may not have gestures, consider real-world behavior.  e.g. scribble scratches-out text.

Even gestures that people are familiar with, can be confusing.  double-tap to like?  To zoom in?

In toast, we have icon and a shortcut.

Use gestures as a shortcut, not a replacement.  Gestures... are invisible.  If there's no way to look at an interface and know how to use a gesture, then use the gesture as an accelerator.

Use animation to hint at gestures (discoverability).  In recipes, tap and hold to save recipes.  Use swipe down as a shortcut to go back.

By looking at a screen, people don't know they can swipe left/right.  So add small images on either side to give a visual queue.

Why do people know things?  Real world?  iOS?  used your app?
# Organize by behavior
The best apps have features you know how to use, navigate.

Organize the content of your app categories by behavior.  What are categories?

* Choose categories that fit people's motivations.  
	* ex, there are too many ingredients to categorize that way.
	* e.g. restaurants, vs "shared by friends"

Still missing one strong category: individual personal taste and flavor.  

Use personalization.  resurface suggestions from recommendation engines.  It makes sense to personalize content.  We can't have a category for each ingredient, but we can have a recommendation engine that lets people target their favorite toast recipes.

"Tasty picks" => our recommendation engine.
Visualize organization.  Making the categories clear.

Already see this is overwhelming.  Too many items that are equally important, difficult to see what you want.

People can easily ignore one category or specific section.  Categories that match people's behavior in real-life context.  

# Convey a sense of control
Let people provide explicit feedback about the item on the screen.  intentional input.  See more of or hide.

People are more likely to give feedback, this feeds thre commendation engine useful data and improves discoverability of the concepts.

Most people thought the heart icon meant "add to favorites".  Icon alone is open to interpretation.  On iOS, you can only show heart/strike when something was loved.

Instead, we tried thumbs up / thumbs down.  But they're still confused.  Does it mean upvote/downvote?

Using the label helps improve comprehensibility.  Now you know what will happen.

Future recommendations will be adapted to show less.

Disclose implicit feedback.  Information that arises as people interact with your app's features.  Happens without people asking for it.  e.g., recipe or sharing with friends.

Where do recommendations come from?  Label can clarify things onscreen.  Helps people understand where the recommendations come from.

How do people edit recommendations?  Give control over recommendations.  Thumbs down vs don't show.  

Or don't show for a specific day?  Don't show one item?

It's fair to debate what feedback to provide and how to label those actions.  

*Make, show, and learn.*

People should feel and have control over personalized content.  

* Prioritize important features
* Provide visual cues
* Hint at gestures
* Convey a sense of control

Make sure people are in control, allow them to make corrections and personalize content.

# Wrap up
* People should know what to do (immediately)
* Understand how people know things
* Make, show, and learn

[[Designing great ML experiences - 19]]
[[The life of a button - 18]]
