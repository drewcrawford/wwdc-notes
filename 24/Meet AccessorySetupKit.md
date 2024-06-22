

Elevate your accessory setup experience with AccessorySetupKit. Display a beautiful pairing dialog with an image of your Bluetooth or Wi-Fi accessory — no trip to the Settings app required. Discover how to improve privacy by pairing only your app with an accessory. And learn how you can migrate existing accessories so they can be managed by AccessorySetupKit.

### Register event handler - 6:02
```swift
import AccessorySetupKit

// Create a session
var session = ASAccessorySession()

// Activate session with event handler
session.activate(on: DispatchQueue.main, eventHandler: handleSessionEvent(event:))

// Handle event
func handleSessionEvent(event: ASAccessoryEvent) {  
    switch event.eventType {
    case .activated:
        print("Session is activated and ready to use")
        print(session.accessories)
    default:
        print("Received event type \(event.eventType)")
    }
}
```

### Create picker display items - 7:23
```swift
// Create descriptor for pink dice
let pinkDescriptor = ASDiscoveryDescriptor()
pinkDescriptor.bluetoothServiceUUID = pinkUUID
// Create descriptor for blue dice
let blueDescriptor = ASDiscoveryDescriptor()
blueDescriptor.bluetoothServiceUUID = blueUUID

// Create picker display items
let pinkDisplayItem = ASPickerDisplayItem(
    name: "Pink Dice",
    productImage: UIImage(named: "pink")!,
    descriptor: pinkDescriptor
)
let blueDisplayItem = ASPickerDisplayItem(
    name: "Blue Dice",
    productImage: UIImage(named: "blue")!,
    descriptor: blueDescriptor
)
```

### Show picker - 8:10
```swift
// Invoke accessory picker
Button {
    session.showPicker(for: [pinkDisplayItem, blueDisplayItem]) { error in
        if let error {
            // Handle error
        }
    }
} label: {
    Text("Add Dice")
}
```

### Handle events - 9:26
```swift
// Handle event
func handleSessionEvent(event: ASAccessoryEvent) {  
    switch event.eventType {
    case .accessoryAdded:
        let newDice: ASAccessory = event.accessory!
    case .accessoryChanged:
        print("Accessory properties changed")
    case .accessoryRemoved:
        print("Accessory removed from system")
    default:
        print("Received event with type: \(event.eventType)")
    }
}
```

### Communicate with accessory - 10:22
```swift
// Connect to accessory using CoreBluetooth
let central = CBCentralManager(delegate: self, queue: nil)

func centralManagerDidUpdateState(_ central: CBCentralManager) {
    switch central.state {
    case .poweredOn:
       // state will only be updated to poweredOn when you have paired accessories
        let peripheral = central.retrievePeripherals(withIdentifiers: [newDice.bluetoothIdentifier]).first
        central.connect(peripheral)
    default:
        print("Received event type \(event.eventType)")
    }
}

func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
    peripheral.delegate = self
    peripheral.discoverServices(pinkUUID)
}
```

### Migrate existing accessories - 11:58
```swift
// Create migration items
let pinkMigration = ASMigrationDisplayItem(name: "Pink Dice", productImage: UIImage(named: "pink")!, descriptor: pinkDescriptor)
pinkMigration.peripheralIdentifier = pinkPeripheral.identifier

// Present picker with migration items
session.showPicker(for: [pinkMigration]) { error in
    if let error {
        // Handle error
    }
}
```

# Resources
* [AccessorySetupKit](https://developer.apple.com/documentation/AccessorySetupKit)
* [Forum: Privacy & Security](https://developer.apple.com/forums/topics/privacy-and-security?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10203/4/B5954562-4B78-4634-8C6B-7CDC4ED9E8B7/downloads/wwdc2024-10203_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10203/4/B5954562-4B78-4634-8C6B-7CDC4ED9E8B7/downloads/wwdc2024-10203_sd.mp4?dl=1)