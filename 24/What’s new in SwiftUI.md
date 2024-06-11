Learn how you can use SwiftUI to build great apps for any Apple platform. Explore a fresh new look and feel for tabs and documents on iPadOS. Improve your window management with new windowing APIs, and gain more control over immersive spaces and volumes in your visionOS apps. We'll also take you through other exciting refinements that help you make expressive charts, customize and layout text, and so much more.

### TabView - 1:38
```swift
import SwiftUI

struct KaraokeTabView: View {
    @State var customization = TabViewCustomization()
    
    var body: some View {
        TabView {
            Tab("Parties", image: "party.popper") {
                PartiesView(parties: Party.all)
            }
            .customizationID("karaoke.tab.parties")
            
            Tab("Planning", image: "pencil.and.list.clipboard") {
                PlanningView()
            }
            .customizationID("karaoke.tab.planning")
            
            Tab("Attendance", image: "person.3") {
                AttendanceView()
            }
            .customizationID("karaoke.tab.attendance")
            
            Tab("Song List", image: "music.note.list") {
                SongListView()
            }
            .customizationID("karaoke.tab.songlist")
        }
        .tabViewStyle(.sidebarAdaptable)
        .tabViewCustomization($customization)
    }
}

struct PartiesView: View {
    var parties: [Party]
    
    var body: some View {
        Text("PartiesView")
    }
}

struct PlanningView: View {
    var body: some View {
        Text("PlanningView")
    }
}

struct AttendanceView: View {
    var body: some View {
        Text("AttendanceView")
    }
}

struct SongListView: View {
    var body: some View {
        Text("SongListView")
    }
}

struct Party {
    static var all: [Party] = []
}

#Preview { KaraokeTabView() }
```

### Presentation sizing - 2:28
```swift
import SwiftUI

struct AllPartiesView: View {
    @State var showAddSheet: Bool = true
    var parties: [Party] = []
    
    var body: some View {
        PartiesGridView(parties: parties, showAddSheet: $showAddSheet)
            .sheet(isPresented: $showAddSheet) {
                AddPartyView()
                    .presentationSizing(.form)
            }
    }
}

struct PartiesGridView: View {
    var parties: [Party]
    @Binding var showAddSheet: Bool
    
    var body: some View {
        Text("PartiesGridView")
    }
}

struct AddPartyView: View {
    var body: some View {
        Text("AddPartyView")
    }
}

struct Party {
    static var all: [Party] = []
}

#Preview { AllPartiesView() }
```

### Zoom transition - 2:39
```swift
import SwiftUI

struct PartyView: View {
    var party: Party
    @Namespace() var namespace
    
    var body: some View {
        NavigationLink {
            PartyDetailView(party: party)
                .navigationTransition(.zoom(sourceID: party.id, in: namespace))
        } label: {
            Text("Party!")
        }
        .matchedTransitionSource(id: party.id, in: namespace)
    }
}

struct PartyDetailView: View {
    var party: Party
    
    var body: some View {
        Text("PartyDetailView")
    }
}

struct Party: Identifiable {
    var id = UUID()
    static var all: [Party] = []
}

#Preview {
    @Previewable var party: Party = Party()
    NavigationStack {
        PartyView(party: party)
    }
}
```

### Controls API - 3:18
```swift
import WidgetKit
import SwiftUI

struct StartPartyControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(kind: "com.apple.karaoke_start_party") {
            ControlWidgetButton(action: StartPartyIntent()) {
                Label("Start the Party!", systemImage: "music.mic")
                Text(PartyManager.shared.nextParty.name)
            }
        }
    }
}

class PartyManager {
    static let shared = PartyManager()
    var nextParty: Party = Party(name: "WWDC Karaoke")
}

struct Party {
    var name: String
}

// AppIntent
import AppIntents

struct StartPartyIntent: AppIntent {
    static let title: LocalizedStringResource = "Start the Party"
    
    func perform() async throws -> some IntentResult {
        return .result()
    }
}
```

### Function plotting - 3:49
```swift
import SwiftUI
import Charts

struct AttendanceView: View {
    var body: some View {
        Chart {
            LinePlot(x: "Parties", y: "Guests") { x in
                pow(x, 2)
            }
            .foregroundStyle(.purple)
        }
        .chartXScale(domain: 1...10)
        .chartYScale(domain: 1...100)
    }
}

#Preview { AttendanceView().padding(40) }
```

