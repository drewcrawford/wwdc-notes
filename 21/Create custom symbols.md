#sfsymbols 

How to create custom symbols?

# Custom symbol recap
If you need a custom symbol, you would draw it as a series of vector pads and place it in a custom symbol template.

Template is the mechanism for your own symbols to take advantage of various features.

This includes, typographical alginment.  Margins.  9 weights.  3 scales.

[[Introducing SF Symbols - 19]]
[[What's new in SF Symbols]]

# Template overview
XC13 or better.  Margins have more explicit names.  e.g. `Regular-M`.

Monochrome, multicolor, hierarchical and palette, etc.  

Full spec in updated documentation.  developer.apple.com/sf-symbols


# Editing custom symbols
Drop existing templates on the app and we'll upconvert (upgrade) to a 3.0 template.

* Static template setup
* Variable template setup

## Static
Similar to 2.0 template.  27 sets of paths, and 1 set of explicit margins in regular/medium scale.  If you plan to design 1/2 variants this setup will work well.

Variable will generate 3 sets of paths and 3 sets of margins.  If you plan on supporting all variants, you may be interested in supporting this setup.

Requires that all your paths have a high level of compatibility and consistency.  Revisit later.

I convert any live strokes to paths once I'm done with the design.  paths ensures that the shape can be filled with color later.  Can also make minor optical adjustments.  

* Avoid strokes, convert to paths
* Avoid open paths.  Paths should connect.  Open paths cannot be colored
* Avoid gradients and effects that involve more than one color.  This overrides any multicolor or hierarchical data for your symbol.
	* Pick a standard flat fill with no additional effect.

Preserving paths across design variants is a requirement if you want to use multicolor or hierarchical data.  Path number is used to color the item.  First color to first path, etc.




# Symbol annotation
Not required, but can control symbol appearance.

Set of layers for each rendering mode.  A layer is a collection of paths with associated data.  Layers in multicolor mdoe get a color, and in hierarchical mode are a hierarchy group.

Layers have exclusive front-back order.  No MC Escher symbols for you.

Recommended to use system-provided colors whenever possible.

To achieve a stroke fill effect, I set fill and use lower opacity.

Each layer has a toggle in hierarchical mode to force it to be opaque.  This is useful for overlapping shapes.  

Hierarchical and palette
* Hierarchical => one color.  Varying opacity based on hierarchy level.
* Pallette => 2-3 colors.  Differ slightly based on number of colors passed.  2 colors distributes to hierarchy groups.  
	* 3 colors => each one is applied to corresponding group.

To edit the symbol again
* Export template
* Edit monochrome representaiton
* Re-import and verify annotations

Be careful about removing or adding paths becuase it will go out of sync with annotation data.


# Advanced techniques
If you provide just 3 variants you get the rest of the template for free.

## Variable templates
Design sources.  Ultralight-S, Regular-S, Black-S.  If these are present, we check if paths match.

Path compatibility.  Same number and they "match up".  

Point compatibility: same number of points.  To be compatible, points must be 1:1 between corresponding paths.  Each pair of matching points creates an imaginary line, we move along the line to blend paths.  Blending (interpolation) allows us to move along paths.

1.  Design sources: Ultralight-S, Regular-S, Black-S
2.  Path compatibility
3.  Point compatibility

Best to copy/paste regular-S paths and only reposition the points from there.  Criticla to make sure your custom symbol can generate the other variants.

Avoid adding or removing points as this will break compatibility.  Instead, only adjust the existing points.


# Export and distribution

2.0 template, 3.0 template

## 2.0
* Use this for back-deploying
* Only contains monochrome
* One set of margins
* Not a source artifact

## 3.0
* Embedded annotation data
* Supports margins per variant
* Cannot back-deploy
* Editing can break annotations
* Not a source artifact
	* If you need to make more edits, import back into SFSymbols app.  

## deployment and sharing
For iOS 14 DT, export both. 
For iOS 15 DT, export 3.0

For sharing with a colleage, export 3.0

# Recap
* Editing a 3.0 template
* Annotation
* Advanced editing and interpolation
* Distribution

[[SF Symbols in UIKit and AppKit]]
[[SF Symbols in SwiftUI]]



