Focused on feature discovery, the TipKit framework makes it easy to display tips in your app. Now you can group tips so features are discovered in the ideal order, make tips reusable with custom tip identifiers, match the look and feel to your app, and sync tips using CloudKit. Learn how you can use the latest advances in TipKit to help people discover everything your app has to offer.

###  1:43 - Create new tips
```swift
// Create new tips
struct ShowLocationTip: Tip {
    var title: Text {
        Text("Show your location")
    }
    var message: Text? {
        Text("Tap the compass to highlight your current location on the map.")
    }
    var image: Image? {
        Image(systemName: "location.circle")
    }
}
```

###  1:54 - Create new tips
```swift
// Create new tips
struct ShowLocationTip: Tip {
    var title: Text {
        Text("Show your location")
    }
    var message: Text? {
        Text("Tap the compass to highlight your current location on the map.")
    }
    var image: Image? {
        Image(systemName: "location.circle")
    }
}
struct RotateMapTip: Tip {
    var title: Text {
        Text("Reorient the map")
    }
    var message: Text? {
        Text("Tap and hold on the compass to rotate the map back to 0° North.")
    }
    var image: Image? {
        Image(systemName: "hand.tap")
    }
}
```

###  2:09 - Show popover tips
```swift
// Show popover tips
struct MapCompassControl: View {
    let showLocationTip = ShowLocationTip()
    let rotateMapTip = RotateMapTip()
    var body: some View {
        CompassDial()
            .popoverTip(showLocationTip)
            .popoverTip(rotateMapTip)
            .onTapGesture {
                showCurrentLocation()
            }
            .onLongPressGesture(minimumDuration: 0.1) {
                reorientMapHeading()
            }
    }
}
```

###  2:41 - Create a TipGroup
```swift
// Create a TipGroup
struct MapCompassControl: View {
    @State var compassTips: TipGroup(.ordered) {
        ShowLocationTip()
        RotateMapTip()
    }
    var body: some View {
        CompassDial()
            .popoverTip(compassTips.currentTip)
            .onTapGesture {
                showCurrentLocation()
            }
            .onLongPressGesture(minimumDuration: 0.1) {
                reorientMapHeading()
            }
    }
}
```

###  3:15 - Show TipGroup tips on different views
```swift
// Show TipGroup tips on different views
struct MapControlsStack: View {
    @State var compassTips: TipGroup(.ordered) {
        ShowLocationTip()
        RotateMapTip()
    }
    var body: some View {
        VStack {
            ShowLocationButton()
                .popoverTip(compassTips.currentTip as? ShowLocationTip)
            RotateMapButton()
                .popoverTip(compassTips.currentTip as? RotateMapTip)
        }
    }
}
```

###  3:50 - Invalidate tips
```swift
// Invalidate tips
struct MapCompassControl: View {
    @State var compassTips: TipGroup(.ordered) {
        showLocationTip
        rotateMapTip
    }
    var body: some View {
        CompassDial()
            .popoverTip(compassTips.currentTip)
            .onTapGesture {
                showLocationTip.invalidate(reason: .actionPerformed)
                showCurrentLocation()
            }
            .onLongPressGesture(minimumDuration: 0.1) {
                rotateMapTip.invalidate(reason: .actionPerformed)
                reorientMapHeading()
            }
    }
}
```

###  5:37 - Create a tip
```swift
// Create a tip
struct ButlerForkTip: Tip {
    var title: Text {
        Text("Butler Fork is now available")
    }
    var message: Text? {
        Text("To see key trail info, tap Big Cottonwood Canyon on the map.")
    }
    var actions: [Action] {
        Action(title: "Go there now")
    }
    var rules: [Rule] {
        #Rule(Region.bigCottonwoodCanyon.didVisitEvent) { $0.donations.count > 3 }
    }
}
```

###  6:01 - Show a TipView
```swift
// Show a TipView
struct ButlerForkTip: Tip {
    var title: Text {
        Text("Butler Fork is now available")
    }
    var message: Text? {
        Text("To see key trail info, tap Big Cottonwood Canyon on the map.")
    }
    var actions: [Action] {
        Action(title: "Go there now")
    }
    var rules: [Rule] {
        #Rule(Region.bigCottonwoodCanyon.didVisitEvent) { $0.donations.count > 3 }
    }
}

struct TrailList: View {
    var trails: [Trail]
    var body: some View {
        ScrollView {
            let butlerForkTip = ButlerForkTip()
            TipView(butlerForkTip) { _ in
                highlightButlerForkTrail()
            }
            ListSection(title: "Trails", trails: trails)
        }
    }
}
```

