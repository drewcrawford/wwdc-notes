

Explore how Quick Look in visionOS can elevate file preview and editing experiences in your app. We'll cover the integration of in-app and windowed Quick Look, as well as a brand-new API that customizes the windowed Quick Look experience in your app. We'll also share the latest enhancements to viewing 3D models within Quick Look.

### Variants USDZ - 12:22
```swift
#usda 1.0
(
	defaultPrim = "iPhone"
)

def Xform "iPhone" (
	variants = {
		string Color = "Black_Titanium"
	}
	prepend variantSets = ["Color"]
)
{
	variantSet "Color" = {
		"Black_Titanium" { }
		"Blue_Titanium" { }
		"Natural_Titanium" { }
		"White_Titanium" { }
 }
}
```

# Resources
* [Forum: Spatial Computing](https://developer.apple.com/forums/topics/spatial-computing?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10105/5/9DD1E3E1-8BCD-498A-9045-F2251FFDF077/downloads/wwdc2024-10105_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10105/5/9DD1E3E1-8BCD-498A-9045-F2251FFDF077/downloads/wwdc2024-10105_sd.mp4?dl=1)