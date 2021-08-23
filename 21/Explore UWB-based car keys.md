# Car keys
* Unlock, lock, and start
* Share with family and friends
* Manage keys remotely
* Secure and private

[[Introducing Car Keys]]

# Hardware
* U1
* Secure element
* BLE

Car connectivity consortium standardization

# Secure and private
* Session-based keys
* Link layer encryption
* UWB and BLE identifiers randomized
* Secure ranging





# Passive entry
## Secure ranging
Triggered when a device is exiting or exiting the zone.

Wider zone: welcome features
Narrow zone unlocks the door
Lock zone locks the car when you exit

We talk to multiple UWB and BLE points.  

Car maps the trajectory of the device.  Based on location etc., car can trigger welcome features or adjust seats.  Car can initiate unlock operations.  With precise knowledge of whether the user's device is inside or outside of the car, unlocks when a valid device is detected.

Works even with low battery.  Still enough power to get you back on the road.
# Remote keyless entry controls
* Lock/unlock, pre-heat, open the trunk
* Access to car state
* Based on BLE
* Car connectivity consortium standard

To trigger a command, device can do a chellenge.  Challenge, response, device signature, Car performs the actions.

# Personalized settings
Seat position, seat heating, etc.  

Today's car keys rely on key fob.  Now with car keys, don't have to worry about that.  Precise tracking, strongly tied to users.  Cars can personalize experiences.

Even when multiple users are approaching the car.

# System architecture
Essential that your system has good performance.  Since each car has multiple transceiver, we have to select a suitable one.
* Link budget calculation
* Antenna directivity
* Antenna diversity
* 3D time of flight accuracy

Since even well-designed tranceivers can have issues, need to identify the best possible positions to place tranceivers around the car to provide good coverage.  Also limit the number to keep cost down.

The greater height, the better the range.  Poor orientation can cause gaps in coverage or in unwanted areas.

System RF performance
* Radiation pattern
* Maximum range coverage
* Unlock link margin

System latency
* Fast crypto processor
* Low latency bus system
* Software architecture


# Time synchronization
Try to shrink scanning window.  This reduces power use.

By only scanning during these time intervals, improve performance etc.  

# Tranceiver synchronization
Tranceivers can share timing information.  

# Localization algorithm
* Fast
* accurate

Trajectory mapping
Inside/outside detection
Optimize for each cabin type

# Getting started
* Establish UWB interoperability
* Support owner pairing over BLE
* Secure rnaging session management over BT
* Remote keyless entry actions

# More information
* Participate in Car Connectivit Consortium
* Enroll in the MFI program

* https://developer.apple.com/mfi