### Dynamic table columns - 4:18
```swift
import SwiftUI

struct SongCountsTable: View {
    var body: some View {
        Table(Self.guestData) {
            // A static column for the name
            TableColumn("Name", value: \.name)
            
            TableColumnForEach(Self.partyData) { party in
                TableColumn(party.name) { guest in
                    Text(guest.songsSung[party.id] ?? 0, format: .number)
                }
            }
        }
    }
    
    private static func randSongsSung(low: Bool = false) -> [Int : Int] {
        var songs: [Int : Int] = [:]
        for party in partyData {
            songs[party.id] = low ? Int.random(in: 0...3) : Int.random(in: 3...12)
        }
        return songs
    }
    
    private static let guestData: [GuestData] = [
        GuestData(name: "Sommer", songsSung: randSongsSung()),
        GuestData(name: "Sam", songsSung: randSongsSung()),
        GuestData(name: "Max", songsSung: randSongsSung()),
        GuestData(name: "Kyle", songsSung: randSongsSung(low: true)),
        GuestData(name: "Matt", songsSung: randSongsSung(low: true)),
        GuestData(name: "Apollo", songsSung: randSongsSung()),
        GuestData(name: "Anna", songsSung: randSongsSung()),
        GuestData(name: "Raj", songsSung: randSongsSung()),
        GuestData(name: "John", songsSung: randSongsSung(low: true)),
        GuestData(name: "Harry", songsSung: randSongsSung()),
        GuestData(name: "Luca", songsSung: randSongsSung()),
        GuestData(name: "Curt", songsSung: randSongsSung()),
        GuestData(name: "Betsy", songsSung: randSongsSung())
    ]
    
    private static let partyData: [PartyData] = [
        PartyData(partyNumber: 1, numberGuests: 5),
        PartyData(partyNumber: 2, numberGuests: 6),
        PartyData(partyNumber: 3, numberGuests: 7),
        PartyData(partyNumber: 4, numberGuests: 9),
        PartyData(partyNumber: 5, numberGuests: 9),
        PartyData(partyNumber: 6, numberGuests: 10),
        PartyData(partyNumber: 7, numberGuests: 11),
        PartyData(partyNumber: 8, numberGuests: 12),
        PartyData(partyNumber: 9, numberGuests: 11),
        PartyData(partyNumber: 10, numberGuests: 13)
    ]
}

struct GuestData: Identifiable {
    let name: String
    let songsSung: [Int : Int]
    let id = UUID()
}

struct PartyData: Identifiable {
    let partyNumber: Int
    let numberGuests: Int
    let symbolSize = 100
    
    var id: Int {
        partyNumber
    }
    
    var name: String {
        "\(partyNumber)"
    }
}

#Preview { SongCountsTable().padding(40) }
```

### Mesh gradients - 4:42
```swift
import SwiftUI

struct MyMesh: View {
    var body: some View {
        MeshGradient(
            width: 3,
            height: 3,
            points: [
                .init(0, 0),
                .init(0.5, 0),
                .init(1, 0),
                .init(0, 0.5),
                .init(0.3, 0.5),
                .init(1, 0.5),
                .init(0, 1),
                .init(0.5, 1),
                .init(1, 1)
            ],
            colors: [
                .red, .purple, .indigo, .orange, .cyan, .blue, .yellow, .green, .mint
            ]
        )
    }
}

#Preview { MyMesh().statusBarHidden() }
```

### Document launch scene - 5:14
```swift
DocumentGroupLaunchScene("Your Lyrics") {
    NewDocumentButton()
    
    Button("New Parody from Existing Song") {
        // Do something!
    }
}
background: {
    PinkPurpleGradient()
}
backgroundAccessoryView: { geometry in
    MusicNotesAccessoryView(geometry: geometry)
        .symbolEffect(.wiggle(.rotational.continuous()))
}
overlayAccessoryView: { geometry in
    MicrophoneAccessoryView(geometry: geometry)
}
```

### Window styling and default placement - 7:04
```swift
Window("Lyric Preview", id: "lyricPreview") {
    LyricPreview()
}
.windowStyle(.plain)
.windowLevel(.floating)
.defaultWindowPlacement { content, context in
    let displayBounds = context.defaultDisplay.visibleRect
    let contentSize = content.sizeThatFits(.unspecified)
    return topPreviewPlacement(size: contentSize, bounds: displayBounds)
}
```

### Window Drag Gesture - 7:30
```swift
Text(currentLyric)
    .background(.thinMaterial, in: .capsule)
    .gesture(WindowDragGesture())
```

### Push window environment action - 8:18
```swift
struct EditorView: View {
    @Environment(\.pushWindow) private var pushWindow
    
    var body: some View {
        Button("Play", systemImage: "play.fill") {
            pushWindow(id: "lyric-preview")
        }
    }
}
```