###  6:45 - Create a reusable tip
```swift
// Create a reusable tip
struct NewTrailTip: Tip {
    let newTrail: Trail
    var title: Text {
        Text("\(newTrail.name) is now available")
    }
    var message: Text? {
        Text("To see key trail info, tap \(newTrail.region) on the map.")
    }
    var actions: [Action] {
        Action(title: "Go there now")
    }
    var id: String {
        "NewTrailTip-\(newTrail.id)"
    }
    var rules: [Rule] {
        #Rule(newTrail.region.didVisitEvent) { $0.donations.count > 3 }
    }
}
```

###  7:26 - Show a TipView
```swift
// Show a TipView
struct NewTrailTip: Tip {
    let newTrail: Trail
    var title: Text {
        Text("\(newTrail.name) is now available")
    }
    var message: Text? {
        Text("To see key trail info, tap \(newTrail.region) on the map.")
    }
    var actions: [Action] {
        Action(title: "Go there now")
    }
    var id: String {
        "NewTrailTip-\(newTrail.id)"
    }
    var rules: [Rule] {
        #Rule(newTrail.region.didVisitEvent) { $0.donations.count > 3 }
    }
}

struct TrailList: View {
    var trails: [Trail]
    let newTrail: Trail
    var body: some View {
        ScrollView {
            let newTrailTip = NewTrailTip(newTrail: newTrail)
            TipView(newTrailTip) { _ in
                highlightTrail(newTrailTip)
            }
            ListSection(title: "Trails", trails: trails)
        }
    }
}
```

###  8:55 - Create a custom TipViewStyle
```swift
// Create a custom TipViewStyle
struct NewTrailTipViewStyle: TipViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        let tip = configuration.tip as! NewTrailTip
        TrailImage(imageName: tip.newTrail.heroImage)
            .frame(maxHeight: 150)
            .overlay {
                VStack {
                    configuration.title.font(.title)
                    configuration.message.font(.subheadline)
                }
            }
    }
}

extension NewTrailTipViewStyle {
    struct TrailImage: View {
        let imageName: String
        var body: some View {
            Image(imageName)
                .resizable()
                .aspectRatio(contentMode: .fill)
        }
    }
}
```

###  9:20 - Apply a TipViewStyle
```swift
// Apply a TipViewStyle
struct NewTrailTipViewStyle: TipViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        let tip = configuration.tip as! NewTrailTip
        TrailImage(imageName: tip.newTrail.heroImage)
            .frame(maxHeight: 150)
            .overlay {
                VStack {
                    configuration.title.font(.title)
                    configuration.message.font(.subheadline)
                }
            }
    }
}

extension NewTrailTipViewStyle {
    struct TrailImage: View {
        let imageName: String
        var body: some View {
            Image(imageName)
                .resizable()
                .aspectRatio(contentMode: .fill)
        }
    }
}

struct TrailList: View {
    var trails: [Trail]
    let newTrail: Trail
    var body: some View {
        ScrollView {
            let newTrailTip = NewTrailTip(newTrail: newTrail)
            TipView(newTrailTip) { _ in
                highlightTrail(newTrailTip)
            }
            .tipViewStyle(NewTrailTipViewStyle())
            ListSection(title: "Trails", trails: trails)
        }
    }
}
```

###  9:45 - Add the tip's action handler
```swift
// Apply a TipViewStyle
struct NewTrailTipViewStyle: TipViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        let tip = configuration.tip as! NewTrailTip
        let highlightTrailAction = configuration.actions.first!
        TrailImage(imageName: tip.newTrail.heroImage)
            .frame(maxHeight: 150)
            .onTapGesture {
                highlightTrailAction.handler()
            }
            .overlay {
                VStack {
                    configuration.title.font(.title)
                    HStack {
                        configuration.message.font(.subheadline)
                        Spacer()
                        Image(systemName: "chevron.forward.circle")
                            .foregroundStyle(.white)
                    }
                }
            }
    }
}

extension NewTrailTipViewStyle {
    struct TrailImage: View {
        let imageName: String
        var body: some View {
            Image(imageName)
                .resizable()
                .aspectRatio(contentMode: .fill)
        }
    }
}

struct TrailList: View {
    var trails: [Trail]
    let newTrail: Trail
    var body: some View {
        ScrollView {
            let newTrailTip = NewTrailTip(newTrail: newTrail)
            TipView(newTrailTip) { _ in
                highlightTrail(newTrailTip)
            }
            .tipViewStyle(NewTrailTipViewStyle())
            ListSection(title: "Trails", trails: trails)
        }
    }
}
```

###  11:38 - Add CloudKit sync for tips
```swift
// Add CloudKit sync for tips
@main
struct TipKitTrails: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .task {
                    await configureTips()
                }
        }
    }
    
    func configureTips() async {
        do {
            try Tips.configure([
                .cloudKitContainer(.named("iCloud.com.apple.TipKitTrails.tips")),
                .displayFrequency(.weekly)
            ])
        } catch {
            print("Unable to configure tips: \(error)")
        }
    }
}
```
# Resources
* https://developer.apple.com/documentation/TipKit
