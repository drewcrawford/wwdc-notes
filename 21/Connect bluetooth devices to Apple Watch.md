# Bluetooth on watchOS 7
* Foreground
* Background session

Currently, the complication cannot be updated, unless
the person updates your app.

C
# What's new in watchOS 8
Connect to a paired accessory during Background App Refresh
Update complications (from accessory)
Send local notification

Add `bluetooth-central` to `UIBackgroundModes` in Info.plist

|                                    | iOS  | macOS | tvOS    | watchOS |
|------------------------------------|------|-------|---------|---------|
| Background execution               | yes  | yes   | no      | new     |
| Foreground execution               | yes  | yes   | yes     | yes     |
| Minimum connection interval        | 15ms | 15ms  | 30ms    | 30ms    |
| Role                               | both | both  | central | central |
| Connections                        | HW   | HW    | 2       | 2       |
| State preservation and restoration | yes  | no    | no      | no      |

## Advertise and connect
App connects to accessory.  Then BLE connection is attached.
When user exits app, connectionw ill terminate.  Won't be available until next time it has runtime.

Now, apple watch connects during background app refresh.

BAR can happen at any time, so your accessory should advertise at any time.

What if you advertise rarely?  Possible that no advertisement is received during BAR.  BAR is not guaranteed.

One possible strategy is to buffer sensor data, when close to the buffer limit, create an advertisement.

You should advertise at least every 2s in ideal power conditions.  If you expect your device to work in a more challenging configuration, advertise more frequently.

> We highly recommend to only connect to the accessory for the time you need.

What if you don't disconnect during BAR?
CB will terminate the connection.  Your app will get didDisconnect at next BAR.  

> We highly recommend to only connect to the accessory for the time you need.

```swift
func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String: Any], rssi RSSI: NSNumber ) {

    // Add to an array of discovered peripherals,
    // then connect to the peripheral.

    central.connect(peripheral, options: nil)

}
```

```swift
func handle(_ backgroundTasks: Set<WKRefreshBackgroundTask>) {
    for task in backgroundTasks {
        if let refreshTask = task as? WKApplicationRefreshBackgroundTask {
            // Insert your code to start background work here.
            central.connect(peripheral, options: nil)
            refreshTask.expirationHandler = {
                // Insert your code to cancel existing work here.
                if let peripheral = self.bluetoothReceiver.connectedPeripheral {
                    self.central.cancelPeripheralConnection(peripheral)
                }
                refreshTask.setTaskCompletedWithSnapshot(false)
            }
        }
    }
}
```

handling disconnects

```swift
// If the app gets woken up to handle a background refresh task, this method will be called
// if a peripheral was disconnected when the app had previously transitioned to the
// background.
func centralManager(_ central: CBCentralManager, didDisconnectPeripheral peripheral: CBPeripheral, error: Error?) {
    connectedPeripheral = nil
    delegate?.didCompleteDisconnection(from: peripheral)
}
```

```swift
// In your WatchKit Extension delegate:

func didCompleteDisconnection(from peripheral: CBPeripheral) {
    if let refreshTask = currentRefreshTask {
        task.setTaskCompletedWithSnapshot(false)
        currentRefreshTask = nil
    }
}
```



# Recommendations
* Foreground (first-time device setup)
	* Scan for new peripherals
	* Initiate bluetooth connection and pairing
* Background app refresh
	* Do not scan for new peripherals
	* Connect to a paired peripheral
	* Disconnect as soon as possible
	* Automatically disconnected when execution stops
	* 