### Hover effects - 8:47
```swift
struct ProfileButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .background(.thinMaterial)
            .hoverEffect(.highlight)
            .clipShape(.capsule)
            .hoverEffect { effect, isActive, _ in
                effect.scaleEffect(isActive ? 1.05 : 1.0)
            }
    }
}
```

### Modifier key alternates - 9:14
```swift
Button("Preview Lyrics in Window") {
    // show preview in window
}
.modifierKeyAlternate(.option) {
    Button("Preview Lyrics in Full Screen") {
        // show preview in full screen
    }
}
.keyboardShortcut("p", modifiers: [.shift, .command])
```

### Responding to modifier keys - 9:32
```swift
var body: some View {
    LyricLine()
        .overlay(alignment: .top) {
            if showBouncingBallAlignment {
                // Show bouncing ball alignment guide
            }
        }
        .onModifierKeysChanged(mask: .option) { showBouncingBallAlignment = !$1.isEmpty }
}
```

### Pointer customization - 9:55
```swift
ForEach(resizeAnchors) { anchor in
    ResizeHandle(anchor: anchor)
        .pointerStyle(.frameResize(position: anchor.position))
}
```

### Pencil squeeze gesture - 10:23
```swift
@Environment(\.preferredPencilSqueezeAction) var preferredAction

var body: some View {
    LyricsEditorView()
        .onPencilSqueeze { phase in
            if preferredAction == .showContextualPalette, case let .ended(value) = phase {
                if let anchorPoint = value.hoverPose?.anchor {
                    lyricDoodlePaletteAnchor = .point(anchorPoint)
                }
                lyricDoodlePalettePresented = true
            }
        }
}
```

### Custom containers - 13:13
```swift
struct DisplayBoard<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        DisplayBoardCardLayout {
            ForEach(subviewOf: content) { subview in
                CardView {
                    subview
                }
            }
        }
        .background {
            BoardBackgroundView()
        }
    }
}

DisplayBoard {
    Text("Scrolling in the Deep")
    Text("Born to Build & Run")
    Text("Some Body Like View")
    
    ForEach(songsFromSam) { song in
        Text(song.title)
    }
}
```

### Custom containers with sectioning - 13:35
```swift
DisplayBoard {
    Section("Matt's Favorites") {
        Text("Scrolling in the Deep")
        Text("Born to Build & Run")
        Text("Some Body Like View")
            .displayBoardCardRejected(true)
    }
    
    Section("Sam's Favorites") {
        ForEach(songsFromSam) { song in
            Text(song.title)
        }
    }
}
```

### Entry macro - 13:52
```swift
extension EnvironmentValues {
    @Entry var karaokePartyColor: Color = .purple
}

extension FocusValues {
    @Entry var lyricNote: String? = nil
}

extension Transaction {
    @Entry var animatePartyIcons: Bool = false
}

extension ContainerValues {
    @Entry var displayBoardCardStyle: DisplayBoardCardStyle = .bordered
}
```

### Default accessibility label augmentation - 14:12
```swift
SongView(song)
    .accessibilityElement(children: .combine)
    .accessibilityLabel { label in
        if let rating = song.rating {
            Text(rating)
        }
        label
    }
```

### Previewable - 14:52
```swift
#Preview {
    @Previewable @State var showAllSongs = true
    Toggle("Show All songs", isOn: $showAllSongs)
}
```

### Programatic text selection - 15:06
```swift
struct LyricView: View {
    @State private var selection: TextSelection?
    
    var body: some View {
        TextField("Line \(line.number)", text: $line.text, selection: $selection)
        // ...
    }
}
```

### Getting selected ranges - 15:19
```swift
InspectorContent(text: line.text, ranges: selection?.ranges)
```

### Binding to search field focus state - 15:29
```swift
// Binding to search field focus state
struct SongSearchView: View {
    @FocusState private var isSearchFieldFocused: Bool
    @State private var searchText = ""
    @State private var isPresented = false
    
    var body: some View {
        NavigationSplitView {
            Text("Power Ballads")
            Text("Show Tunes")
        }
        detail: {
            // ...
            if !isSearchFieldFocused {
                Button("Find another song") {
                    isSearchFieldFocused = true
                }
            }
        }
        .searchable(text: $searchText, isPresented: $isPresented)
        .searchFocused($isSearchFieldFocused)
    }
}
```

### Text suggestions - 15:41
```swift
TextField("Line \(line.number)", text: $line.text)
    .textInputSuggestions {
        ForEach(lyricCompletions) { completion in
            Text(completion.attributedCompletion)
                .textInputCompletion(completion.text)
        }
    }
```

### Color mixing - 15:59
```swift
Color.red.mix(with: .purple, by: 0.2)
Color.red.mix(with: .purple, by: 0.5)
Color.red.mix(with: .purple, by: 0.8)
```

