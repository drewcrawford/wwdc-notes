Bring your app's controls to Control Center, the Lock Screen, and beyond. Learn how you can use WidgetKit to extend your app's controls to the system experience. We'll cover how you can to build a control, tailor its appearance, and make it configurable.

### 3:13 - Add the control to the Widget Bundle
```swift
@main struct ProductivityExtensionBundle: WidgetBundle { 
    var body: some Widget { 
        ChecklistWidget() 
        TaskCounterWidget() 
        TimerToggle() 
    } 
}
```

### 3:29 - Complete the control
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        StaticControlConfiguration( kind: "com.apple.Productivity.TimerToggle" ) { 
            ControlWidgetToggle( "Work Timer", isOn: TimerManager.shared.isRunning, action: ToggleTimerIntent() ) { _ in 
                Image(systemName: "hourglass.bottomhalf.filled") 
            } 
        } 
    } 
}
```

### 4:41 - Specify different symbols when on and off​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration( kind: "com.apple.Productivity.TimerToggle" ) { 
             ControlWidgetToggle( "Work Timer", isOn: TimerManager.shared.isRunning, action: ToggleTimerIntent() ) { isOn in 
                 Image(systemName: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
             }
         }
     }
}
```

### 5:21 - Specify custom value text​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​ and add a custom tint color
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
         StaticControlConfiguration( kind: "com.apple.Productivity.TimerToggle" ) { 
             ControlWidgetToggle( "Work Timer", isOn: TimerManager.shared.isRunning, action: ToggleTimerIntent() ) { isOn in 
                 Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
             } 
             .tint(.purple) 
         } 
     } 
}
```

### 8:14 - Implement timer toggling
```swift
struct ToggleTimerIntent: SetValueIntent, LiveActivityIntent { 
    static let title: LocalizedStringResource = "Productivity Timer" 
    @Parameter(title: "Running") var value: Bool 
    
    // The timer’s running state 
    func perform() throws -> some IntentResult { 
        TimerManager.shared.setTimerRunning(value) 
        return .result() 
    } 
}
```

### 8:54 - Refresh the control from within the app
```swift
func timerManager(_ manager: TimerManager, timerDidChange timer: ProductivityTimer) { 
    ControlCenter.shared.reloadControls( ofKind: "com.apple.Productivity.TimerToggle" ) 
}
```

### 10:03 - Define a Value Provider
```swift
struct TimerValueProvider: ControlValueProvider { 
    func currentValue() async throws -> Bool { 
        try await TimerManager.shared.fetchRunningState() 
    } 
     
    let previewValue: Bool = false 
}
```

### 11:00 - Provide asynchronously fetched state with a Value Provider
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        StaticControlConfiguration( kind: "com.apple.Productivity.TimerToggle", provider: TimerValueProvider() ) { isRunning in 
            ControlWidgetToggle( "Work Timer", isOn: isRunning, action: ToggleTimerIntent() ) { isOn in 
                Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
            } 
            .tint(.purple) 
        } 
    } 
}
```
### 13:06 - Make the Value Provider configurable
```swift
struct ConfigurableTimerValueProvider: AppIntentControlValueProvider { 
    func currentValue(configuration: SelectTimerIntent) async throws -> TimerState { 
        let timer = configuration.timer 
        let isRunning = try await TimerManager.shared.fetchTimerRunning(timer: timer) 
        return TimerState(timer: timer, isRunning: isRunning) 
    } 
    func previewValue(configuration: SelectTimerIntent) -> TimerState { 
        return TimerState(timer: configuration.timer, isRunning: false) 
    } 
}
```

### 13:40 - Make the timer configurable
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        AppIntentControlConfiguration( kind: "com.apple.Productivity.TimerToggle", provider: ConfigurableTimerValueProvider() ) { timerState in 
            ControlWidgetToggle( timerState.timer.name, isOn: timerState.isRunning, action: ToggleTimerIntent(timer: timerState.timer) ) { isOn in 
                Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
            } 
            .tint(.purple) 
        } 
    } 
}
```

### 14:26 - Prompt for user configuration automatically
```swift
struct SomeControl: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        AppIntentControlConfiguration( // ... ) 
            .promptsForUserConfiguration() 
    } 
}
```

### 15:42 - Custom action hint -> hint treated as verb phrase
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        AppIntentControlConfiguration( kind: "com.apple.Productivity.TimerToggle", provider: ConfigurableTimerValueProvider() ) { timerState in 
            ControlWidgetToggle( timerState.timer.name, isOn: timerState.isRunning, action: ToggleTimerIntent(timer: timerState.timer) ) { isOn in 
                Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
                    .controlWidgetActionHint(isOn ? "Start" : "Stop") 
            } 
            .tint(.purple) 
        } 
    } 
}
```

### 16:56 - Specify a display name and add a description
```swift
struct TimerToggle: ControlWidget { 
    var body: some ControlWidgetConfiguration { 
        AppIntentControlConfiguration( kind: "com.apple.Productivity.TimerToggle", provider: ConfigurableTimerValueProvider() ) { timerState in 
            ControlWidgetToggle( timerState.timer.name, isOn: timerState.isRunning, action: ToggleTimerIntent(timer: timerState.timer) ) { isOn in 
                Label(isOn ? "Running" : "Stopped", systemImage: isOn ? "hourglass" : "hourglass.bottomhalf.filled") 
                    .controlWidgetActionHint(isOn ? "Start" : "Stop") 
            } 
            .tint(.purple) 
        } 
        .displayName("Productivity Timer") 
        .description("Start and stop a productivity timer.") 
    } 
}
```
# Resources
* https://developer.apple.com/documentation/WidgetKit/Adding-refinements-and-configuration-to-controls
* https://developer.apple.com/documentation/WidgetKit/Adding-refinements-and-configuration-to-controls
* https://developer.apple.com/documentation/WidgetKit/Creating-controls-to-perform-actions-across-the-system
* https://developer.apple.com/design/human-interface-guidelines/controls
* https://developer.apple.com/documentation/WidgetKit/Updating-controls-locally-and-remotely
