Find out how you can take your scroll views to the next level with the latest APIs in SwiftUI. We'll show you how to customize scroll views like never before. Explore the relationship between safe areas and a scroll view's margins, learn how to interact with the content offset of a scroll view, and discover how you can add a bit of flair to your content with scroll transitions.

believe it or not, things don't fit on screen.

building block that lets your content scroll.
* axis
* content
	* clipped
	* placed in the safe area
	* evaluates its content eagerly by default
		* use a lazyvstack if needed


### ScrollView - 0:46
```swift
struct Item: Identifiable {
    var id: Int
}

struct ContentView: View {
    @State var items: [Item] = (0 ..< 25).map { Item(id: $0) }

    var body: some View {
        ScrollView(.vertical) {
            LazyVStack {
                ForEach(items) { item in
                    ItemView(item: item)
                }
            }
        }
    }
}

struct ItemView: View {
    var item: Item

    var body: some View {
        Text(item, format: .number)
            .padding(.vertical)
            .frame(maxWidth: .infinity)
    }
}
```

the eact positionis the content offset.  previously we had scrollviewreader.  This year we have new things.

# Margins and safe area

### Basic Featured Section - 2:29
```swift
struct ContentView: View {
    @State var palettes: [Palette] = [
        .init(id: UUID(), name: "Example One"),
        .init(id: UUID(), name: "Example Two"),
        .init(id: UUID(), name: "Example Three"),
    ]

    var body: some View {
        ScrollView {
            GalleryHeroSection(palettes: palettes)
        }
    }
}

struct Palette: Identifiable {
    var id: UUID
    var name: String
}

struct GalleryHeroSection: View {
    var palettes: [Palette]

    var body: some View {
        GallerySection(edge: .top) {
            GalleryHeroContent(palettes: palettes)
        } label: {
            GalleryHeroHeader(palettes: palettes)
        }
    }
}

struct GallerySection<Content: View, Label: View>: View {
    var edge: Edge? = nil
    @ViewBuilder var content: Content
    @ViewBuilder var label: Label

    var body: some View {
        VStack(alignment: .leading) {
            label
                .font(.title2.bold())
            content
        }
        .padding(.top, halfSpacing)
        .padding(.bottom, sectionSpacing)
        .overlay(alignment: .bottom) {
            if edge != .bottom {
                Divider().padding(.horizontal, hMargin)
            }
        }
    }

    var halfSpacing: CGFloat {
        sectionSpacing / 2.0
    }

    var sectionSpacing: CGFloat {
        20.0
    }

    var hMargin: CGFloat {
        #if os(macOS)
        40.0
        #else
        20.0
        #endif
    }
}

struct GalleryHeroContent: View {
    var palettes: [Palette]

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: hSpacing) {
                ForEach(palettes) { palette in
                    GalleryHeroView(palette: palette)
                }
            }
        }
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroView: View {
    var palette: Palette

    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        colorStack
            .aspectRatio(heroRatio, contentMode: .fit)
            .containerRelativeFrame(
                [.horizontal], count: columns, spacing: hSpacing
            )
            .clipShape(.rect(cornerRadius: 20.0))
    }

    private var columns: Int {
        sizeClass == .compact ? 1 : regularCount
    }

    @ViewBuilder
    private var colorStack: some View {
        let offsetValue = stackPadding
        ZStack {
            Color.red
                .offset(x: offsetValue, y: offsetValue)
            Color.blue
            Color.green
                .offset(x: -offsetValue, y: -offsetValue)
        }
        .padding(stackPadding)
        .background()
    }

    var stackPadding: CGFloat {
        20.0
    }

    var heroRatio: CGFloat {
        16.0 / 9.0
    }

    var regularCount: Int {
        2
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroHeader: View {
    var palettes: [Palette]

    var body: some View {
        Text("Featured")
            .padding(.horizontal, hMargin)
    }

    var hMargin: CGFloat {
        20.0
    }
}
```

safeAreaPadding - instead of padding content, we pad the safe area, this expands content to full width when scrolling.

commonly come from the active device.  Can also come from `safeAreaPadding`, `safeAreaInsets`, etc.  scrollview resolves the safe area from various sources.  

Not posisble to configure different insets for different kidns of content by modifying the safe area.

if you want to do that, use `ontentMargins`.  Insets content margin separately from scrollindicators, or indicators separately from content.

