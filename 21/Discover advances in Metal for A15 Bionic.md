#metal #a15

# New GPU enhancements
TBDR, etc.

* Performance
	* 5 shader cores.  5th core provides additional 25%
	* FP32 math units
* Responsiveness
* Power efficiency
* `MTLGPUFamilyApple8`

* lossy compression
* sparse depth and stencil
* SIMD shuffle and fill

## Lossy compression
First let's revisit lossless:
* Saves memory bandwidth on texture access
* Maintains image quality
* Applied automatically to textures where possible
* [[Discover Metal enhancements for A14 Bionic]]
* "Optimizing Texture Data" article

Lossy compression:
* A15
* 50% memory saving
* Quality preserved where possible
* Applicable to render targets

`textDesc.compressionType = .lossy`

### Why?
* Reduces bandwidth
	* With lossless, maybe we can't compress.
	* But here, can save 50% of memory footprint, guaranteed
* Minimal loss in quality

### when?
* Most pixel formats and texture types
* No other modifications to your app
* whenever quality tradeoff is acceptable to you
* Final and intermediate render targets are best
* Review post-processing chain to find candidates

Evidently the comparison is good but I can't tell due to poor resolution on this video

Evidently you should consider increasing texture size.  

Texture types:
Suported: 2d, 3d, array, cube
Unsupported: 1D, linear, texture buffers

Pixel formats:
Supported: standard, 8, 16, 32-bit, 10/10/10/2 and 10-bit XR
Unsupported: Packed (5/6/5, 4/4/4/4, 5/5/5/1, 11/11/10 float, 9/9/9/e5 float)

Operations:
Supported: render, blit, sample, and read
Unsupported: Shader write

Storage: 
Supported: Private, unsupported: Shared, managed

Features: Supported: MSAA, sRGB, mipmapping
Unsupported: ??

* Compression saves bandwidth
* 50% memory saving on lossy render targets
* Preserves texture detail where possible
* Easy to enable and use



## Sparse depth and stencil
Map and unmap texture type from GPU.
[[Metal enhancements for A13 Bionic]]
[[Tailor your Metal apps for Apple M1]]

enables sparse depth and stencil attachments.

Don't allocate what you won't use!  E.g., stuff behind UI elements.  App can leave those unmapped.

### Shadow maps.  
A large portion of the shadow texture does not need to be mapped because the lighting pass will not sample, outside the view frustrum.

Cascaded maps.  Instead of using a fixed-resolution texture, use sparse maps across the chain.  Only maps the tiles needed for the sampling rate.

1.  Generate density map based on sampling rate
	1.  Geometry map populates the den.  Track shadow-space derivatives of renderred geometry
```cpp
uint2 tileIndex = uint2(shadowUv * kGridRes));
float dUV = fwidth(shadowUv);

//atomics to store max derivatives in grid
float m = round(max(0.0, -log2(dUV)));
atomicMaxToGrid(tileIndex, m);
```
2.  Construct surface and map 
3.  Render sparse tiled shadow map

Map tiles of the depth texture by iteratively dividing the surface and scheduling mapping of each level.

1.  Check sampling rate of current mip from density map.  
2.  Promote the whole tile by mapping the right mip

Write a 2d table that translates between UV and mip labels
```cpp
float tocSample(texture2d tex, float2 uv, char *toc) {
	// kTocRes = mip0 res / tile size
	uint2 idx = uint2(uv * float2(kTocRes));
	uint lod = toc[idx.x + idx.y * kTocRes];
	
	return tex.sample2d(kSampler,uv,level(lod));
}
```

Fill each mipmap with an indirect comand buffer.  Compute passes can pull and sort each shadow geometry mesh in parallel.  For large objects that stretch acorss the shadowmap, shader tests the relevant tiles of each mip and encodes a draw command if at least 1 tile overlaps.

If no tiles overlap, don't emit a draw command to that mip's ICB.  The optimized draw commands is encoded by the shadow cull compute pass.


|                  | Single Map | CSM (3 cascades) | STSM               |
|------------------|------------|------------------|--------------------|
| Resolution       | 16K        | 3x1K             | 16K                |
| Upsampled tiles  | ~75%       | 50%              | 0%                 |
| Memory footprint | 1024MB     | 12MB             | 1365MB, 8MB mapped |


## SIMD shuffle and fill
In modern image processing, convolution kernels are applied for edge detection, blur, and sharpness.

May be limited by texture sampling or threadgroup memory.  But we have SIMD instructions to optimize workloads.

SIMD group functions share data between threads using registers.

[[Metal enhancements for A13 Bionic]]
[[Discover Metal enhancements for A14 Bionic]]

`simd_shuffle_and_fill_up`, `simd_shuffle_and_fill_down` `quad_shuffle_and_fill_up`, down.
* Designed for sliding-window image processing
* Allow data sharing between threads without using memory

```
shift = 1
quad_shuffle_down(data,shift)

ABCD
|
v
BCDD (doesn't wraparound)

but fill down

ABCD EFGH
BCDE
```

The idea is we have a 'fill buffer' and shuffle in from that.

simd is 32 wide.  

Also have a modulo argument.  For modulo 8 it's effectively split into 4 vectors, each one is shuffled separately.

Application to convolution images I didn't completely follow


| Naive implementaiton       | SIMD shuffle/fill          |
|----------------------------|----------------------------|
| 25 samples per thread      | 4 samples per thread       |
| 800 samples per simd group | 128 samples per simd group |

Many common image processing and ML algorithsm can apply the same approach to optimize shared data across SIMD groups.

# Wrap up
* Lossy compression is easy to enable
* Sparse textures help create efficient, high-quality shadow maps
* SIMD shuffle and fill reduce overlap and improve sliding-window image operations
* All metal apps get an additional boost in perf, responsiveness, and power, from A15