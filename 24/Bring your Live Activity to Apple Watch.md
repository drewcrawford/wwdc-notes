Bring Live Activities into the Smart Stack on Apple Watch with iOS 18 and watchOS 11. We'll cover how Live Activities are presented on Apple Watch, as well as how you can enhance their presentation for the Smart Stack. We'll also explore additional considerations to ensure Live Activities on Apple Watch always present up-to-date information.

###  Existing Live Activity views - 1:28
```swift
struct DeliveryLiveActivity: Widget { 
    var body: some WidgetConfiguration { 
        ActivityConfiguration(for: DeliveryActivityAttributes.self) { context in 
            DeliveryActivityContent(context: context)
        } 
        dynamicIsland: { context in 
            DynamicIsland { 
                DynamicIslandExpandedRegion(.leading) { 
                    DeliveryExpandedLeadingView(context: context)
                } 
                DynamicIslandExpandedRegion(.trailing) { 
                    DeliveryExpandedTrailingView(context: context)
                } 
                DynamicIslandExpandedRegion(.bottom) { 
                    DeliveryExpandedBottomView(context: context)
                } 
            } 
            compactLeading: { 
                DeliveryCompactLeading(context: context)
            } 
            compactTrailing: { 
                DeliveryCompactTrailing(context: context)
            } 
            minimal: { 
                DeliveryMinimal(context: context)
            } 
        } 
    } 
}
```

###  Preview Live Activities with Xcode Previews - 3:43
```swift
extension DeliveryActivityAttributes.ContentState { 
    static var shippedOrder: DeliveryActivityAttributes.ContentState { 
        .init(status: .shipped, courierName: "Johnny") 
    } 
    
    static var packedOrder: DeliveryActivityAttributes.ContentState { 
        .init(status: .packed, courierName: "Contacting Courier...") 
    } 
} 

#Preview( "Dynamic Island Compact", as: .dynamicIsland(.compact), using: DeliveryActivityAttributes.preview ) { 
    DeliveryLiveActivity() 
} 
contentStates: { 
    DeliveryActivityAttributes.ContentState.packedOrder 
    DeliveryActivityAttributes.ContentState.shippedOrder 
}
```

###  Add .supplementalActivityFamilies to indicate support for the Smart Stack - 4:15
```swift
struct DeliveryLiveActivity: Widget { 
    var body: some WidgetConfiguration { 
        ActivityConfiguration(for: DeliveryActivityAttributes.self) { context in 
            DeliveryActivityContent(context: context)
        } 
        dynamicIsland: { context in 
            DynamicIsland { 
                DynamicIslandExpandedRegion(.leading) { 
                    DeliveryExpandedLeadingView(context: context)
                } 
                DynamicIslandExpandedRegion(.trailing) { 
                    DeliveryExpandedTrailingView(context: context)
                } 
                DynamicIslandExpandedRegion(.bottom) { 
                    DeliveryExpandedBottomView(context: context)
                } 
            } 
            compactLeading: { 
                DeliveryCompactLeading(context: context)
            } 
            compactTrailing: { 
                DeliveryCompactTrailing(context: context)
            } 
            minimal: { 
                DeliveryMinimal(context: context)
            } 
        } 
        .supplementalActivityFamilies([.small]) 
    } 
}
```

###  Customize view layout for the small activity family - 4:49
```swift
struct DeliveryActivityContent: View { 
    @Environment(\.activityFamily) 
    var activityFamily 
    
    var context: ActivityViewContext<DeliveryActivityAttributes> 
    
    var body: some View { 
        switch activityFamily { 
        case .small: 
            DeliverySmallContent(context: context) 
        case .medium: 
            DeliveryMediumContent(context: context) 
        @unknown default: 
            DeliveryMediumContent(context: context) 
        } 
    } 
}
```

###  Preview customized layouts for the Smart Stack - 5:06
```swift
#Preview("Content", as: .content, using: DeliveryActivityAttributes.preview) { 
    DeliveryLiveActivity() 
} 
contentStates: { 
    DeliveryActivityAttributes.ContentState.packedOrder 
    DeliveryActivityAttributes.ContentState.shippedOrder 
}
```

###  Use isLuminanceReduced to remove bright elements with Always On Display - 8:37
```swift
struct DeliveryGauge: View { 
    @Environment(\.isLuminanceReduced) 
    private var isLuminanceReduced 
    
    var context: ActivityViewContext<DeliveryActivityAttributes> 
    
    var body: some View { 
        Gauge(value: context.state.progressPercent) { 
            GaugeLabel(context: context) 
        } 
        .tint(isLuminanceReduced ? .gaugeDim : .gauge) 
    } 
}
```

###  For Live Activities with a light appearance, use a light preferredColorScheme - 8:57
```swift
struct DeliveryActivityContent: View { 
    @Environment(\.activityFamily) 
    var activityFamily 
    
    var context: ActivityViewContext<DeliveryActivityAttributes> 
    
    var body: some View { 
        switch activityFamily { 
        case .small: 
            DeliverySmallContent(context: context) 
                .preferredColorScheme(.light) 
        case .medium: 
            DeliveryMediumContent(context: context) 
        @unknown default: 
            DeliveryMediumContent(context: context) 
        } 
    } 
}
```

# Resources
* https://developer.apple.com/documentation/ActivityKit/displaying-live-data-with-live-activities
* https://developer.apple.com/documentation/ActivityKit/starting-and-updating-live-activities-with-activitykit-push-notifications
