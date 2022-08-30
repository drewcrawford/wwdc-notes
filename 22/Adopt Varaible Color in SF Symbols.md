Variable Color.  We'll be going over ohw to bring these symbols into your projects and how the app can help.

[[Explore the SF Symbols 3 app]]
[[What's new in SF Symbols 4]]

# Variable color overview
New feature of SF symbols taht allows you to affect the appearance of a symbol using a percentage value.  Easily create symbols that can change over time, like signal strength, or progress.

Jump into SF Symbols app.

## Demo
Preview area in the righthand panel.  See a symbol in every mode, at a glance.  Click on each representation to switch between rendering modes.

New automatic mode allows each symbol to choose its own preferred rendering mode, which you can see selected in the picker here.  

New controls for Variable Color.  New category called "Variable".  

## rEcap
* Used with every single rendering mode available for SF Symbols
* Every system symbol that supoprts VC, supports in all rendering modes.
* No rules for how many parts of a symbol can be affected by variable color.  ex 1, dozens, etc.  
* Because variable color is ocntrolled using percentages you don't need to *worry* about this.  lol
* how do we know if a particular layer will have variable color applied?
	* we took inspiration from the behavior of system-level indicators that you might be familiar with.

0% special case where no layers will be active
26% 
51%
76%

only empty at exactly 0%.  Similar to wifi strength or battery level indicators.

Can appear visually full at values less than 100%.  Ex Brightness, volume

3 layers for VC.  Threshold for layers that can occur at odd values like 33.333%.  We didn't want roudning errors to make symbols appear in unexpected ways, significant digits, etc.  Thersholds between layers are rounded to the nearest percentage point, and we don't activate the next layer until you're 1% above that values.

ex
0-33%
34-67%
68-100%

# Custom symbols

Make your custom symbols just as flexible and powerful as symbols provided by the system.

System-provided SF Symbols are available in 9 differeng weights.  Each weight is available at 3 scales.  Each one of those is available in four rendering modes.  Wtih and without variable color.
216 configurations.  

Last year we itnroduced variable templates.  You only draw 3 scales, system can generate the other 24.  After you've drawn your custom symbol, you can adopt different rendering modes through a process called "annotation".

Last year, you broke your symbol into different layers and assigned each a  hierarchy level.  To adopt multicolor, you broke your symbol into different layers again and assigned a color to each layer.  This meant that to support all rendering modes you needed separate layer structures.

This year we're doing unified annotation.  Single layer structure per symbol, shared across all rendering modes.  
Monochrome rendering.  In addition to the previous control you had over hierarchical, palette, and multicolor.
Variable color.

Last year, I was workign on an app to do card games.  a few months after that I learned a new obsession.  Puzzle cubes.  Practice solving them.

As I change the annotation, I can see how my custom symbol looks in different rendering modes.  Switching rendering modes to change accordingly.

Can drag from main picture into layers in the righthand panel.

When I switch to multicolor mode, I see the same information in the other mode.

Set colors of different layers.  The most important part of learning to solve is practice.  

Layers I want to fill in first go on the bottom, *last* go on *top*.  

Remember to keep an eye on the preview area to see what's happening in every rendering mode.  

Edits in one rendering mode can carry through to other rendering modes.  Often you just need to make changes for a single mode for it to carry over.

When all paths were in one layer, the cube paths created holes in the circle path which worked in monochrome.  But now that the circle is in its own layer, the cube paths no longer create holes.  Instead they create a solid cube on top of a solid surface.  Choose Erase to make a layer create a hole in the layers behind it.

## Recap
* Layer structure applies to all rendering modes
* Monochrome rendering mode
* Add variable colro to individual layers in the symbol
* Percentage thresholds are assigned using layer z-order.
* Percentage thresholds are spaced evenly between 0% and 100%
* Applies across all rendering modes

New layer options
* "Erase" uses a layers' shape to erase layers behind it
* "Hidden" excludes al ayer from a rendering mode
	* can apply that layer to certain modes only

4.0 template format supports monochrome rendering mode and variable color
Existing custom symbols will be automatically updated to unified annotation
Export 3.0 or earlier to maintain compatibility with earlier platforms.

Symbols with VC truly shine.  We could design our timer UI with text and numbers.  But they could be intimidating for someone who's still learning to read.  

Power of symbols.  Convey ideas that transcend language and text.  Make our apps more inclusive.  Variable color gives us even more expressive power for ocncepts like progress, signal strength, etc.



