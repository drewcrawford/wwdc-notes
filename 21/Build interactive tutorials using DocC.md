#docc 

You can learn more about using DocC 

[[Meet DocC documentation in Xcode]]
[[Elevate your DocC documentation in Xcode]]

I'll focus on writing tutorials.  A highly interactive form of documentation.

Markdown *directives*.  Apps should use API in a realistic way.

ex, in SwiftUI, we show you how to build an app called landmarks.

# Overview
Highly interactive way to learn.  Like article and reference documentation, they're authored in markdown with a special syntax

`@Step`, `@Code`, etc.  Nest directives.

1.  TOC starts with a tutorials directive.
	1.    Intro.  Some elements generated automatically
		1.    Image
	2.    Chapter, organize into groups that make sense.
		1.    Image
		2.    TutorialReference

1.  Tutorial
	1.  Intro.  Title, description.
		1.  Image
	2.  Section.  
		1.  ContentAndMedia
			1.  Image
		2.  Steps
			1.  Step

# Planning
Consistent and clear tutorial steps.  Easy to create a cohesive set of steps through a collection of tutorials if you start with a clear plan.

Group them by general area.
1.  Creating
2.  Caring
3.  Interacting

These are chapters, sections for each chapter that involve some demo app.

# Creation

[[Elevate your DocC documentation in Xcode]]

DocC provides a template for me to start with.  In tutorial file template, there's an intro, a section, and a set of steps.

Use alt text for accessibility

Keep in mind background could be light or dark.  So either create a compatible image, or provide a separate image for each appearance.

Seems `@Code` steps tend to link to an external file.

# Wrap up
* Overview of tutorials
* What makes a good tutorial
* Outlining tutorials
* Using DocC to write and preview tutorials

[[Meet DocC documentation in Xcode]]
[[Elevate your DocC documentation in Xcode]]
[[Host and automate your DocC documentation]]

* https://developer.apple.com/documentation/DocC
* https://developer.apple.com/documentation/DocC/tutorial-syntax
* https://developer.apple.com/documentation/DocC/building-an-interactive-tutorial
* https://developer.apple.com/documentation/xcode/slothcreator_building_docc_documentation_in_xcode

