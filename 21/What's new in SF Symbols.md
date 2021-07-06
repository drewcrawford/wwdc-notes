#sfsymbols 

Simplest form of communication between user and the product.

Make interfaces faster to use.  Most symbols are language-independent.

Communicate content or trigger an action.  

Library of iconography.

# SF Symbols basics
Dynamic type, bold text accessiblity features.
Available in small, medium, large
Same point size.  

* Vertical alignment.  Vertical centered to... cap height?
* Horizontal alignment.  Horizontally centered to give a system look.

Symbol is part of a visual system that conveys meaning.

Universal variant => outline.  Kinda like typography.

Fills variant is a great choice for selections.

Symbol may come in different forms.  e.g. outline, filled, slash.  (inactive/unavailable)

Circle, square, rectangle.

More flexibility for interface elements and contents.  A clear visual system will go to great lengths for complexity etc.

Filled variant, system appearance.  

Outline: toolbars, navigation bars, etc.  Where symbols are presented alongside text or uniform appearance.

Remember to be consistent when designing symbols.  



# New symbols additions
600 new symbols available this year.  New apple products/devices.  New additions for controllers.  Health-related symbols.  Communication.  

watchOS.  New objects.  3000+ symbols.


# Localization
International and multicultural audiences.  Symbols for hebrew, arabic, etc.  Thai, chinese, japanese, korean.

Easy to use localized symbols automatically based on user device language, e.g. rtl, etc.  Battery icon flips.


# Anatomy of a symbol
Every symbol starts with a vector form.  

Each source has the same number of vector points.  Then we interpolate to get variable items.  Important to achieve consistent shape.

[[Create custom symbols]]

A stroke is a given thickness value assigned to follow a path.  Although a stroke can haveany thickness, it uses the path as the center.  So the actual vector points are not relaible for interpolation, since thereare no points on the borders.

Stroke must be converted to outlines.  
Determine that fg/bg elements in just one symbol.  Different variants with different shapes.  Done with annotations.  We group like layers inside a symbol.

* Primary
* secondary
* tertiary


# Rendering modes
Highly versatile when color is applied, making them a more powerful tool.  When color is present, symbols become one of the most essential methods.  Helping create a statement or draw attention.

* Hierarchical
* palette
* Multicolor
* monochrome

## Hierarchical
Single color but opacity for layers.

Makes symbols more legibile.  Reduces visual noise by modulating opacity without removing essential elements.

Base color applied to all layers using different opacities.  

Note that most symbols don't have all 3 levels.  

Shapes that don't touch or have a gap are considered secondary.  Shapes that do touch, and are encapsulated, are tertiary.

## Palette

Use 2 or more contrasting colors.  Gives elements of a symbol more prominence and versatility.

Symbols can be customized to better integrate with the UI.  Relies on same annotation data.

Tertiary layer will replicate secondary layer (if it isn't specified).

Same defined color palette, helping the user identify a group of symbols with similar functionality.  In contrast, different colors can be distinct.

## Multicolor
Rendering code that represents the *intrinsic* color of the symbol.  e.g. physical world, meaning, etc. 

Many symbols have an intrinsic palette, rendered by default.  Colors can be assigned to an arbitrary color.

Several symbols will have multicolor functionality.

## Monochrome
Typographic nature.  Neutral, allows ymbols to commingle without emphasizing any of them.

Beneifts from color and opacity controls, but the difference is that monochrome, the opacity applies to all without layer annotations.

When uniformity is of importance, monochrome is ideal since it's unified/consistent.
# SF Symbols app

As we've seen, new/improved rendering modes in righthand inspector.

Color picker.  Light, dark, increased contrast variants.

Some platforms will get new colors from other platforms.  e.g. brown, was only available on macOS.

Teal => Cyan

[[Explore the SF Symbols 3 app]]

# Recap
* App
* Variants
* Localized symbols
* Anatomy of a symbol
* Color, rendering modes
* Multicolor, monochrome

Adjusts vibrancy, appearance.  Work with custom colors as well.

