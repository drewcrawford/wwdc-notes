#watchOS 
# Update a complication
Bluetooth LE peripheral.  When background refresh happens, allows your pap to conenct to peripheral

[[Connect bluetooth devices to Apple Watch]]

What if a time-sensitive event happens?  
# Listen for alerts
In watchOS 9, we introduce listening for alerts.

Connect in the foreground.
When app stops running, #corebluetooth maintains connection.  When value changes, your app gets runtime.  Can post notification, etc.  Provide users with time-sensitive information they care about.

1.  Add `bluetooth-central` to `UIBackgroundModes`.  a.k.a. "App communicates using core bluetooth".  SAme as iOS for background execution.
2. Edit plist manually, can't rely on iOS capabilities
```swift
func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {
    peripheral.setNotifyValue(true, for: characteristic)
}

func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {
    if let newData = characteristic.value {
        // Post a local notification.
    }
}
```

## Reconnecting
If you go out of range, may disconnect.  App briefly gets bg runtime in order to attempt a connection.  Same as iOS.

When device is in range, CB reconnects.  

## Limits
If your device is on the edge of range, and repeatedly disconnects.  Range will be reduced.  Only devices close to apple watch will reconnect.  Rolling window of 24 hours, reset whenever the user interacts with your app.

If you need to gather periodical data, do this with background app refresh.  When your app exceeds the limit, `LeGattNearBackgroundNotificationLimit` will be posted.  Consider listening to this error.

If that's important, find another way to communicate with the user, such as ??

After limiti is exceeded, you get `LeGattExceededBackgroundNotificationLimit`.  After this point, your app will no longer receive runtime and revert back to watchOS 8 behavior where there is no background connection and only app refresh.

We recommend using the error to know when the limit is reached rather than counting

* Background runtime opportunities => 5 times.  Reset 24 hours or when user interacts with the app.  Only apply to bluetooth background connections.
* Processing time => very short.  Not enough time to do compelx processing.
* Apple watch series 6 or later
# Discover peripherals
My watchOS app gets timely alerts.  Suppose we have no connection.  watchOS app wills can for unique service UUID.  When medical device detects something serious, it transmits.  Apple watch discovers this peripheral and can notify the user immediately.

1.  Initiate a BLE scan, corebluetooth continues scanning in the background
2. peripheral advertisement
3. app gets background runtime to initiate a connection.

```swift
func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
    central.scanForPeripherals(withServices: [myCustomUUID])
}
```

Same API as iOS 8 but we honor this in the background.  Know that if you ask for `allowDuplicateKeys` it's only available in foreground.

## Limits
* Background runtime
	* combined with previous section limit
* Apple watch series 6 or later
# Design your accessory
How to design your accessory to maek the most of this.

If power consumption is a concern, maybe your item should only advertise when a problem.  Increases discovery time but you save power.

On theo ther hand, if you need low latency but power's not much of a concern, can use background connection.  Limit of 2 bluetooth connections per app.  

* On-device (peripheral) processing/filtering.
	* Send only the relevant events or when temperature changes
	* Both your peripheral and the apple watch user will save on power
* Disconnection behavior
* Advertisement interval
	* battery life, reconnection time, etc.

We recommend a few values.  ex, if you are battery constrained, use a value of 122.5ms.  20ms => detection within a second in idle conditions.

High advertisement rate can only be used while a critical event happens.

Connection interval.  If your deivce maintains connection, we recommend a long interval, >150ms.  Save battery and provide best ux on apple watch.  
Bluetooth 5.3 is coming.  This will allow ot increase ocnnection interval when idle and quickly change when you need lower latency.

|                                    | ios  | macos | tvos    | watchos |
| ---------------------------------- | ---- | ----- | ------- | ------- |
| background execution               | yes  | yes   | no      | yes,new |
| foreground execution               | yes  | yes   | yes     | yes     |
| minimum connection interval        | 15ms | 15ms  | 30ms    | 30ms    |
| role                               | both | both  | central | central |
| connections                        | HW   | HW    | 2       | 2       |
| state preservation and restoration | yes  | no    | no      | no        |






* https://developer.apple.com/accessories/Accessory-Design-Guidelines.pdf
* https://developer.apple.com/documentation/corebluetooth