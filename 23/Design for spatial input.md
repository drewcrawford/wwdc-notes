available input modalities.
You can simply look at a button and tap fingers to select, keeping your arm relaxed.

our system is designed to interact with UI comfortably at a distance.  Can also itneract with elements directly.  Typing on a virtual keyboard with fingertips.

holding your hands can cause... we will see that some tasks are better suited to interact directly.

can also use keyboard/trackpad, etc.

connect game controllers.  We're going to focus on new / exciting spatial input: eyes and hands.

**spatial input is personal**.
**spatial input is comfortable**.
**spatial input is precise**.

* eyes
* hands

# eyes
primary targeting mecahnism for spatial experiences.  everything in the system reacts to where you look, no matter how far they are.

most comfortable to look in the center, less comfortable to look at the edges.  Design apps taht fit inside the FOV, evidently 60 degrees?

keep the main content in the center of FOV.
use ancilleary content outside

**place content inside the FOV**.

consider depth when thinking about eye content.  depth is a great feature.  Placing content near/far creates different feelings.  but our eyes focus on 1 distance at a time, and changing the focus depth can create eye strain.

**keep interactive content at the same depth**.
ex, presenting a modal view pushes the main view in the Z zxis, and modal is placed on top.

By maintaining the same position, your eyes need to adapt to the same distance.
Use subtlel depth changes to communicate hierarchy.

using depth meaningfully while avoiding eye strain.
## comfortable
consider
## easy to use
eyes are very precise, but there are certain qualities that help eyes target successfully.  eyes naturally focus on shapes.

use round shapes - circles, pills, rounded rects
avoid usign shapes with sharp edges.  Sharpe dges, eyes tend to focus on the outside, decreasing precision.

keep shapes flat.  avoid thick outlines or effects thatcall attention to edges.
Make sure to center text using generous padding.

guide the eyes to the center of the content.

let's look at right size for control.  We recommend 60pt.  Can combine size and spacing (44 + 8).  Use generous spacing, to help yuo target quickly and accurately with your eyes.

make target areas of at least 60 points.  Combining size and spacing.  

see [[design for spatial user interfaces]]

scale mechanisms.  system provides dynamic scale parameters.  You can see how the window scales larger as it scales away, and smaller as it moves close.  Dynamic scale makes your UI fill the same FOV and preserve the size of the target areas, no matter the window position.

fixed scale can make your UI smaller as it moves away.  Difficult to fix with your eyes.  

**use dynamic scale for UI**.

orientation also affects the use of your UI.  

system windows are always oriented to face default.  If you create custom windows, keep UI oriented to field of viewer.

see [[Principles of spatial design]]

## responsive
when elements highlight, you understand that your eyes are driving the interaction.  Let's see what happens when you look at buttons - they highlight.  all interactive elements should be highlighted, we do this with hover.  The effect needs to be subtle.

system-provided controls highlight when you look at them.  

**use hover effects to provide feedback**

privacy is our top priority when dealing with eye data.  hover effect happens out of processed.  You will only get information from gestures.
## intentional

when you look at something for a long time, we know that you're interested.  opportunity to show your more information.

buttons can have two groups that reveal as you look at them.  Tab bars expand when you focus on them.

focusing on micrphone will trigger 'speak to search', revealing this layer and allowing you to perform a search using eyes and voice.

**take advantage of eye intent**

also provides great opportunities to assistive tech.  Can select content just with your eyes.

* place content in front of the viewer
* design elements that attract your eyes
* respect the minimum target area for eyes (60pt)
* use hover effects to provide feedback
* reveal extra information on eye intent
# hands

gestures.
pinch -> press
system supports pinch/drag to scroll
two-handed gestures like zoom and rotate.

connect UI feedback to gesture

**use familiar patterns**

successful custom gesture.  Ensure your gesture is easy to explain and perform
avoid gesture conflicts
comfortable and reliable
accessible to everyone
[[Create accessible spatial experiences]]
unambiguous

consider fallbacks in UI affordance

## eyes and intent
start of zoom is determined by where you are looking.  That particular area becomes magnified in the center.  Navigate the image easily just by looking around.

can draw with touch,b ut look antoher place to move cursor.  These are examples of interactions that use both to make simple behaviors more precise and satisfying

eyes implicitly provide a more granular location for the interaction.

## direct touch
ex scroll safari directly, tap on keyboard, etc.
interactions at a distance stay comfortable for a long time.  Hands can stay rested, etc.

holding hands in the airw ill cause fatigue.

when you use direct
* inspection and manipulation
* familiar mechanics
* physical activity

consider lack of tactile response.  **provide extensive feedback for direct touch**.

for keyboard, we highlight as you seem to touch.  sound effect, etc.  These layers of feedback are really important.

audio plays a special role in connecting input with content.  

[[Explore immersive sound design]]

* use familiar gesture language
* only introduce custom gestures when necessary
* use eye direction in your interaction
* use direct interaction when it fits your experience
* show clear feedback for interaction states

