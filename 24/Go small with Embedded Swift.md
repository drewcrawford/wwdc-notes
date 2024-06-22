Embedded Swift brings the safety and expressivity of Swift to constrained environments. Explore how Embedded Swift runs on a variety of microcontrollers through a demonstration using an off-the-shelf Matter device. Learn how the Embedded Swift subset packs the benefits of Swift into a tiny footprint with no runtime, and discover plenty of resources to start your own Embedded Swift adventure.

### Empty Embedded Swift application - 3:50
```swift
@_cdecl("app_main") func app_main() { 
    print("🏎️ Hello, Embedded Swift!") 
}
```

### Turning on LED to blue color - 6:48
```swift
@_cdecl("app_main") func app_main() { 
    print("🏎️ Hello, Embedded Swift!") 
    var config = led_driver_get_config() 
    let handle = led_driver_init(&config) 
    led_driver_set_hue(handle, 240) // blue 
    led_driver_set_saturation(handle, 100) // 100% 
    led_driver_set_brightness(handle, 80) // 80% 
    led_driver_set_power(handle, true) 
}
```

### Using an LED object - 8:32
```swift
let led = LED() 

@_cdecl("app_main") func app_main() { 
    print("🏎️ Hello, Embedded Swift!") 
    led.color = .red 
    led.brightness = 80 
    
    while true { 
        sleep(1) 
        led.enabled = !led.enabled 
        
        if led.enabled { 
            led.color = .hueSaturation(Int.random(in: 0 ..< 360), 100) 
        } 
    } 
}
```

### Matter application controlling an LED light - 12:44
```swift
let led = LED() 

@_cdecl("app_main") func app_main() { 
    print("🏎️ Hello, Embedded Swift!") 
    
    // (1) create a Matter root node 
    let rootNode = Matter.Node() 
    rootNode.identifyHandler = { 
        print("identify") 
    } 
    
    // (2) create a "light" endpoint, configure it 
    let lightEndpoint = Matter.ExtendedColorLight(node: rootNode) 
    lightEndpoint.configuration = .default 
    lightEndpoint.eventHandler = { event in 
        print("lightEndpoint.eventHandler:") 
        print(event.attribute) 
        print(event.value) 
        
        switch event.attribute { 
            case .onOff: 
                led.enabled = (event.value == 1) 
            case .levelControl: 
                led.brightness = Int(Float(event.value) / 255.0 * 100.0) 
            case .colorControl(.currentHue): 
                let newHue = Int(Float(event.value) / 255.0 * 360.0) 
                led.color = .hueSaturation(newHue, led.color.saturation) 
            case .colorControl(.currentSaturation): 
                let newSaturation = Int(Float(event.value) / 255.0 * 100.0) 
                led.color = .hueSaturation(led.color.hue, newSaturation) 
            case .colorControl(.colorTemperatureMireds): 
                let kelvins = 1_000_000 / event.value 
                led.color = .temperature(kelvins) 
            default: break 
        } 
    } 
    
    // (3) add the endpoint to the node 
    rootNode.addEndpoint(lightEndpoint) 
    
    // (4) provide the node to a Matter application, start the application 
    let app = Matter.Application() 
    app.eventHandler = { event in 
        print(event.type) 
    } 
    app.rootNode = rootNode 
    app.start() 
}
```

### Reflection example - 18:03
```swift
// Reflection needs metadata records 
let mirror = Mirror(reflecting: s) 
mirror.children.forEach { … } 

struct MyStruct { 
    var count: Int 
    var name: String 
}
```

### Unavailable features will produce errors - 18:57
```swift
// Unavailable features will produce errors 
protocol Countable { 
    var count: Int { get } 
} 

func count(countable: any Countable) { 
    print(countable.count) 
}
```

### Prefer generics over “any” types - 19:24
```swift
// Prefer generics over “any” types 
protocol Countable { 
    var count: Int { get } 
} 

func count(countable: some Countable) { 
    print(countable.count) 
}
```
# Resources
* https://github.com/apple/swift-evolution/blob/main/visions/embedded-swift.md
* https://github.com/apple/swift/blob/main/docs/EmbeddedSwift/UserManual.md
* https://github.com/apple/swift-embedded-examples
* https://forums.swift.org/c/development/embedded/107
* https://github.com/apple/swift-matter-examples
* https://github.com/apple/swift-mmio
* https://neovim.io/
