#swiftui  #accessibility 

How to deliver exceptional and accessible apps.

Technologies make sure anyone can use app.

New tools and APIs for swiftUI that make enriching experience easy.

SwiftUI previews.

Curated list of accessibility editors.  Make your views accessible!

Since modifiers don't have visual chages in previews, a new tool allows you to inspect accessibility w/o leaving xcode.

XC13 has a new "accessibility" thing in righthand inspector.  

> Gamechanger.

Closer look at how changes are reflected.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("WWDC 2021")
                .accessibilityAddTraits(.isHeader)

            Text("SwiftUI Accessibility")
            Text("Beyond the Basics")

            Image(systemName: "checkmark.seal.fill")
        }
    }
}
```

Adding `isHeader` trait.

`checkmark.sealed.fill` becomes `verified` in a label.

Important that anyone be able to improve the app.
# Custom controls
Budget planner view.

```swift
struct BudgetSlider: View {
    @Binding var value: Double
    var label: String

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Text(label)
                Text(value.toDollars()).bold()
            }
            SliderShape(value: value)
                .gesture(DragGesture().onChanged(handle))
        }
    }
}

struct SliderShape: View {
    var value: Double

    private struct BackgroundTrack: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .foregroundColor(Color(white: 0.2))
        }
    }

    private struct OverlayTrack: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .foregroundColor(Color(white: 0.95))
        }
    }

    private struct Knob: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .strokeBorder(Color(white: 0.7), lineWidth: 1)
            .shadow(radius: 3)
        }
    }

    var body: some View {
        GeometryReader { geometry in
            ZStack(alignment: .leading) {
                BackgroundTrack(cornerRadius: geometry.size.height / 2)

                OverlayTrack(cornerRadius: geometry.size.height / 2)
                    .frame(
                        width: max(geometry.size.height, geometry.size.width * CGFloat(value) + geometry.size.height / 2),
                        height: geometry.size.height)

                Knob(cornerRadius: geometry.size.height / 2)
                    .frame(
                        width: geometry.size.height,
                        height: geometry.size.height)
                    .offset(x: max(0, geometry.size.width * CGFloat(value) - geometry.size.height / 2), y: 0)
            }
        }
    }
}

extension Double {
    func toDollars() -> String {
        return "$\(Int(self))"
    }
}
```

Shapes make it easy to create stunning, unique views. But not accessible by default.  Neither is my slider.

Why some users are not able to edit budgets.

Confirm it's not accessible by running a preview.

Since standard slider is accessible by default, allow one view to be accessible by another.  `accessibilityRepresentation`.



```swift
struct BudgetSlider: View {
    @Binding var value: Double
    var label: String

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Text(label)
                Text(value.toDollars()).bold()
            }
            SliderShape(value: value)
                .gesture(DragGesture().onChanged(handle))
                .accessibilityRepresentation {
                    Slider(value: $value, in: 0...1) {
                        Text(label)
                    }
                    .accessibilityValue(value.toDollars())
                }
        }
    }
}

struct SliderShape: View {
    var value: Double

    private struct BackgroundTrack: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .foregroundColor(Color(white: 0.2))
        }
    }

    private struct OverlayTrack: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .foregroundColor(Color(white: 0.95))
        }
    }

    private struct Knob: View {
        var cornerRadius: CGFloat
        var body: some View {
            RoundedRectangle(
                cornerRadius: cornerRadius,
                style: .continuous
            )
            .strokeBorder(Color(white: 0.7), lineWidth: 1)
            .shadow(radius: 3)
        }
    }

    var body: some View {
        GeometryReader { geometry in
            ZStack(alignment: .leading) {
                BackgroundTrack(cornerRadius: geometry.size.height / 2)

                OverlayTrack(cornerRadius: geometry.size.height / 2)
                    .frame(
                        width: max(geometry.size.height, geometry.size.width * CGFloat(value) + geometry.size.height / 2),
                        height: geometry.size.height)

                Knob(cornerRadius: geometry.size.height / 2)
                    .frame(
                        width: geometry.size.height,
                        height: geometry.size.height)
                    .offset(x: max(0, geometry.size.width * CGFloat(value) - geometry.size.height / 2), y: 0)
            }
        }
    }
}