### Custom shaders - 16:13
```swift
ContentView()
    .task {
        let slimShader = ShaderLibrary.slim()
        try! await slimShader.compile(as: .layerEffect)
    }
```

### React to scroll geometry changes - 16:23
```swift
struct ContentView: View {
    @State private var showBackButton = false
    
    ScrollView {
        // ...
    }
    .onScrollGeometryChange(for: Bool.self) { geometry in
        geometry.contentOffset.y < geometry.contentInsets.top
    } action: { wasScrolledToTop, isScrolledToTop in
        withAnimation {
            showBackButton = !isScrolledToTop
        }
    }
}
```

### React to scroll visibility changes - 16:42
```swift
struct AutoPlayingVideo: View {
    @State private var player: AVPlayer = makePlayer()
    
    var body: some View {
        VideoPlayer(player: player)
            .onScrollVisibilityChange(threshold: 0.2) { visible in
                if visible {
                    player.play()
                } else {
                    player.pause()
                }
            }
    }
}
```

### New scroll positions - 16:54
```swift
struct ContentView: View {
    @State private var position: ScrollPosition = .init(idType: Int.self)
    
    var body: some View {
        ScrollView {
            // ...
        }
        .scrollPosition($position)
        .overlay {
            FloatingButton("Back to Invitation") {
                position.scrollTo(edge: .top)
            }
        }
    }
}
```

### Gesture interoperability - 18:17
```swift
struct VideoThumbnailScrubGesture: UIGestureRecognizerRepresentable {
    @Binding var progress: Double
    
    func makeUIGestureRecognizer(context: Context) -> VideoThumbnailScrubGestureRecognizer {
        VideoThumbnailScrubGestureRecognizer()
    }
    
    func handleUIGestureRecognizerAction(
        _ recognizer: VideoThumbnailScrubGestureRecognizer,
        context: Context
    ) {
        progress = recognizer.progress
    }
}

struct VideoThumbnailTile: View {
    var body: some View {
        VideoThumbnail()
            .gesture(VideoThumbnailScrubGesture(progress: $progress))
    }
}
```

### SwiftUI animations in UIKit and AppKit - 18:34
```swift
let animation = SwiftUI.Animation.spring(duration: 0.8)

// UIKit
UIView.animate(animation) {
    view.center = endOfBracelet
}

// AppKit
NSAnimationContext.animate(animation) {
    view.center = endOfBracelet
}
```

### Representable animation bridging - 18:57
```swift
struct BeadBoxWrapper: UIViewRepresentable {
    @Binding var isOpen: Bool
    
    func updateUIView(_ box: BeadBox, context: Context) {
        context.animate {
            box.lid.center.y = isOpen ? -100 : 100
        }
    }
}
```

### Volume baseplate visibility - 19:59
```swift
struct KaraokePracticeApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .windowStyle(.volumetric)
        .defaultWorldScaling(.trueScale)
        .volumeBaseplateVisibility(.hidden)
    }
}
```

### React to volume viewpoint changes - 20:15
```swift
struct MicrophoneView: View {
    @State var micRotation: Rotation3D = .identity
    
    var body: some View {
        Model3D(named: "microphone")
            .onVolumeViewpointChange { _, new in
                micRotation = rotateToFace(new)
            }
            .rotation3DEffect(micRotation)
            .animation(.easeInOut, value: micRotation)
    }
}
```

### Control allowed immersion levels - 20:38
```swift
struct KaraokeApp: App {
    @State private var immersion: ImmersionStyle = .progressive(0.4...1.0, initialAmount: 0.5)
    
    var body: some Scene {
        ImmersiveSpace(id: "Karaoke") {
            LoungeView()
        }
        .immersionStyle(selection: $immersion, in: immersion)
    }
}
```

### Preferred surrounding effects - 21:00
```swift
struct LoungeView: View {
    var body: some View {
        StageView()
            .preferredSurroundingsEffect(.colorMultiply(.purple))
    }
}
```

### Custom text renderers - 21:33
```swift
struct KaraokeRenderer: TextRenderer {
    func draw(layout: Text.Layout, in context: inout GraphicsContext) {
        for line in layout {
            for run in line {
                var glow = context glow.addFilter(.blur(radius: 8))
                glow.addFilter(purpleColorFilter)
                glow.draw(run)
                
                context.draw(run)
            }
        }
    }
}

struct LyricsView: View {
    var body: some View {
        Text("A Whole View World")
            .textRenderer(KaraokeRenderer())
    }
}

#Preview { LyricsView() }
```
# Resources
https://developer.apple.com/documentation/Updates/SwiftUI
