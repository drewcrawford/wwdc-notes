Move into the future with Core Location! Meet the CLLocationUpdate class, designed for modern Swift concurrency, and learn how it simplifies getting location updates. We'll show you how this class works with your apps when they run in the foreground or background and share some best practices.

```swift

for try await update in CLLocationUpdate.liveUpdates() {
    print("My current location : \(update.location)")
}
```


# Resources
* https://developer.apple.com/documentation/corelocation/adopting_live_updates_in_core_location
* https://developer.apple.com/documentation/corelocation
* https://developer.apple.com/documentation/corelocation/monitoring_location_changes_with_core_location
