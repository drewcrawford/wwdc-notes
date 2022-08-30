Today I'd like to talk about new features and enhancements we made in AVQT.

# What is AVQT?
Command-line executable.  Full reference video quality metric.  Intends to mimic how people rate the quality of videos.  Supports all AVFoundation-based formats.  HDR10, etc.

## Key attributes
* perceptually aligned
* Quick to compute
* Viewing setup aware

[[Evaluate videos with the Advanced Video Quality Tool]]

We presented the imrpessive processing speeds it achieves.  Since then, we have new processors

| processor | 4k 2160p | full hd 1080p |
| --------- | -------- | ------------- |
| m1        | 63       | 175           |
| m1 pro    | 91       | 186           |
| m1 max    | 112      | 254           |
| m1 ultra  | 145      | 262              |

# Enhancements
Visualize your AVQT scores
* HTML based (no dependencies, viewed in safari)
* Easy to generate (just add the `--visualize` flag)
* Identify issues and share reports with others

## Demo
Frame and segment scores over time.  Select a specific interval and zoom in.

Bottom track has PSNR.

## Evaluate time window in your videos
Focus on evaluating a specific time window (e.g. a scene)
Enable scoring vicdeos that are not temporally aligned
Command line arguments to specify start and end frame indices for both videos.

clicking on time in quicktime player can show me each frame???

## Evaluate more pixel formats in AVQT
* evaluate raw videos or videos compressed outside the ecosystem
* Extended pixel formats to include
* chroma subscmapling 444 422, 420,411,410
* bit depth 8,10,12,16
* fourcc flags are deprecated
* use bit dpeth and chroma subsampling flags

# Linux support
* evaluate videos on linux servers and in the cloud
* wide range of linux distributions (ubuntu, centos, etc.)
* Easy to deploy and use

* same command line flags and output file formats
* supports all 20 raw video formats
* viewing conditions model will be enabled in the future

# wrap up
* evaluate the perceptual quality of your videos using AVQT
* Visualize the aVQT scores of your video in an interactive report
* use AVQT to focus on specific scenes in your video
* AVQT is now available on linux!


* https://developer.apple.com/forums/tags/wwdc2022-10149
* https://developer.apple.com/forums/create/question?&tag1=327&tag2=416030
* http://developer.apple.com/services-account/download?path=/Developer_Tools/Advanced_Video_Quality_Tool/AdvancedVideoQualityTool.dmg