### Featured Section with Margins - 4:00
```swift
struct ContentView: View {
    @State var palettes: [Palette] = [
        .init(id: UUID(), name: "Example One"),
        .init(id: UUID(), name: "Example Two"),
        .init(id: UUID(), name: "Example Three"),
    ]

    var body: some View {
        ScrollView {
            GalleryHeroSection(palettes: palettes)
        }
    }
}

struct Palette: Identifiable {
    var id: UUID
    var name: String
}

struct GalleryHeroSection: View {
    var palettes: [Palette]

    var body: some View {
        GallerySection(edge: .top) {
            GalleryHeroContent(palettes: palettes)
        } label: {
            GalleryHeroHeader(palettes: palettes)
        }
    }
}

struct GallerySection<Content: View, Label: View>: View {
    var edge: Edge? = nil
    @ViewBuilder var content: Content
    @ViewBuilder var label: Label

    var body: some View {
        VStack(alignment: .leading) {
            label
                .font(.title2.bold())
            content
        }
        .padding(.top, halfSpacing)
        .padding(.bottom, sectionSpacing)
        .overlay(alignment: .bottom) {
            if edge != .bottom {
                Divider().padding(.horizontal, hMargin)
            }
        }
    }

    var halfSpacing: CGFloat {
        sectionSpacing / 2.0
    }

    var sectionSpacing: CGFloat {
        20.0
    }

    var hMargin: CGFloat {
        #if os(macOS)
        40.0
        #else
        20.0
        #endif
    }
}

struct GalleryHeroContent: View {
    var palettes: [Palette]

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: hSpacing) {
                ForEach(palettes) { palette in
                    GalleryHeroView(palette: palette)
                }
            }
        }
        .contentMargins(.horizontal, hMargin)
    }

    var hMargin: CGFloat {
        20.0
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroView: View {
    var palette: Palette

    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        colorStack
            .aspectRatio(heroRatio, contentMode: .fit)
            .containerRelativeFrame(
                [.horizontal], count: columns, spacing: hSpacing
            )
            .clipShape(.rect(cornerRadius: 20.0))
    }

    private var columns: Int {
        sizeClass == .compact ? 1 : regularCount
    }

    @ViewBuilder
    private var colorStack: some View {
        let offsetValue = stackPadding
        ZStack {
            Color.red
                .offset(x: offsetValue, y: offsetValue)
            Color.blue
            Color.green
                .offset(x: -offsetValue, y: -offsetValue)
        }
        .padding(stackPadding)
        .background()
    }

    var stackPadding: CGFloat {
        20.0
    }

    var heroRatio: CGFloat {
        16.0 / 9.0
    }

    var regularCount: Int {
        2
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroHeader: View {
    var palettes: [Palette]

    var body: some View {
        Text("Featured")
            .padding(.horizontal, hMargin)
    }

    var hMargin: CGFloat {
        20.0
    }
}
```



# Targets and positions

new, you can change `scrollTargetBehavior` modifier.  Here we specified `.paging`.  Now we swipe one page at a time.

custom deceleration rate, and chooses where to scroll based on contained size of scrollview itself.  

viewAligned - aligns to each view.  So the scrollview needs to know which views it should consider.  scroll targets.  New family of modifiers to specify which views are scroll targets.

`scrollTargetLayout` -> each hero view is a scroll target.  Can also mark individual views as scroll targets.  When using lazy stack, it's important to use the scroll target layout modifier.  Views outside the visible region have not yet been created.  Layout knows about which views will create, to ensure it scrolls to the right place.

can conform your own types to `ScrollTargetBehavior` by implementing `updateTarget`.  calculates where a scroll should end, etc.

