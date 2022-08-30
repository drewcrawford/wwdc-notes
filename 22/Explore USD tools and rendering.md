#usd 
# USD
Content creation tools.  More and more ways of creating assets, etc.  rendering for games, AR, films, and the web.  All with USD at the center.

two parts of the ecosystem
* tools
* rendering

# USD tools
* command-line inrterface
* essential python USD tools
* `usdzconvert`: Create USDZ from other 3d file formats
* Automation and education
tools updates
* Python 3 support
* Apple silicon support
* Upgraded USD version

```bash
% python usdzconvert --help
usdzconvert 0.66
usage: usdzconvert inputFile [outputFile]
                   [-h] [-version] [-f file] [-v]
                   [-path path[+path2[...]]]
                   [-url url]
                   [-copyright copyright]
                   [-copytextures]
                   [-metersPerUnit value]
                   // ...
                   [-diffuseColor           r,g,b]
                   [-diffuseColor           <file> fr,fg,fb]
                   [-normal                 x,y,z]
                   [-normal                 <file> fx,fy,fz]
                   // ...
```

* .objc: materials
* .gltf: points, lines, KHR_materials_clearcoat
* .fbx: scene time for animations

#realityconverter : built with usdz tools.  Making it easy ot convert, view, etc.

This year we added support for material customization.  

Can reduce size by compressing textures.  80% file size reduction by compressing to JPEG.

## Fix common issues with USD assets
* correct mismatched attributes
* fix a stage with multiple top level prims and no default prim
* update deprecated attributes
* add missing stage metadata

Added support for universal binaries.

## USD content creation
* object capture
* Reality Composer
* RoomPlan API

[[The ARtist's AR Toolkit]]
[[Create 3D models with Object Capture]]
[[Create parametric 3D room scans with RoomPlan]]



# Lighting in AR Quick Look
Place virtual content in real world
Supporst models and interactive scenes
interact and share

* Brighter
* enhanced contrast
* improved shape definition

brighter color, higher contrast, and more highlights.

```usd
// asset.usda
#usda 1.0
(
    customLayerData = {
        dictionary Apple = {
             int preferredIblVersion = 2
        }
    }
)
```

0 => default lighting
1=> classic lighting
2=> new lighting

we recommend making an explicit choice for each asset.  To do that, either use `usdzconvert` with `--preferrediblversion 2`.  Or RealityConverter.  Which uses new lighting by default.

AR quicklook will determine lighting version automatically based on file's state timestamp.

If asset was created after uly 1 2022, it will use new lighting
older assets willc ontinue to use classic lighting.


# USD rendering
Takes a collection of 3d models, cameras, lights, etc., and uses themt o generate an image.  many different renderers, each one is built for specific purpose and optimzed for different use.  Unique set of features.  

Different USD renderers support different usecases.
coming to developer.apple.com
render support matrix
USD asset recommendations

## RealityKit
* photorealistic rendering
* Interactive experience
* Used in AR Quick Look
* Primary USDZ renderer

## Storm
* Open source from Pixar
* Optimized for large scale scenes
* real-time renderer (preview?)
* Uses Hydra.  

## What is Hydra?
Storm produces a preview render.  Hydra acts as an abstract layer that lets you transport data from scenes to renderers.  Can swap out renderer without changes to scene graph.  ex, Storm + RenderMan.

Hydra also lets you switch scene graph.  USD or a different one.  

Interfaces on each side of hydra are called *delegates*.  Front end *scene delegate*, backend *render delegate*.

## Metal support for USD
* full Metal support for Storm
* Collaboration with Pixar
* USD 2205

# Integrate Hydra
Our sample app is very simple but powerful.  Renders a USD file with storm, and lets us move teh camera around it.

* Samplep roject with Metal Support => HydraPlayer
* Available in session resources
* Instructions in ReadMe.
## Project structure
* app Delegate
	* Manages some itneractions with the system
* camera
	* Representation of USD scene camera
* renderer
	* Handles all interactions with USD and Hydra
* VC
	* Handles user input

## Build USD + Hydra
* prepare environment
* Rosetta
* Download and build source

### Prepare
Install xcode
python
cmake

### rosetta
```bash
// Rosetta
% arch -x86_64 /bin/zsh

// Download source code
% git clone https://github.com/PixarAnimationStudios/USD.git 

// Build USD + Hydra
% python3 USD/build_scripts/build_usd.py --generator Xcode --no-python USDInstall
```

## Key code parts
### Load USD

```objc
// AAPLViewController.mm

- (void)viewDidAppear
{   
    NSOpenPanel* panel = [NSOpenPanel openPanel];
    panel.allowedContentTypes = @[UTTypeUSD, UTTypeUSDZ];
   
    [panel beginWithCompletionHandler:^(NSModalResponse result) {
        if (result == NSModalResponseOK)
        {
            NSURL* url = panel.URLs[0];
            [self->_renderer setupScene:[url path]];
        }
    }];
}

// AAPLRenderer.mm

- (bool)loadStage:(NSString*)filePath
{
    _stage = UsdStage::Open([filePath UTF8String]);
    // ...
}
```
### Setup camera

```objc
// AAPLRenderer.mm

- (void)setupCamera
{
    _viewCamera = [[AAPLCamera alloc] initWithRenderer:self];
    
    [self calculateWorldCenterAndSize];
    
    [_viewCamera setDistance:_worldSize];
    [_viewCamera setFocus:_worldCenter];

}
```

### Lighting

```objc
// AAPLRenderer.mm

GlfSimpleLight computeCameraLight(const GfMatrix4d& cameraTransform)
{
    GlfSimpleLight light;
    light.SetPosition(GfVec4f(cameraPosition[0], cameraPosition[1], cameraPosition[2], 1));
    
    return light;
}
```
### Transfer to Storm
```objc
// AAPLRenderer.mm

- (void)initializeEngine
{
    _engine.reset(new UsdImagingGLEngine(_stage->GetPseudoRoot().GetPath(),
                                         excludedPaths,
                                         SdfPathVector(),
                                         SdfPath::AbsoluteRootPath(),
                                         driver));
}

// AAPLRenderer.mm

- (HgiTextureHandle)drawWithHydraAt:(double)timeCode
                           viewSize:(CGSize)viewSize
{
      _engine->SetCameraState(modelViewMatrix, projMatrix);
      _engine->SetLightingState(lights, _material, _sceneAmbient);
  
      UsdImagingGLRenderParams params;
      params.clearColor = GfVec4f(0.0f, 0.0f, 0.0f, 0.0f);
      params.frame = timeCode;

      // ... 
}
```

# Download sample project
* available in resources
* instructions in readme
# Wrapup
* Powerful USD tool updates
* Vibrant new lighting
* flexible rendering
* HydraPlayer sample

[[Understand USD fundamentals]]

* https://developer.apple.com/forums/tags/wwdc2022-10141
* https://developer.apple.com/forums/create/question?&tag1=251&tag2=456030
* https://developer.apple.com/documentation/metal/metal_sample_code_library/creating_a_3d_application_with_hydra_rendering
* https://graphics.pixar.com/usd/docs/index.html
* https://wiki.aswf.io/display/WGUSD/USD+Working+Group
* https://developer.apple.com/documentation/arkit/usdz_schemas_for_ar