extension Double {
    func toDollars() -> String {
        return "$\(Int(self))"
    }
}
```

Just one way to use.  

We decided to use SFSymbols.  Whiel it may have great default labels, may not fit the intended usecase.

```swift
struct NavigationBarView: View {
    var body: some View {
        HStack {
            Text("Wallet Pal")
                .font(.largeTitle)
                .bold()

            Spacer()

            Button("Edit Budgets", action: { ... })
                .buttonStyle(
                    SymbolButtonStyle(
                        systemName: "slider.vertical.3"))
        }
    }
}

struct SymbolButtonStyle: ButtonStyle {
    let systemName: String

    func makeBody(configuration: Configuration) -> some View {
				Image(systemName: systemName)
            .accessibilityRepresentation { configuration.label }
    }
}
```

Despite intiializing the button with the label `Edit Budgets`, the label comes from the SFSymbol!

We want the accessibility to be represented by label.  So use `accessibilityRepresentation` here too

Preserve the label we used to create the button.

Not just the ideal/recommended way to make custom contorls accessible, also opens up new ways.

# Children
Create an accessibility container, wrapping existing elements as children.  But what if you have an element, and want to set its children?

```swift
struct Budget: Identifiable {
    var month: String
    var amount: Double

    var id: String { month }
}

struct BudgetHistoryGraph: View {
    var budgets: [Budget]

    var body: some View {
        GeometryReader { proxy in
            VStack {
                Canvas { ctx, size in
                    let inset: CGFloat = 25
                    let insetSize = CGSize(width: size.width, height: size.height - inset * 2)
                    let width = insetSize.width / CGFloat(budgets.count)
                    let max = budgets.map(\.amount).max() ?? 0
                    for n in budgets.indices {
                        let x = width * CGFloat(n)
                        let height = (CGFloat(budgets[n].amount) / CGFloat(max)) * insetSize.height
                        let y = insetSize.height - height
                        let p = Path(
                            roundedRect: CGRect(
                                x: x + 2.5,
                                y: y + inset,
                                width: width - 5,
                                height: height),
                            cornerRadius: 4)
                        ctx.fill(p, with: .color(Color.green))

                        ctx.draw(Text(budgets[n].amount.toDollars()), at: CGPoint(x: x + width / 2, y: y + inset / 2))

                        ctx.draw(Text(budgets[n].month), at: CGPoint(x: x + width / 2, y: y + height + 1.5*inset))
                    }
                }
                .accessibilityLabel("Budget History Graph")
                .accessibilityChildren {
                    HStack {
                        ForEach(budgets) { budget in
                            Rectangle()
                                .accessibilityLabel(budget.month)
                                .accessibilityValue(budget.amount.toDollars())

                        }
                    }
                }

            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 16)
                .foregroundColor(Color(white: 0.9)))
        .padding(.horizontal)
    }
}
```

[[Add rich graphics to your SwiftUI app]]

Most important takeaway is that canvas draws a shape collection.

Users need to be able to view their budget history.

1.  canvas should have a label
2.  Accessibility container.  Want to provide its children.  Provide `.accessibilityChildren` modifier.

Having a large but consistent frame makes it easier to navigate on iOS.  OK to have larger frame.

If I select HStack, accessibility preview confirms that an eleent is created for each bar in the graph.  Children of the canvas container.

[[Bring accessibility to charts in your app]]

Combine child behavior will merge properties from multiple elements into a new one.

Can now be used to compose accessibility in a generic way.  

With accessibilityRepresentation => no composition can take place
accessiblityChildren => additive.  Can combine into original element.

See sample project.  



# Navigation
Preview shows elements in their sorted order.  As expected, sorted order matches what we saw.

Fix with containers?
```swift
struct User: Identifiable {
    var id: Int
    var name: String
    var photo: String
}

struct FriendCellView: View {
    var user: User

    var body: some View {
        ZStack(alignment: .topLeading) {
            VStack(alignment: .center) {
                Image(user.photo)
                Text(user.name)
            }

            Button("Send Challenge", action: { /* ... */ })
                .buttonStyle(
                    SymbolButtonStyle(
                        systemName: "gamecontroller.fill"))
        }
    }
}
      
struct FriendsView: View {
    var users: [User]

    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack {
                ForEach(users) { user in
                    FriendCellView(user: user)
                         .accessibilityElement(children: .contain)
                        .onTapGesture { /* ... */ }
                }

                AddFriendButton()

                Spacer()
            }
        }
    }
}
  
struct AddFriendButton: View {
    var body: some View {
        Button(action: { /* ... */ }) {
            Circle()
                .foregroundColor(Color(white: 0.9))
                .frame(width: 50, height: 50)
                .overlay(
                    Image(systemName: "plus")
                        .resizable()
                        .foregroundColor(Color(white: 0.5))
                        .padding(15)
                )
        }
        .buttonStyle(PlainButtonStyle())
    }
}

