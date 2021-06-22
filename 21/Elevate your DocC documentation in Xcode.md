#docc #xcode 

Documentation for swift frameworks

Create new adorable virtual sloths.

# Page types
* Documentation catalog
* Reference
	* Concise, in-depth information.
	* Backbone
* Articles
	* Free-form content.
	* Explaining how to complete a specific task.
	* Connect dots between different symbols
* Tutorials
	* Step-by-step walkthrough of a project that uses your framework.
	* Beginner-friendly format

Here we will focus on *Articles*

[[Meet DocC documentation in Xcode]]
[[Build interactive tutorials using DocC]]

See apple technologies, and your own documentation.  

Product=>Build documentation.

If I approach this page from the perspective of someone who's never used this framework, it's unclear what it does.

Top level article => paints a big picture. 
Bototm: topics section


To add an article, I need to setup my project with a documentation catalog.  In xcode navigator that contains my documentation files.

Quick to access at a glance.  Makes it easier to prioritze documentation in my day-to-day workflow.

New file => Documentation catalog.
Name it to match my framework target.  Xcode gives me a top-level article.  Rename it to whatever.

```markdown
# ``SlothCreator``

Catalog sloths you find in nature and create new adorable virtual sloths.

## Overview

SlothCreator provides models and utilities for creating, tracking, and caring for sloths. The framework provides structures to model an individual ``Sloth``, and identify them by key characteristics, including their ``Sloth/name`` and special supernatural ``Sloth/power-swift.property``. You can create your own custom sloths using a ``SlothGenerator``, and name them using a ``NameGenerator``.

Sloths need careful feeding and maintenance to ensure their health and happiness. You maintain their ``Sloth/energyLevel`` by providing the correct ``Sloth/Food`` and a suitable ``Habitat``. You can exercise your sloth by providing a fun or restful ``Activity``. 

You can visualize and observe your sloths by adding a ``SlothView`` to a SwiftUI view structure.

![A sloth hanging off a tree.](sloth.png)
```

I'd recommend adding @2x, and either dark mode-compatible, or have an alternative.

Don't forget `sloth~dark@2x.png` format.

Assets go into "Resources" folder.

Xcode automatically selects the right variant.  just use `sloth.png` in markdown

# Organization
Another way to guide everyone through the framework.

DocC compiler created a topic session.  But we can improve it.  

* Creation
* Health/happiness
* Visualization

Organize documentation pages into categories
Sort from least to most advanced.

Framework adopters interested in specific features will know where to look.

`- <doc:GettingStarted>`
``- Symbol``

Warnings for bad references.

Getting started article is the first one.

1.  Pages are ordered automatically and can customize with a `## Topics` section.
2.  Add a group with a third-level heading, followed by a list of links `### Essentials`.  

-----
1.  Choose your framework's themes
2.  Organize supporting pages under primary pages
3.  Sort by complexity

# Extensions
Extensions give me the flexiblity to choose how I write documentation.

First, create a new markdown file.  Associate it with an API using two backticks in title.

To improve focus and clarity, I'll leave primary content in code, and extract the topic section.

When docc builds, it merges each source comment and the doc extension, into a single documentation page.  

In project navigator, right click on documentation and add file.  In template chooser, select "Extension File".

Connect extension file to type using link in title.  Include name of module.

```
# ``SlothCreator/Sloth``
## Topics
### Creating a sloth
- etc.
```

Navigator is updated as well.

[[Host and automate your DocC documentation]]

How can you elevate your documentation

# Wrap up
* Add articles
* Group related pages
* Use documentation extensions

[[Host and automate your DocC documentation]]
[[Build interactive tutorials using DocC]]
