#sfsymbols 

A symbol is one of the most important parts of the UI.  Essential part of interface design.
Symbol can bring many benefits.  User interaction, space-efficient.  Enhancing the aesthetic appeal, and being user-friendly.  Transcend language.  

We care deeply about making UX better, SF symbols, an extensive library of iconography designed to integrate seamlessly with San Francisco.  Creating experiences on apll apple platforms.

Designed with typography in mind.  Awesome features such as weights, scales, outlines, fields variants, shapes, alignment.  To learn more about these features, see [[What's new in SF Symbols]]

# New symbols
great additions for homes: lights, blinds, windows, doors.  Light switches/power outlets.
Furniture and appliances.
Health symbols.

This year, our fitness figures are available.  Expanded the library of currency symbols, many new object s to choose from.  We keep expanding our localized symbols with RTL, scripts, etc.

700+ new symbols, making SFSymbols a library of more than 4,000 symbols.

5 new categories that we think will be super hepful.
1.  camera/photos
2. accessibility
3. privacy/security
4. home
5. fitness

create your own collection!
# Rendering modes
As you may know, 4 rendering modes
* Monochrome - most neutral.  Uniform and consistent look, most locally reflects the typographic nature
* Hierarchical  – single color hue.  Depth by highlighting the most improtant part or differentiate foreground/background.  Visual hierarchy that emphasizes part.
* Palette => two or more contrasting colors.  Symbols can be customized to integrate into a color palette in the environment.  Contrasting and unique look without fighting your aesthetic
* Multicolor => intrinsic, coordinated color of symbols.  Range of colors that can be applied to describe the apeparance of an object in the world.  Use multicolor when the symbols are very prominent as this will help createa  color narrative.

Previously, you get monochrome by default.  This year, each symbol has its own preferred rendering mode.  

## Automatic rendering
Provides a preferred rendering mode for each symbol without having to specify it manually.

e.g. 
* camera filter => hierarchical.  Highlights the opacity/translucency of the filters
* shareplay => hierarchical.  Lets the shape stand prominently while waves play a secondary role.

Be aware of the context.  For example, airpods will render as hierarchical.  But if you put it on white, the symbol is very small and has low contrast.  Remember that rendering modes can still be exclusively specified for uniform materials across symbols in a particular context.  Consider monochrome in this example.


# Variable color
Color is a powerful tool and we can explore it further.  Some symbols are more dynamic.  volume, signal strength, etc.

With a range of symbols into layers, organize layers into sequential order, creating a new method of distributing color to these layers.  Allows us to ocmmunicate different levels of strength, etc.

Some symbols have all paths embedded in the sequence.  In other symbols, only a subset.

Important to know that we define how we want to group the paths.  We group the paths following in to out.  No support for e.g. right to left.

Increasing or decreasing the level.  Like in the iPhone example, we have a pth that doesn't participate in the sequence.  paths that do participate, the three waves that define the volume strength.  Low volume, mid volume, high volume.  Organized into layers and the selected layer opts into the variable color feature.

Represent strength with percentage values.  0% off.  100% on.  in between – a mystery.

Variable color is not meant to create depth.  meant to highlight a sequence of steps or stages that a symbol can represent.  e.g., number of people in a room.

No limits to how many paths can opt into the feature.  You decide your strategy.

If you want to represent the strength level of shapes that follow a sequence, you can do so with variable color.  Available in **all** rendering modes.

# Unified annotations
How to annotate symbols that participate in variable color.

1.  Draw
2. erase

Used to help deifne the way a layer renders.  e.g., square overlaps circle.  If we choose draw, the layer will draw the path contained in that layer.  If we choose erase, the llayer will erase the path.

Multiple paths can be defined for a single layer.  But layers get the color I guess?

organizing things this way gives us more flexibility for multiple rendering modes.

Multicolor.  I already ahve the shape set up, so I just need to choose the right colors.  Follow SFSymbols for badge/plus.

We already defined multicolor.  now let's look at hierarchical and palette.  I would expect the badge to be primary.  Now i need the plus shape to erase part of the badge.  Evidently the plus erases the badge.  Unclear to me why that is distinct from drawing black, but they suggest it is.

Kitchen timer example.  Each segment in its own layer.  need to organize the shapes for variable color.
for more,
[[Adopt variable color in SF Symbols]]

developer.apple.com/sf-symbols/