struct SymbolButtonStyle: ButtonStyle {
    let systemName: String

    func makeBody(configuration: Configuration) -> some View {
				Image(systemName: systemName)
            .accessibilityRepresentation { configuration.label }
    }
}
```

Children are navigated before moving to next element.

VO might want tok now the name of a user before sending a challenge.  So maybe sort last?

`accessibilitySortPriority` => order of elements within a container.  Higher priority sorted first, lower priority sorted last.

Single element for each users?   Often ideal to combine views in a foreach.

`.combine` is a super useful child behavior.  Merges properties into a single navigable element.

For when you need a single element but don't want to inherit properties, use `.ignore`
Use `contain` to wrap children in container.


# Rotors
Consider rotors.  


## What are rotors?
Powerfl navigation tool that can be thought of as bookmarks.  System rotors provide the foundation.

Users can navigate through sections using the headings rotor.  Section view automatically adds `isHeader` trait.

`accessibilityAddTraits` to add yourself!

`.contain` added to the containers rotor.

Warnings when budgets near or exceed their limit.  Alerts.

VO users must exclusively navigate warnings.

[[VoiceOver efficiency with Custom Rotors]]

VStacks, HStacks are not containers by dfault.  So add modifier with `.contain` child modifier
Create rotor with `accessibilityRotor`
Declare which alerts to be included.

```swift
struct Alert: Identifiable {
    var id: Int
    var isUnread: Bool
    var isFlagged: Bool
    var subject: String
    var content: String
}

struct AlertsView: View {
    var alerts: [Alert]

    var body: some View {
        VStack {
            ForEach(alerts) { alert in
                AlertCellView(alert: alert)
                    .accessibilityElement(children: .combine)
            }
        }
        .accessibilityElement(children: .contain)
        .accessibilityRotor("Warnings") {
            ForEach(alerts) { alert in
                if alert.isWarning {
                    AccessibilityRotorEntry(alert.title, id: alert.id)
                }
            }
        }
    }
}

struct AlertCell: View {
    var alert: Alert

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                if alert.isUnread {
                    Circle()
                        .foregroundColor(.blue)
                        .frame(width: 10, height: 10)
                }
                if alert.isFlagged {
                    Image(systemName: "exclamationmark.triangle.fill")
                        .foregroundColor(.orange)
                        .frame(width: 10, height: 10)
                }
                Text(alert.subject)
                    .font(.headline)
                    .fontWeight(.semibold)
                Spacer()
                Text("04/30/21")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            Text(alert.content)
                .lineLimit(3)
        }
        .padding(10)
        .background(
            RoundedRectangle(cornerRadius: 8)
                .foregroundColor(Color(white: 0.9))
        )
    }
}
```

[[Demystify SwiftUI]]

Accessibility rotor can scale from simple to complex views.

```swift
struct Alert: Identifiable {
    var id: Int
    var isUnread: Bool
    var isFlagged: Bool
    var subject: String
    var content: String
}

struct AlertsView: View {
    var alerts: [Alert]

    var body: some View {
        VStack {
            ForEach(alerts) { alert in
                AlertCellView(alert: alert)
                    .accessibilityElement(children: .combine)
            }
        }
        .accessibilityElement(children: .contain)
        .accessibilityRotor("Warnings") {
            ForEach(alerts) { alert in
                if alert.isWarning {
                    AccessibilityRotorEntry(alert.title, id: alert.id)
                }
            }
        }
    }
}

struct AlertCell: View {
    var alert: Alert

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                if alert.isUnread {
                    Circle()
                        .foregroundColor(.blue)
                        .frame(width: 10, height: 10)
                }
                if alert.isFlagged {
                    Image(systemName: "exclamationmark.triangle.fill")
                        .foregroundColor(.orange)
                        .frame(width: 10, height: 10)
                }
                Text(alert.subject)
                    .font(.headline)
                    .fontWeight(.semibold)
                Spacer()
                Text("04/30/21")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            Text(alert.content)
                .lineLimit(3)
        }
        .padding(10)
        .background(
            RoundedRectangle(cornerRadius: 8)
                .foregroundColor(Color(white: 0.9))
        )
    }
}
```
```swift
struct Alert: Identifiable {
    var id: Int
    var isUnread: Bool
    var isFlagged: Bool
    var subject: String
    var content: String
}

