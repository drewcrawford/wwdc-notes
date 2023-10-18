Learn about the latest image formats and video technologies supported in Safari 17. Discover how you can use JPEG XL, AVIF, and HEIC in your websites and experiences and learn how they differ from previous formats. We'll also show you how the Managed Media Source API draws less power than Media Source Extensions (MSE) and explore how you can use it to more efficiently manage streaming video over 5G.
#safari 

# Image formats
* gif
	* 36 years ago
	* does not support a full color palette
	* filesizes quite large
* jpeg
	* progressive loading
	* photograph and other images
	* lossy format
	* smaller filesizes and faster loading times
* png
	* 26 years ago
	* overlaying images on top of each other
	* images with large color, sharp textures like logos, etc.

safari supports an additional 4 modal formats
each have their key advantages.

* webp - safari 14 and big sur.  Modern image format that uses advanced compression to achieve smaller filesizes without sacrificing image quality.
* typically smaller than those earlier image formats, which can help improve website performance and loading times

jpegXL - new algorithm.
images over slow connections, like jpeg.
losslessly convert jpeg to XL, while significantly reducing their size

AVIF.  Uses av1 video codec to achieve high comperssion rates without sacrificing image quality.  well-suited 

HEIC.  Image format that uses HTTP video codec to achieve smaller filesizes.  Since it's not widely supported, you wll only want to use it as an alternative format.

Used by iPhone/iPad to set photos.  So can directly handle photos from iOS.

jpeg/avif/heic - wide color.  

Will likely still need to provide formats for the years to come.  But new formats can make your site quicker to load.  Don't need to choose!

Use srcset to select several images.  Provide wide format for people regardless of device support, without the need for looking at UA string or looking at the browser.

# Adaptive streaming
A fascinating evolution.  Long way since the early versiosn of the web.  Key milestones in the evolution of video presentation on websites.  In the early days of the web, video was not commonly used.  Websites were primarily made out of text and static images.

In early 2000s, flash.
Then we had html5 video.

Mobile: increasing important to display video content on smaller screens.  new techniques to allow websites to adapt to different screen sizes.

HTTP Live Streaming: 2009.  #hls -> adaptive bitrate streaming.  
2-10s in length.  Multiple bitrates, different versions made available via manifest, etc.  In the form of an m3u8 multivariant playlist.  Brilliant job at selecting the best variant.  Simple to use, best solution for the end users.

Media Source Extension.  Low-level toolkit taht allows for adaptive streaming by giving the webpage more control/responsibilities for managing buffer/resolution.

Now the most-used web technology.  Not particularly good at managing buffer levels, timing, network access, media variant selection.  Could not enable this on iPhone due to battery life issues.

MSE transfers more control from the user agent to the application running in the page.  This transfer of control added points of inefficiencies.  We wanted to combine flexibility of MSE with efficiency of HLS.

## managed Media Source
More control over media source and objects has been given over to the browser.  Easier for media website authors to support media playback on constrained capability devices.

* lower power usage
* Better memory handling
* Less buffer management
* Access to 5G connectivity
* you are still in control

check if available

Now, whenever you are referring to media source, using MMS.  Always prefer this over MSE.

If all you care about is apple devices, maybe consider HLS?


# AirPlay with MSE
Great benefits of native HLS was automatic support.  With airplay, you can move the video from your phone to a bigger play device.

Requires a URL you can send, which doesn't exist in MSE.  In the choosing the righ timage format, I showed you how you could add alternative sources.  Video element adds the same thing.

If you provide an HLS stream, we will switch over to that for airplay.  Let the user airplay the video.  If this sounds too complicated, you can use framework such as HLS-js?  That will support MME when available.

* requires airplay source alternative
* Or, `video.dispableRemotePlayback = true`.

# Wrap up
* test in safari
* try out safari technology preview
* send us feedback!

[[What's new in CSS]]
[[Rediscover safari developer features]]

# Resources
* https://github.com/video-dev/hls.js/
* https://developer.apple.com/safari/download/
* https://webkit.org/

