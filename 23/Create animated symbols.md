Discover animation presets and learn how to use them with SF Symbols and custom symbols. We'll show you how to experiment with different options and configurations to find the perfect animation for your app. Learn how to update custom symbols for animation using annotation features, find out how to modify your custom symbols with symbol components, and explore the redesigned export process to help keep symbols looking great on all platforms. To get the most out of this session, check out “What's new in SF Symbols 5” from WWDC23.

# Previewing animations

bounce direction, animate by whole symbol or by layer, etc.

copy code in swift or objc.

variable color: cumulative, iterative, etc.
# Animating custom symbols

pulse by layer - need to mark individual layers.  If you don't, the whole symbol will pulse.

by default, every layer moves independently with 'by layer'.  Might be a lot.  Create groups of layers that animate all sublayers together.  All layers inside the group keep their own annotations and settings, but they all move together.

by grouping layers together, we make sure they move as a single unit.  Look at your custom symbols individually and decide what motions are right for each one.

circular borders, modifiers, slashes, etc.  Consistency of these treatments make SF symbols recognizable and familiar.  Custom symbols should try to match these treatments.  How can I make sure my custom symbols are consistent?

# Symbol components

provides artwork and behavior that looks and feels like  a system-provided SF symbol.  Combine your symbol with a system component.

context menu, "combine symbol with components".  colored correctly, animates correctly, etc.

depending on appearance and design, may want to make some adjustments in inspector.  ex, nudge the badge to a different offset.  Change length of a slash, etc.

ensure it looks great in different scales and weights.  a total of 27 different variations.  Good news is that symbol components are powered by variable templates which take care of most of the work for you.

you only need to provide ultralight, regular, and black weights.  System can mix your drawings together to create the remaining 24 possible variants.  24 symbols for the price of 3.

components work the same way – you have control over those 3 weights.  

you can check 'override' checkbox in the other weights and adjust.  

as base symbol is scaled down to fit the enclosure, system will use the variable template to increase weight.  helps to keep looking consistent.

* custom symbols can be combined with components to create new symbols
* base custom symbols must use variable templates
* overrides in variable templates are ignored
* editable templates cannot be exported from symbols created with components
# Compatibility

when exporting a template you have a 'compatability' picker.  By default you get latest/greatest symbols.  If targeting older platforms, you can export that.

you can set a symbol to target a specific platform.  also speicfy what version of xcode you're using.  Helps make sure the file will work properly.