### Featured Section + Container Relative Frame - 7:42
```swift
struct ContentView: View {
    @State var palettes: [Palette] = [
        .init(id: UUID(), name: "Example One"),
        .init(id: UUID(), name: "Example Two"),
        .init(id: UUID(), name: "Example Three"),
    ]

    var body: some View {
        ScrollView {
            GalleryHeroSection(palettes: palettes)
        }
    }
}

struct Palette: Identifiable {
    var id: UUID
    var name: String
}

struct GalleryHeroSection: View {
    var palettes: [Palette]

    var body: some View {
        GallerySection(edge: .top) {
            GalleryHeroContent(palettes: palettes)
        } label: {
            GalleryHeroHeader(palettes: palettes)
        }
    }
}

struct GallerySection<Content: View, Label: View>: View {
    var edge: Edge? = nil
    @ViewBuilder var content: Content
    @ViewBuilder var label: Label

    var body: some View {
        VStack(alignment: .leading) {
            label
                .font(.title2.bold())
            content
        }
        .padding(.top, halfSpacing)
        .padding(.bottom, sectionSpacing)
        .overlay(alignment: .bottom) {
            if edge != .bottom {
                Divider().padding(.horizontal, hMargin)
            }
        }
    }

    var halfSpacing: CGFloat {
        sectionSpacing / 2.0
    }

    var sectionSpacing: CGFloat {
        20.0
    }

    var hMargin: CGFloat {
        #if os(macOS)
        40.0
        #else
        20.0
        #endif
    }
}

struct GalleryHeroContent: View {
    var palettes: [Palette]

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: hSpacing) {
                ForEach(palettes) { palette in
                    GalleryHeroView(palette: palette)
                }
            }
        }
        .contentMargins(.horizontal, hMargin)
    }

    var hMargin: CGFloat {
        20.0
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroView: View {
    var palette: Palette

    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        colorStack
            .aspectRatio(heroRatio, contentMode: .fit)
            .containerRelativeFrame(
                [.horizontal], count: columns, spacing: hSpacing
            )
            .clipShape(.rect(cornerRadius: 20.0))
    }

    private var columns: Int {
        sizeClass == .compact ? 1 : regularCount
    }

    @ViewBuilder
    private var colorStack: some View {
        let offsetValue = stackPadding
        ZStack {
            Color.red
                .offset(x: offsetValue, y: offsetValue)
            Color.blue
            Color.green
                .offset(x: -offsetValue, y: -offsetValue)
        }
        .padding(stackPadding)
        .background()
    }

    var stackPadding: CGFloat {
        20.0
    }

    var heroRatio: CGFloat {
        16.0 / 9.0
    }

    var regularCount: Int {
        2
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroHeader: View {
    var palettes: [Palette]

    var body: some View {
        Text("Featured")
            .padding(.horizontal, hMargin)
    }

    var hMargin: CGFloat {
        20.0
    }
}
```

