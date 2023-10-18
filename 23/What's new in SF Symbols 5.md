#sfsymbols 

# Animation
originalyl we had monochrome
layer we did more color
last year we had variable color

layers and animation.
Remember, each symbol in the library has a unified layer structure, ensuring that itsl ayers are arranged in the orrect order determines how color is applied.  As you can imagine, layers play an essential role in how a symbol animates.

By default, it animates 'by layer', each layer animates one at a time.
Choose to animate the whole symbol.
Dimensionality that symbols use to create a sense of depth.  Not visible, but they help us understand how symbols move and interact.

* middle -> normal
* front -> larger
* back -> smaller

* appear => kind of a scaling effect.
* disappear
used when appears int he interface.

* bounce
	* simulates an object bouncing
	* feedback that their interaction was successful, etc.
	* action has taken place
	* create a sense of energy
* scale
	* provides visual feedback bychanging the size of the symbol.  Scale up, scale down, etc.
	* visual feedback that it's acknowledged.  On the other hand, if it scales down and then up, it gives the impression of being pressed.  like a button.
	* ex consider a tab bar.  maybe we scale up the active tab.
how to choose bounce vs scale?
* boucne => action has occurred or needs to take place
* scale => provide focus or feedback when selected?  also stateful, so state persists until removed

* pulse
	* ongoing activity by changing its opacity.
	* 
* variable color
	* conveys varying levels of strength and relies on color to communicate state.
	* also part of animation library!
	* cumulative vs iterative
		* cumulative: layers one after the other while keeping previous state.  Great way to represent wifi symbol, etc.
		* iterative: highlights each layer 1 at a time.  Represents wifi symbol searching for networks.
		* also reversing option!
* replace
	* one symbol is swapped with another, and used to communicate changes in state.
	* by layer, or whole symbol.
	* directionality.  down->up.
	* up->up.  Sense of forward progression, allowing us to represent events 
	* Important to achieve right balance between extensions and functional interface, so that the experience doesn't result in something overwhelming.


# Drawing for animation

[[Create animated symbols]]

You want to use erase layers, so that we have info behind the top layer.  This is important when we animate and you see behind those layers.

Make sure to review info, may seem different when in motion!
# New symbols

over 700+ new symbols.  Now 5k symbols in total!

developer.apple.com/sf-symbols

* [[Create animated symbols]]
* [[Animate symbols in your app]]
* [[What's new in SF Symbols 4]]
* 
