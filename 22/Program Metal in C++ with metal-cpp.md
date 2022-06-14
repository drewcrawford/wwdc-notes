#metal 

# Overview of metal-cpp
metal-cpp.  Hub between c++ and objc.  Your application can use metal classes and functions in C++.  

Features
* lightweight metal c++ wrapper
* all headers
* all metal APIs <-> C++ calls
* Parts of foundation and quartzcore
* Apple maintained
* Open source

Uses C to call directly into the ObjC runtime
"same mecahnism that the objc compiler uses"
little overhead
Cocoa-style memory management
Developer tools support
* GPU frame capture
* xcode debugger
```cpp
MTL::CommandBuffer* pCmd = _pCommandQueue->commandBuffer();
MTL::RenderCommandEncoder* pEnc = pCmd->renderCommandEncoder( pRpd );
pEnc->setRenderPipelineState( _pPSO );
pEnc->drawPrimitives( MTL::PrimitiveTypeTriangle, 
                      NS::UInteger(0), 
                      NS::UInteger(3));
pEnc->endEncoding();
pCmd->presentDrawable( pView->currentDrawable() );
pCmd->commit();
```

We now provide lighting sample with metal-cpp.  Hope that helps!

developer.apple.com/metal/cpp/

1.  Download
2.  add metal-cpp folder to header search paths
3.  enable c++17 or better
4.  Add frameworks (foundation,quartzcore,metal)
5.  generate the implementation by using some macros

```cpp
#define NS_PRIVATE_IMPLEMENTATION
#define CA_PRIVATE_IMPLEMENTATION
#define MTL_PRIVATE_IMPLEMENTATION
```
Do not define the NS,CA,or MTL macros more than once.  

# Manage object lifecycle
Allocate and release memory
manage command buffers, pipeline objects, resources, ...
objc and cocoa objects use a reference count.

reference counting helps you manage thelifecycle of an object
objects contain a `retainCount` property

manual retain-release (MRR) vs ARC

**metal-cpp uses MRR**

* you own any object you create
* startging with `alloc` `new` `copy` `mutableCopy` or `create`
* retain takes ownership
* release gives up ownership
* autorelease delays the release
* don't release objects you don't own

example
autoreleasepools, etc.  AP Takes ownership of the object, decrements the retainCount when pool is destroyed.

**metal creates autoreleased objects too**
add temporary objects to autorelease pool
create and manage local APs.

```cpp
NS::AutoreleasePool* pPool = NS::AutoreleasePool::alloc()->init();
MTL::CommandBuffer* pCmd = _pCommandQueue->commandBuffer();
MTL::RenderPassDescriptor* pRpd = pView->currentRenderPassDescriptor();
MTL::RenderCommandEncoder* pEnc = pCmd->renderCommandEncoder( pRpd );
pEnc->endEncoding();
pCmd->presentDrawable( pView->currentDrawable() );
pCmd->commit();
pPool->release();
```

# Object lifecycle utilities

## `NS::SharedPtr`
`NS::SharedPtr` helps manage object lifecycle, under Foundation framework.
Not the same as `std::shared_ptr`, so no dependency on c++ stdlib.

```cpp
{
   auto ptr = NS::TransferPtr( pMRR );
   // Do something with ptr 
   . . . 
}
```
copy and move ocnstruction and assignment
one release on destruction
detach (no release)

|                 | use case         | ref count  | destruction                        |
| --------------- | ---------------- | ---------- | ---------------------------------- |
| NS::TransferPtr | Take ownership   | No changes | CAll `release`, reference count -1 |
| NS::RetainPtr   | retain an object | +1         | Call `release`, reference count -1                                   |

## NSZombie
Use-after-free bugs
NSZombie, via `NSZombieEnabled=YES`
scheme=>run=>diagnostic=>zombie objects

[[Profile and optimize your game's memory]]

# Interface with ObjC
Ex
* viewcontroller: objc
* renderer: c++ with metal-cpp
* challenge
* call renderer(c++) in VC (ObjC)

Create an adapter which calls C++ from objc.

```objc
@interface AAPLRendererAdapter () 
{
    AAPLRenderer* _pRenderer;
}
@end

@implementation AAPLRendererAdapter
- (void)drawInMTKView:(MTKView *)pMtkView
{
    _pRenderer->draw((__bridge MTK::View*)pMtkView);
}

@end
```

reverse case
```cpp
CA::MetalDrawable* AAPLViewAdapter::currentDrawable() const
{
    return (__bridge CA::MetalDrawable*)[(__bridge MTKView *)m_pMTKView currentDrawable];
}

MTL::Texture* AAPLViewAdapter::depthStencilTexture() const
{
    return (__bridge MTL::Texture*)[(__bridge MTKView *)m_pMTKView depthStencilTexture];
}
```

## bridge casting
|                     | type cast                                    | ownership                |
| ------------------- | -------------------------------------------- | ------------------------ |
| `__bridge`          | Cast b/w objc and metal-cpp objects          | no changes               |
| `__bridge_retained` | Casts an objc poitner to a metal-cpp pointer | takes ownership from arc |
| `__bridge_transfer` | Move a metal-cpp pointer to objc             | transfer ownership to ARC                         |

If no transfer, you can use bridge casts.  If you want to cast from metal-cpp to objc objects, you should use bridge transfer casts.

```objc
MTL::Texture* newTextureFromCatalog( MTL::Device* pDevice, const char* name,   
                                     MTL::StorageMode storageMode, MTL::TextureUsage usage )
{
    NSDictionary<MTKTextureLoaderOption, id>* options = @{
            MTKTextureLoaderOptionTextureStorageMode : @( (MTLStorageMode)storageMode ),
            MTKTextureLoaderOptionTextureUsage : @( (MTLTextureUsage)usage )
    };

    MTKTextureLoader* textureLoader = [[MTKTextureLoader alloc] 
                                        initWithDevice:(__bridge id<MTLDevice>)pDevice];
    
    NSError* __autoreleasing err = nil;
    id< MTLTexture > texture = [textureLoader 
                        newTextureWithName:[NSString stringWithUTF8String:name] 
                               scaleFactor:1 
                                            bundle:nil 
                                           options:options 
                                             error:&err];

    return (__bridge_retained MTL::Texture*)texture;
}
```

# wrap up
adapter pattern
interface objc/c++

* metal-cpp is an efficient wrapper
* object lifecycle management
* elegant interface with objc
* apple developer tools

