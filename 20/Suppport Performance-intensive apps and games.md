Required device capabilities
Lets you target the capabilities of specific device families
Used by appstore to determine app compatibility with hardware
UIRequiredDeviceCapabilities

`metal` key.  A7 and GPU. #metal 
`arkit` key.  A9 or higher #arkit
New this year: requires performance level of A12 bionic chip
`iphone-ipad-minimum-performance-a12`
iOS 14 and xcode 12

A12 bionic
* 6-core CPU and 4-core GPU
* Second-generation neural engine
* ARKit 3 people occlusion and mobile capture
* Metal GPU family apple 5

Devices:
* iPhone 11, 11 pro
* iPhone se 2nd gen
* iPad mini 5g
* ipad pro 6g

When to enable:
* perf analysis indicates the quality target requires the latest hardware
* App needs the additional processing power of A12 bionic
* aligned with iOS 14

[[Optimize Metal apps and games with GPU counters]]
[[Delivering optimized metal apps and games]]

Changes your app's compatibility on the product page
* prevents users from downloading on unsupported devices
* communicates which devices are supported


