Typical video workflow.  AVASsetWriter, Compressor, etc.

Can add artifacts.  Blockiness.  Blur.  

Evaluate video quality subjectively.

Or objectively?  AVQT Advanced Video Quality Tool.

# AVQT
macOS command line executable
Mimics how real people rate video quality
Computes frame level and segment level scores
Full support for AVFoundation video formats
SDR and HDR videos

# Key attributes
* Perceptually aligned
* Quick to compute
* Viewing hardware setup

## Perceptually aligned
* HIgh correlation with human opinions
* Works across content types – animation, natural scenes, etc.
* Traditional metrics – PSNR and SSIM fail in cross content analysis

Examples.

Performance on datasets.  Waterloo IVC 4k, VQEG HD3.

To make user correlation and distance measures, PCC.  Measures linear correlation

* Higher PCC implies higher correlation
* RMSE measures distance to subjective score
* Lower RMSE implies higher prediction accuracy

## Computational speed
* Optimized for Metal
* Native pre-processing - decoding and scaling

3840x2160 63fps
1920x1080 175fps
1280x720 261fps
960x540 335fps

## Viewing setup aware
Viewing setup affects subjective video quality
* Can mask or exaggerate video artifacts

* viewing distance
* display size
* display resolution

Takes parameters as input.  

A: 4k video viewed at 1.5 times screen height (viewing distance is)
B: 4k video viewed at 3 times screen height

As viewing distance increases, AVQT score increases as well.  We recommend you go over the readme doc for more information.

# Using AVQT
> soon via developer downloads

```bash
AVQT --reference sample_reference.mov --test sample_compressed.mov --output sample_output.csv
```

`viewing-distance`
`display-resolution`

# Optimize bit rates for HLS

Different content has different encoding complexity
Optimal bitrate varies across content

Use AVQT score as feedback

1.  Choose bit rate
2.  Encode source video
3.  Compute AVQT scores
4.  Increase/decrease bitrate

# Wrap up
* Video compression can lead to visible artifacts
* Evaluate your video's perceptual quality using AVQT
	* macOS command line tool
	* Quick to compute and viewing setup aware
	* Suports all AVFoundation based video formats
* Use AVQT to optimize quality of your HLS tiers

http://developer.apple.com/services-account/download?path=/Developer_Tools/Advanced_Video_Quality_Tool/AdvancedVideoQualityTool.dmg
