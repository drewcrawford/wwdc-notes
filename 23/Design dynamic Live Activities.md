Live Activities allow your app to display live information in key system locations on iOS and iPadOS. Learn the best way to create graphically rich layouts that update seamlessly on the Lock Screen, in StandBy, and in the Dynamic Island. Incorporate interactivity and animation to help people stay in touch with live updating events from your app as they navigate outside of your app.

# Live activities

We wanted to better accommodate too many notifications.  Follow an activity.

A few examples of how to take advantage of new capabilities.  Anything someone wants to keep track of can be an activity  Exciting new ideas to explore.

Live at the top of the list, alongside notifications.  Same 14 point margins around eahc of their edges.  Even though they use different layouts, elements can cleanly align.

Create a layout that is unique.  Rich layouts are a key advantage.  

Let's talk about personality of live activity.  As you design a layout, consider colors, iconography, typefaces, etc.  Goal is when your live activity and app share same aesthetic and personality.

Draw inspiration from app icon, colors, etc.  Don't alternate colors for light/dark if it breaks this association.   Change colors based on content?  Integrate a logo 'uncontianed'.

Check that the clear button looks correct and matches design.

Spacing - Reduce height of your design to make more compact.  Be efficient.  Change height between moments as you have more/less info to display.  Transition to taller as you have more info to show.

Transitions.  When updating between different moments, apply transitions to individual elements themselves.  Use numeric content transition to count up/down.  For animating in and out grpahic elements, use content replace transition.  

Combine differet animations of scale, opacity, and position of elements.  Pins can smoothly transition to new locations.  


# Lock screen
Alerting.  Alert with your live activity to notify when there's an update requiring attention.  Emphasize the information that caused the alert in your layout during this transition.

Remove it from the lock screen after a short duration of time.

# standby

Great for displaying live activities, etc.  Layout is caled up 200% to maximize size.  Avoid drawing content in your layout under device's sensor region.  System automtaically extends bg color to fill remaining space.  As bg gets extended, things get cutoff.  Consider using a dividing line or containing shape.

Consider removing bg entirely in standby.  Layout can be slightly larger scale since we dont' have sensor margins.  

Ensure assets are high resolution to be displayed at this size.

Ensure your colors have enough contrast for night mode.

# dynamic island
By designing uniaque and vibrant experiences, they have a distinct identity.  Anchor each one in your mind.  

Look of these exepriences are varied, and yet family together in their own way.  Use of animation and continuously updating data feels more alive.  Extra rounded, thicker weights, heavier text, etc.  Use of color.

Make shapes concentric with the island shape.  Even margins, etc.  Achieve a good fit.

Visual mass or center of the shape should nestle inside the walls of the island nicely.  Consider blurring the object and make sure the result is concentric to the border.  When placing objects or text, make a concentric margin. 

dont' draw all the way to the edge.  Place inside an inset shape or use a separator line.

* compact view (grows horizontally)
* expanded (horizontal/vertical)
* minimal (kinda iconish)

compact -> most common.  Reflect the identity of your app.  Content snug against sensor region.  Fill as much of the view with content as possible.  Consider truncating space or sohwing less precise.  Make as narrow as possible.

Consider ticking between multiple displays.  Consider expanding the island to do an alert, rather than relying on a push.  DI floats above apps.  Dont' draw UI pointing to the island or that interacts with it.

People can press on the DI to zoom in.  Try to get to the essence of your activity here.  Maintain relative placement of elements between two views.

Can be taller, or more pill-sized.  Avoid height on the edge caught int he middle between these.

Try to hug the sensor as tightly as possible, and wrap content all the way around.

Minimal view -> juggling between multiple sessions at once.  Avoid reverting ot purely just a log here.  Convey information even in a tiny state.  

New kind of multitasking.  Unlock experiences that weren't possible before.  


# Resources
* https://developer.apple.com/documentation/ActivityKit/displaying-live-data-with-live-activities
* https://developer.apple.com/design/human-interface-guidelines/components/system-experiences/live-activities