struct AlertsView: View {
    var alerts: [Alert]
    @Namespace var namespace

    var body: some View {
        VStack {
            ForEach(alerts) { alert in
                VStack {
                    AlertCellView(alert: alert)
                        .accessibilityElement(children: .combine)
                        .accessibilityRotorEntry(id: alert.id, in: namespace)
                    AlertActionsView(alert: alert)
                }
            }
        }
        .accessibilityElement(children: .contain)
        .accessibilityRotor("Warnings") {
            ForEach(alerts) { alert in
                if alert.isWarning {
                    AccessibilityRotorEntry(alert.title, id: alert.id, in: namespace)
                }
            }
        }
    }
}

struct AlertCell: View {
    var alert: Alert

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                if alert.isUnread {
                    Circle()
                        .foregroundColor(.blue)
                        .frame(width: 10, height: 10)
                }
                if alert.isFlagged {
                    Image(systemName: "exclamationmark.triangle.fill")
                        .foregroundColor(.orange)
                        .frame(width: 10, height: 10)
                }
                Text(alert.subject)
                    .font(.headline)
                    .fontWeight(.semibold)
                Spacer()
                Text("04/30/21")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            Text(alert.content)
                .lineLimit(3)
        }
        .padding(10)
        .background(
            RoundedRectangle(cornerRadius: 8)
                .foregroundColor(Color(white: 0.9))
        )
    }
}
```

Here, the VStack the root of our foreach and has the identity.  So to include the alert cellview, need to mark it as  a rotor entry.

`.accessibiltyRotorentry` with a namespace and ID.

Must match ID used to match rotor entry.

Lastly, need to include namespace fore each entry.  Explicit namespace enables rotor API to scale from simple to complex usecases, allowing elements to scale across multiple views.

Can provide text range as well.

# Focus
Accessibility focus.  e.g. VO focus.

Focus can change
* User interaction
* UI changes => previously focused view is no longer onscreen or covered
* Programatic request

Can now request to move focus.  

We want to use this to show alerts for PNs received while app is in foreground.

We can request apps do so with `AccessilbityFocusState`.

```swift
struct Notification: Equatable {
    enum Priority {
        case low, high
    }
    var content: String
    var priority: Priority
}

struct AlertNotificationView<Content: View>: View {
    @ViewBuilder var content: Content
    @Binding var notification: Notification?
    @AccessibilityFocusState var isNotificationFocused: Bool

    var body: some View {
        ZStack(alignment: .top) {
            content

            if let notification = $notification {
                NotificationBanner(notification: notification)
                    .accessibilityFocused($isNotificationFocused)
            }
        }
        .onChange(of: notification) { notification in
            if notification?.priority == .high {
                isNotificationFocused = true
            } else {
                postAccessibilityNotification()
            }
        }
    }

    func postAccessibilityNotification() {
        guard let announcement = notification?.content else {
            return
        }
        #if os(macOS)
        NSAccessibility.post(
            element: NSApp.accessibilityWindow(),
            notification: .announcementRequested,
            userInfo: [.announcement: announcement])
        #else
        UIAccessibility.post(notification: .announcement, argument: announcement)
        #endif
    }
}

struct NotificationBanner: View {
    @Binding var notification: Notification?
    @State var timer: Timer?
    @AccessibilityFocusState var isNotificationFocused: Bool

    var body: some View {
        if let notification = notification {
            Text(notification.content)
                .accessibilityFocused($isNotificationFocused)
                .onAppear { startTimer() }
                .onDisappear { stopTimer() }
        } else {
            EmptyView()
        }
    }

    func startTimer() {
        timer = Timer.scheduledTimer(
            withTimeInterval: 3,
            repeats: true) { _ in
            if !isNotificationFocused {
                notification = nil
            }
        }
    }

    func stopTimer() {
        timer?.invalidate()
    }
}
```


For lower prioritiy notifications, we post a notificaton for accessibility to announce rather than moving focus programmatically, it's too disruptive.

Delay notification dismissal.  VO focus is no longer reset since the view is no longer removed while focused.

Unlimited number of time to digest the content and interact if desired.

Deliver exceptional, accessible apps this year and beyond.  Read and direct assistive technolgoies to create smooth transitions between views.

# Wrap up
* New accessibility preview tool
* Custom controls and complex graphs accessiliby
* Improving navigation with grouping, rotors and focus

https://developer.apple.com/documentation/accessibility/creating_accessible_views
https://developer.apple.com/documentation/swiftui/view-accessibility