### Featured Section + Scroll Position - 9:46
```swift
struct ContentView: View {
    @State var palettes: [Palette] = [
        .init(id: UUID(), name: "Example One"),
        .init(id: UUID(), name: "Example Two"),
        .init(id: UUID(), name: "Example Three"),
    ]

    var body: some View {
        ScrollView {
            GalleryHeroSection(palettes: palettes)
        }
    }
}

struct Palette: Identifiable {
    var id: UUID
    var name: String
}

struct GalleryHeroSection: View {
    var palettes: [Palette]
    @State var mainID: Palette.ID? = nil

    var body: some View {
        GallerySection(edge: .top) {
            GalleryHeroContent(palettes: palettes, mainID: $mainID)
        } label: {
            GalleryHeroHeader(palettes: palettes, mainID: $mainID)
        }
    }
}

struct GallerySection<Content: View, Label: View>: View {
    var edge: Edge? = nil
    @ViewBuilder var content: Content
    @ViewBuilder var label: Label

    var body: some View {
        VStack(alignment: .leading) {
            label
                .font(.title2.bold())
            content
        }
        .padding(.top, halfSpacing)
        .padding(.bottom, sectionSpacing)
        .overlay(alignment: .bottom) {
            if edge != .bottom {
                Divider().padding(.horizontal, hMargin)
            }
        }
    }

    var halfSpacing: CGFloat {
        sectionSpacing / 2.0
    }

    var sectionSpacing: CGFloat {
        20.0
    }

    var hMargin: CGFloat {
        #if os(macOS)
        40.0
        #else
        20.0
        #endif
    }
}

struct GalleryHeroContent: View {
    var palettes: [Palette]
    @Binding var mainID: Palette.ID?

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: hSpacing) {
                ForEach(palettes) { palette in
                    GalleryHeroView(palette: palette)
                }
            }
            .scrollTargetLayout()
        }
        .contentMargins(.horizontal, hMargin)
        .scrollTargetBehavior(.viewAligned)
        .scrollPosition(id: $mainID)
        .scrollIndicators(.never)
    }

    var hMargin: CGFloat {
        20.0
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroView: View {
    var palette: Palette

    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        colorStack
            .aspectRatio(heroRatio, contentMode: .fit)
            .containerRelativeFrame(
                [.horizontal], count: columns, spacing: hSpacing
            )
            .clipShape(.rect(cornerRadius: 20.0))
    }

    private var columns: Int {
        sizeClass == .compact ? 1 : regularCount
    }

    @ViewBuilder
    private var colorStack: some View {
        let offsetValue = stackPadding
        ZStack {
            Color.red
                .offset(x: offsetValue, y: offsetValue)
            Color.blue
            Color.green
                .offset(x: -offsetValue, y: -offsetValue)
        }
        .padding(stackPadding)
        .background()
    }

    var stackPadding: CGFloat {
        20.0
    }

    var heroRatio: CGFloat {
        16.0 / 9.0
    }

    var regularCount: Int {
        2
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroHeader: View {
    var palettes: [Palette]
    @Binding var mainID: Palette.ID?

    var body: some View {
        VStack(alignment: .leading, spacing: 2.0) {
            Text("Featured")
            Spacer().frame(maxWidth: .infinity)
        }
        .padding(.horizontal, hMargin)
        #if os(macOS)
        .overlay {
            HStack(spacing: 0.0) {
                GalleryPaddle(edge: .leading) {
                    scrollToPreviousID()
                }
                Spacer().frame(maxWidth: .infinity)
                GalleryPaddle(edge: .trailing) {
                    scrollToNextID()
                }
            }
        }
        #endif
    }

    private func scrollToNextID() {
        guard let id = mainID, id != palettes.last?.id,
              let index = palettes.firstIndex(where: { $0.id == id })
        else { return }

        withAnimation {
            mainID = palettes[index + 1].id
        }
    }

    private func scrollToPreviousID() {
        guard let id = mainID, id != palettes.first?.id,
              let index = palettes.firstIndex(where: { $0.id == id })
        else { return }

        withAnimation {
            mainID = palettes[index - 1].id
        }
    }

    var hMargin: CGFloat {
        20.0
    }
}

struct GalleryPaddle: View {
    var edge: HorizontalEdge
    var action: () -> Void

    var body: some View {
        Button {
            action()
        } label: {
            Label(labelText, systemImage: labelIcon)
        }
        .buttonStyle(.paddle)
        .font(nil)
    }

    var labelText: String {
        switch edge {
        case .leading:
            return "Backwards"
        case .trailing:
            return "Forwards"
        }
    }

    var labelIcon: String {
        switch edge {
        case .leading:
            return "chevron.backward"
        case .trailing:
            return "chevron.forward"
        }
    }
}

private struct PaddleButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .imageScale(.large)
            .labelStyle(.iconOnly)
    }
}

extension ButtonStyle where Self == PaddleButtonStyle {
    static var paddle: Self { .init() }
}
```

scrollPosition modifier is a binding that we can read/write to.  Scrollview ill scroll ot the view with that ID, etc.

sometiems, i want to visually alter a view based on where it is within the scrollview.

# Scroll transitions

like a normal transition.  Describes the changes a view should undergo when it appears/disappears.  

scroll transition applies when enters or leaves visible region.  By default, when a view is in the center of the visible region, it's in the identity phase of the scroll transition.

