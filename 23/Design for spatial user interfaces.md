consistency and familiarity with the existing platforms
# UI Foundations
* appi cons
* materials
* typography
* vibrancy
* colors


## app icons
one step further -making app icons 3D and in real space. When peopel look at them, they expand.  
specularl  highlight and shadow.  How can you design a great icon?
**use multiple layers**.  Use layers to create a parallax effect.  Just by using flat layers, system can create a truly 3D effect.

* background
* fg1
* fg2
(up to 3 layers)
each layer is a square image of 1024x1024.  Both foreground layers have a transparent bg.

all layers are cropped with a circular mask.  Shadows and depth/specular are provided automatically.  Keep things to the center.  
avoid using large regions of semitransparent pixels.  there's a shadow behind it.

## materials

**apps should be easy to place**, use, view.

system-defined beuatiufl glass window, definesp hysical wall. Allows light from surroundings and mutual content to show through.  specular highlights and shadows aruond corners.  

lighter and adding a sense of physicality.  Materials also give peopel a sense of what might be behind a dinwo, such as app or people.  To deliver spatial experiences, be aware of your surroundings.  Avoid opaque windows.  too many opaque windows feel constricting, amake interface feel heavy.

in this transition from day to night.  unlike iOS?macOS, this platform does nto have light or dark appearance.  UI naturally adopts depending on lighting conditions.

system-provided materials are nice.

to separate sections, use system materials.  you may even consider using darker materials in input fields, etc.  Here is how music app looks.  As you can see..

do not stack lighter materials on top of each other, as this effects legibility.

## typography
use semantic names that work on all platforms.
legible, etc.
to learn more, check out the session [[Principles of spatial design]]

we also modified font weights to improve legibility.

on this platform, we use medium rather than regular.  And we use semiobold instead of bold for titles.  This just improves legibility.

brand-new font styles.

extra-large title 1.
even though windows can scale up, smaller/lightweight fonts.  

**use bolder font weights**.
## vibrancy
brightens content that displays on top of material, lighten color, etc.
on this platform, since the bg can be constantly changing, vibrancy updates in real-time to ensure your text is always legible.

vibrancy improves leigbility.  Let's see how you can take advantage.

indicate hierarchy for text, symbols, etc.  3 modes
* primary
* secondary
* tertiary

## colors
most of the time, consider using white text or symbols to ensure these are visible.  If you need to use color, use it in a bg layer, so people can see it.  When possible, use system color instead of custom color, as these are optimized for legibility, etc.

how to create layout that is easy to target as you take your app from screen to spatial.
# Layout
## ergonomics
prioritize peoples' physical comfort and safety.  Consider the ergonomics of your design, ensuring that the placement of content is intentional and doesn't cause eye/neck fatigue

easy for most people to to move horizontally rather than vertically.

be careful about placing things too high up or far down.  Go with a wider aspect ratio rather than taller.  ex, 

center important information

## sizing
slight variations.  Element should use sizes that are easy to target.

tap target area of 60pt.  

ensure UI has 8 pts of empty space around it, so it meets the minimum ahving a target area of 60 pts.

if you need to have several buttons in a stack, use standard system buttons with 16pt of space between them.

say you need to use a visually smaller elements.  mini button - 28 points, even though this might look small, use 60pt around it, so it's easy to select.

using large/XL buttons requires less spacing around them. Give interactive elements 60 points of space.

## focus feedback

powerful tool built into every interactive element.  When people look at system-provided components, subtle hover effect.  This effect allows peopel to understand which parts of the interface are interactive by looking at them.  Whena n item becomes inactive, it no longer gets focus feedback.  Confidence that tehy're focused on the intended element.

small amount of padding between list items (4pt) to avoid having hover effect overlap.  Include a shape that allows the system to display the hover effect.  helps someone understand the entire element can be selected.  Keep small space between containing shape.

ensure nested elements have relative corner radii and padding.
keep corners concentric.  Use a simple formula.

`inner corner radius + padding = outer corner radius`

remember, to keep your conrner smooth, set them to be continuous corners.

throughout the system, every element is concentric.

keep nested elements concentric.


# From screen to spatial

* window
* tab bar
* side bar
* ornaments
* menus
* popovers
* sheets

inputs.

connect keyboard/trackpad.  Interact with wide array of inputs.
interacting with the system feels magical. Elements ahve to provide the apporpriate feedback for each models.  system components built to provide each input.

**use system components**.  spend time finding what makes your app unique.

check out [[Design for spatial input]]

many components will look familiar.

## window, tab bar, side bar
you need a window, which has an opaque material, and provides canvas for elements to sit on top.  The window is made of a glass material, that allows peopel to move your app fluidly in the space.  

on top of it, we have content.  On iOS, window horizontally, with tab bar.  
Here, tab bar is vertical, floating on a fixed position on left side of window.  Out of the way, there where you need it, etc.  

in general, keep this feelign light.  Avoid more than 6 items.
when peopel look at the tab bar, they can quickly select an item.  If they ylook longer, it expands, showing labels for each section.

when people look away, it automatically closes again, allowing peopel to focus on the content.

if you need to provide navigation, sidebars are nice.  Gives people a clear sense of where they are in that tab.

## ornaments
in the photos app, we have a top bar.  On this platform, we display this at the bottom, slightly in front of the window.  Additional persistent controls that are easy to access using depth to create hierarchy.

toolbars.  Allow peopel to perform quick actions in a convenient location.  Add depth to the app.  Since ornaments are a collection of buttons, this is a perfect place to use borderless buttons.  In this case, it's clear that these are interactive elemetns, and you'll still get hover effect.

now playing controls.

when ornaments sit at the bottom edge, place them so they overlap the bottom edge by 20 points.  Integrated with the main window, without blocking too much content.

nice to see content go through glass, etc.

can appear/disappear entirely.  Only recommended while focusing on a single piece of content.  ex, looking at ap hoto, watching a movie, etc.

quick access to important controls with a tap.  Ornaments can expand, etc.

**take advantage of ornaments**.

## menus, popovers, sheets
menus align to leading edge.  
popovers point to elements, nav bar becomes inactive, etc.
menus/popovers can expand outside the window.  centered by default, ensuring that content appears where the user is looking.

Keep in mind that to change the button to selected state... helps people have clear feedback, etc.

**avoid white-background buttons unless selected**

sheets represented as modal views.  Center of the app.  Modals have the same z-position as the window, the window pushes back.  Focuses the experience and prevents UX interaction.

to present another sheet, more modals!

because we're stalckign so many elements, sconsider using push navigation for nested views

push navigation... secondary view will present a back button instead of close.  Also, notice how close and back buttonsa re in the top left corner.

**always place close buttons on the top left**.

in photos, we keep the browsing experience familiar, spatial capture, etc.  Relive you rmoments and experience content in a unique, special way.  

