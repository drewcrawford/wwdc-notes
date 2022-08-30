You may have come across USD aready #usd 

An important technology, let'st ake a look under teh hood.

# What is USD?
Developed by Pixar Animation Studios
Wide adoption, many industries
Designed to be extensible and collaborative
Open-source project

Core aspects of USD.
* scene description specification
* api
* rendering system

You interact with USD via the API.  
rendering sytsem provides support for rendering.  SEe [[Explore USD tools and rendering]]

In this session we focus on the scene description specification.  How data is described, organized, represented, etc.

Fundamentally, these USD files contain data describing how a scene should look.  Apps interpret the data.

# Fundamental concepts
USD has many features we want to talk about.  For the sake of time, we'll focus on essential features.  Stage, frame, layer, etc.

## Stage
* represents a scene graph
* composed of one or more layers
* layers are rtypically files with scene information
## Prim
* primary container object
* can contain other prims
### Type
* many different types in USD
	* meshes, lights, materials

How does USD know what these types mean?

## Schema
Structure data that defines the role of ap rim on a stage
Schemas for geometry, materials, and more

Here' sa schema definition of a sphere.  We define that every sphere has ar adius and bounding box extent.  USD gives a rich foundation of built-in types to describe scnees.

custom schemas let us extend USD.
* represent custom data
* do not require a visual representation

## Attributes
* prims contain attributes
* type value pairs
* Can ahve default value

## Metadata
Can be defined on stage, prim, attribute
Key value pair for auxiliary data


# Composing with USD
In production-centric proejcts, need to collaborate with different team members and organizations.

## Composition
* create a stage from multiple stages
* efficient data use
* collaboration and fast iteration
* Different types of composition
	* layering, references, payloads, variant sets

## Overview
chess set scene.  WE use a catalog of assets in a catalog layer.  Then we arrange on the chessboard in the layout layer.  final result: chess set layer.

## Referencing
* refers to prims instead of copying them
* reduce data duplication
* allows for collaboration

We can specify the default prim with the default prim metadata

### default prim
* defined on the stage
* default prim used when a stage is referenced
* recommend you author this

### Referencing by path
* can be different than the default prim
* explicit prim path
* allows to reference pirms in hierarchy

paths
* stage elemnt identifier
* prim path is a unique identifier

For larger scenes, it can be expensive to load all info at once.
* defer the loading of data being referenced, with **payload**
* useful for working with large scenes

## Layering
* stacked
* Higher layers are stronger
* Can add or override data in lower layers
* Non-destructive edits

Adjust positions with the layout layer.

## VariantSet
Dynamic swapping of discrete alternatives
variants contain various USD representations
Non-destructive switching

In our pawn asset, we add a variant set called color, so we can switch between different layers for the pawn.  Dark => dark material, etc.

Now we are back in our catalog layer.  Setup all board pieces with light material.  The default variant is set to the light material.

## Scene graph instancing
* resuse parts of scene graph multiple times
* Memory and performance improvements
* Defined by metadata
* Scene instances are candidates to share the same scene graph

`instanceable = true`.  Rather than duplicate data for each prim.  That's it basically.
## Review
* layering, referencing, payloads, and VariantSets are some types of composition
* defined strength order
* LIVRPS

# File formats
several types
* usda/usd => text.  Human readable files
* Crate (usdc/usd) binary.  Lossless, compact files
	* note usd can be either kind
* usdz => bundle of assets, single archive

# Wrap up
* USD fundamental concepts
	* stage
	* layers
	* prims
	* schemas
	* attributes
	* metadata
* USD composition
	* referencing
	* payloads
	* defaultPrim
	* prim paths
	* layering
	* instancing
* file formats
	* usda, usdc, usd, usdz


* https://developer.apple.com/forums/tags/wwdc2022-10129
* https://developer.apple.com/forums/create/question?&tag1=251&tag2=377030
* https://developer.apple.com/documentation/metal/metal_sample_code_library/creating_a_3d_application_with_hydra_rendering
* https://graphics.pixar.com/usd/docs/index.html
* https://wiki.aswf.io/display/WGUSD/USD+Working+Group
* https://developer.apple.com/documentation/arkit/usdz_schemas_for_ar