###  12:34 - Featured Section + Scroll Transition
```swift
struct ContentView: View {
    @State var palettes: [Palette] = [
        .init(id: UUID(), name: "Example One"),
        .init(id: UUID(), name: "Example Two"),
        .init(id: UUID(), name: "Example Three"),
    ]

    var body: some View {
        ScrollView {
            GalleryHeroSection(palettes: palettes)
        }
    }
}

struct Palette: Identifiable {
    var id: UUID
    var name: String
}

struct GalleryHeroSection: View {
    var palettes: [Palette]
    @State var mainID: Palette.ID? = nil

    var body: some View {
        GallerySection(edge: .top) {
            GalleryHeroContent(palettes: palettes, mainID: $mainID)
        } label: {
            GalleryHeroHeader(palettes: palettes, mainID: $mainID)
        }
    }
}

struct GallerySection<Content: View, Label: View>: View {
    var edge: Edge? = nil
    @ViewBuilder var content: Content
    @ViewBuilder var label: Label

    var body: some View {
        VStack(alignment: .leading) {
            label
                .font(.title2.bold())
            content
        }
        .padding(.top, halfSpacing)
        .padding(.bottom, sectionSpacing)
        .overlay(alignment: .bottom) {
            if edge != .bottom {
                Divider().padding(.horizontal, hMargin)
            }
        }
    }

    var halfSpacing: CGFloat {
        sectionSpacing / 2.0
    }

    var sectionSpacing: CGFloat {
        20.0
    }

    var hMargin: CGFloat {
        #if os(macOS)
        40.0
        #else
        20.0
        #endif
    }
}

struct GalleryHeroContent: View {
    var palettes: [Palette]
    @Binding var mainID: Palette.ID?

    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack(spacing: hSpacing) {
                ForEach(palettes) { palette in
                    GalleryHeroView(palette: palette)
                }
            }
            .scrollTargetLayout()
        }
        .contentMargins(.horizontal, hMargin)
        .scrollTargetBehavior(.viewAligned)
        .scrollPosition(id: $mainID)
        .scrollIndicators(.never)
    }

    var hMargin: CGFloat {
        20.0
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroView: View {
    var palette: Palette

    @Environment(\.horizontalSizeClass) private var sizeClass

    var body: some View {
        colorStack
            .aspectRatio(heroRatio, contentMode: .fit)
            .containerRelativeFrame(
                [.horizontal], count: columns, spacing: hSpacing
            )
            .clipShape(.rect(cornerRadius: 20.0))
            .scrollTransition(axis: .horizontal) { content, phase in
                content
                    .scaleEffect(
                        x: phase.isIdentity ? 1.0 : 0.80,
                        y: phase.isIdentity ? 1.0 : 0.80)
            }
    }

    private var columns: Int {
        sizeClass == .compact ? 1 : regularCount
    }

    @ViewBuilder
    private var colorStack: some View {
        let offsetValue = stackPadding
        ZStack {
            Color.red
                .offset(x: offsetValue, y: offsetValue)
            Color.blue
            Color.green
                .offset(x: -offsetValue, y: -offsetValue)
        }
        .padding(stackPadding)
        .background()
    }

    var stackPadding: CGFloat {
        20.0
    }

    var heroRatio: CGFloat {
        16.0 / 9.0
    }

    var regularCount: Int {
        2
    }

    var hSpacing: CGFloat {
        10.0
    }
}

struct GalleryHeroHeader: View {
    var palettes: [Palette]
    @Binding var mainID: Palette.ID?

    var body: some View {
        VStack(alignment: .leading, spacing: 2.0) {
            Text("Featured")
            Spacer().frame(maxWidth: .infinity)
        }
        .padding(.horizontal, hMargin)
        #if os(macOS)
        .overlay {
            HStack(spacing: 0.0) {
                GalleryPaddle(edge: .leading) {
                    scrollToPreviousID()
                }
                Spacer().frame(maxWidth: .infinity)
                GalleryPaddle(edge: .trailing) {
                    scrollToNextID()
                }
            }
        }
        #endif
    }

    private func scrollToNextID() {
        guard let id = mainID, id != palettes.last?.id,
              let index = palettes.firstIndex(where: { $0.id == id })
        else { return }

        withAnimation {
            mainID = palettes[index + 1].id
        }
    }

    private func scrollToPreviousID() {
        guard let id = mainID, id != palettes.first?.id,
              let index = palettes.firstIndex(where: { $0.id == id })
        else { return }

        withAnimation {
            mainID = palettes[index - 1].id
        }
    }

    var hMargin: CGFloat {
        20.0
    }
}

struct GalleryPaddle: View {
    var edge: HorizontalEdge
    var action: () -> Void

    var body: some View {
        Button {
            action()
        } label: {
            Label(labelText, systemImage: labelIcon)
        }
        .buttonStyle(.paddle)
        .font(nil)
    }

    var labelText: String {
        switch edge {
        case .leading:
            return "Backwards"
        case .trailing:
            return "Forwards"
        }
    }

    var labelIcon: String {
        switch edge {
        case .leading:
            return "chevron.backward"
        case .trailing:
            return "chevron.forward"
        }
    }
}

private struct PaddleButtonStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .padding()
            .imageScale(.large)
            .labelStyle(.iconOnly)
    }
}

extension ButtonStyle where Self == PaddleButtonStyle {
    static var paddle: Self { .init() }
}
```

functions that are safe to use as functions of layout.

scaleEffect, rotationEFfect, offset, etc.  However, not all view modifiers can be sfaely used inside of a scroll transition.

ex: no font.  Anything that will change the overall content size cannot be used within a scroll transition modifier.

# Wrap up
* contentMargin vs safe areas.
* scrollTargetBehavior
* containerRelativeFrame
* scrollPosition -> change or be notified
* scrollTransition


# Resources
