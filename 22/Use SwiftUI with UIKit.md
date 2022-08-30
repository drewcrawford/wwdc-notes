#swiftui #uikit 

Health app has many visualizations to hepl peopple understand their heatlh data.  Interested in taking advantage of SwiftUI.

How easy it is to start using SwiftUI in your own UIKit apps.

# UIHostingController
A UIViewController that contains a SwiftUI vier hierarchy.  Anywhere you use a VC.  An easy way to start using SwiftUI.  

Is a VC, which has a UIView in its `view` property.

```swift
// Presenting a UIHostingController

let heartRateView = HeartRateView() // a SwiftUI view
let hostingController = UIHostingController(rootView: heartRateView)

// Present the hosting controller modally
self.present(hostingController, animated: true)
```

```swift
// Embedding a UIHostingController

let heartRateView = HeartRateView() // a SwiftUI view
let hostingController = UIHostingController(rootView: heartRateView)

// Add the hosting controller as a child view controller
self.addChild(hostingController)
self.view.addSubview(hostingController.view)
hostingController.didMove(toParent: self)

// Now position & size the hosting controller’s view as desired…
```


New in IOS 16, UIHostingController lets us update `.preferredContentSize` and view's `.intrinsicContentSize`.  Must enable sizing options property.

```swift
// Presenting UIHostingController as a popover

let heartRateView = HeartRateView() // a SwiftUI view
let hostingController = UIHostingController(rootView: heartRateView)

// Enable automatic preferredContentSize updates on the hosting controller
hostingController.sizingOptions = .preferredContentSize

hostingController.modalPresentationStyle = .popover
self.present(hostingController, animated: true)
```

This ensures that the popover is always sized appropriately for swiftUI content.

# Bridging data
App data.  VCs.  To use SwiftUI, you need
UIHostingController containing swiftUI view.  Populate this view with data that is still owned by existing model layer.  In this section, we're going to focus on how to bridge data across that boundary.

A variety of dataflow primitives to help manage that data.

| type                       | column           | notes                            |
| -------------------------- | ---------------- |  -------------------------------- |
| data owned by SwiftUI view | `@State`         | Owned by swiftui, not uikit apps |
| external                   | Passed arguments |  You responsible for updates.                                 |
[[Data Essentials in SwiftUI]] for data owned by swiftui view

passed arguments pattern:

```swift
// Passing data to SwiftUI with manual UIHostingController updates

struct HeartRateView: View {
    var beatsPerMinute: Int

    var body: some View {
        Text("\(beatsPerMinute) BPM")
    }
}

class HeartRateViewController: UIViewController {
    let hostingController: UIHostingController< HeartRateView >
    var beatsPerMinute: Int {
        didSet { update() }
    }

    func update() {
        hostingController.rootView = HeartRateView(beatsPerMinute: beatsPerMinute)
    }
}
```

simple way to get data in.  But it requires manually updating the root view, any time the data changes.  Let's go through some other swiftui data primitives.

| type                                    | column                                 | notes                            |
| --------------------------------------- | -------------------------------------- | -------------------------------- |
| data owned by SwiftUI view              | `@State`                               | Owned by swiftui, not uikit apps |
| external                                | Passed arguments                       | You are responsible for updates.     |
| external reference with change tracking | `@ObservedObject` `@EnvironmentObject` |                                  |

Environment object - [[Data Essentials in SwiftUI]]

Take a model object and conform it to the protocol.
Store the model as an `@ObservedObject` property wrapper.
View will update when properties change.

observed object pattern:
```swift
// Passing an ObservableObject to automatically update SwiftUI views

class HeartData: ObservableObject {
    @Published var beatsPerMinute: Int

    init(beatsPerMinute: Int) {
       self.beatsPerMinute = beatsPerMinute
    }
}

struct HeartRateView: View {
    @ObservedObject var data: HeartData

    var body: some View {
        Text("\(data.beatsPerMinute) BPM")
    }
}
```

vc:
```swift
// Passing an ObservableObject to automatically update SwiftUI views

class HeartRateViewController: UIViewController {
    let data: HeartData
    let hostingController: UIHostingController<HeartRateView>  

    init(data: HeartData) {
        self.data = data
        let heartRateView = HeartRateView(data: data)
        self.hostingController = UIHostingController(rootView: heartRateView)
    }
}
```

We no longer need to manually update hosting controller when the data changes.  Great way to bridge data from UIKit.
# SwiftUI in cells
* Seamlessly build cells using SwiftUI
* Works in collection and table views
* No extra `UIView` or `UIViewController` needed.
```swift
cell.contentConfiguration = UIHostingConfiguration {
  // Start writing SwiftUI here!
}
```

Let's review cell configuration in UIKit.
* Lightweight description of cell apperance
* Applied to a cell to render
* composable
[[Modern cell configuration]]

In order to render thehosting configuration, we set the `contentConfiguration` to a UIHostingConfiguration instance.
```swift
// Building a custom cell using SwiftUI with UIHostingConfiguration

cell.contentConfiguration = UIHostingConfiguration {
    HeartRateTitleView()
}

struct HeartRateTitleView: View {
    var body: some View {
        HStack {
            Label("Heart Rate", systemImage: "heart.fill")
                .foregroundStyle(.pink)
                .font(.system(.subheadline, weight: .bold))
            Spacer()
            Text(Date(), style: .time)
                .foregroundStyle(.secondary)
                .font(.footnote)
        }
    }
}
```

```swift
// Building a custom cell using SwiftUI with UIHostingConfiguration

cell.contentConfiguration = UIHostingConfiguration {
    VStack(alignment: .leading) {
        HeartRateTitleView()
        Spacer()
        HeartRateBPMView()
    }
}

struct HeartRateBPMView: View {
    var body: some View {
        HStack(alignment: .firstTextBaseline) {
            Text("90")
                .font(.system(.title, weight: .semibold))
            Text("BPM")
                .foregroundStyle(.secondary)
                .font(.system(.subheadline, weight: .bold))
        }
    }
}
```

#swiftcharts 

```swift
// Building a custom cell using SwiftUI with UIHostingConfiguration

cell.contentConfiguration = UIHostingConfiguration {
    VStack(alignment: .leading) {
        HeartRateTitleView()
        Spacer()
        HStack(alignment: .bottom) {
            HeartRateBPMView()
            Spacer()
            Chart(heartRateSamples) { sample in
                LineMark(x: .value("Time", sample.time),
                         y: .value("BPM", sample.beatsPerMinute))
                   .symbol(Circle().strokeBorder(lineWidth: 2))
                   .foregroundStyle(.pink)
            }
        }
    }
}
```

[[Hello Swift Charts]]

SwiftUI content is insets to UIKit cell alyout margins by default.

This ensures that cell content is properly aligned with content of adjacent cells and UI elements.

Sometimes you may want to use `.margins(.horizontal, 16)` or similar.  


```swift
cell.contentConfiguration = UIHostingConfiguration {
    HeartRateBPMView()
}
.margins(.horizontal, 16)
```

```swift
cell.contentConfiguration = UIHostingConfiguration {
   HeartTitleView()
} 
.background(.pink)
```

bakcground vs content

|                     | background                   | content                                   |
| ------------------- | ---------------------------- | ----------------------------------------- |
| position in cell    | behind content & accessories | on top of background, next to accessories |
| margins             | no                           | yes (default)                             |
| affects cell sizing | no                           | yes                                          |

## List separators
* automatically aligned to text by default
* Use `.alignmentGuide` modifier to customize

## Swipe actions
Cell must be inside a collection view list or table view
Make sure to use item identifier, not index path

```swift
cell.contentConfiguration = UIHostingConfiguration {
    MedicalConditionView()
        .swipeActions(edge: .trailing) { … }
}
```

When definint swipe actions, make sure that your buttons perform their actions using a stable identifier
index may change while the cell is visible.

Cell interactons such as tap handling, highlighting, and selection, will still be handled by the collection view or tableview.  If you need to customize for these UIKit updates, create your configurationUpdateHandler

```swift
// Incorporating UIKit cell states

cell.configurationUpdateHandler = { cell, state in
    cell.contentConfiguration = UIHostingConfiguration {
      HStack {
        HealthCategoryView()
            Spacer()
            if state.isSelected {
                Image(systemName: "checkmark")
            }
        }
    }
}
```

this runs again any time the cell state changes.

# Data flow for cells
Our goal is to build this list of medical conditions.  

* MedicalCondition model objects
* CollectionView
	* cells
		* UIHostingConfiguration
			* swiftUI view
* Diffable Data Source
	* Snapshot
		* identifiers of model objects
		* unique identifier, not medical condition objects themselves
			* diffable datasource can accurately track identity of each item.

changes:
1.  Collection itself changes.  ex, item is inserted, reordered, or deleted.  These changes are handled by applying a new snapshot to the diffableDataSource.  
	2. Inserted, moved, deleted
	3. Don't affect cells themselves.
4. Properties of individual model objects change.
	5. Require updating the view in existing cells
	6. Diffable DS only contains item identifiers, it doesn't know when the properties of existing items change
	7. Traditionally when using UIKit you would need to manually tell the diffable datasource about these changes by reconfiguring or reloading the items in the snapshot.  This isn't necessary in swiftUI
	8. Store the object model in an `@ObservedObject` property in swiftui view
		9. changes automatically in swiftui
		10. direct connection
		11. do not go through diffabledatasource or uicollectionview
		12. do not pass go

sizing.  Suppose our cells change size based on their model.  But if the cells figure themselves out via SwiftUI, without going through the uicollectionview, how does the collectionview find out about the height change?

UIHostingConfiguration takes advantage of a brand new feature in UIKit.

**Self-resizing cells**
* enabled by default for collection and table views
* cells automatically resized when their content changes
[[22/What's new in UIKit]]

Evidently, we used to have self *sizing* cells, but now they are self **re**sizing cells.  (???)

Sending data out from a swiftUI view back to other parts of your app.  You can createa  two-way binding through a published property of an `ObservableObject`.  Data flwos from object to swiftUI, but swiftUI can write back changes to the property on the model object.

```swift
// Creating a two-way binding to data in SwiftUI

//populates diffable datasource
class MedicalCondition: Identifiable, ObservableObject {
    let id: UUID
   
    @Published var text: String
}

struct MedicalConditionView: View {
    @ObservedObject var condition: MedicalCondition

    var body: some View {
        HStack {
			TextField("Condition", text: $condition.text) //swiftUI writes changes back
            Spacer()
        }
    }
}
```

# Recap
two ways to add swiftUI
* UIHostingController
	* When using a UIHostingController, always add the VC together with the view to your app.  Many swiftui features, such as toolbars, keyboard shorcuts, and views that use UIViewControllerRepresentable, require a connection to the VC hierarchy.
	* Never separate a hosting controller's view from the controller itself
* UIHostingConfiguration
	* No VC.
	* Supports the vast majority of swiftui features
	* keep in mind that the swiftui features that depend on UIViewRepresentable can't be hosted in a cell.

Integrates seamlessly into existing apps
* Use UIHostingController as a building block
* Create custom cells using UIHostingConfiguration
* Adopt ObservableObjecct for automatic data flow

Add SwiftuI to your app today.


* https://developer.apple.com/documentation/uikit/uitableview/4001105-selfsizinginvalidation
* https://developer.apple.com/documentation/uikit/uicollectionview/4001100-selfsizinginvalidation
* https://developer.apple.com/documentation/SwiftUI/Managing-Model-Data-in-Your-App
* https://developer.apple.com/documentation/SwiftUI/UIHostingConfiguration
* https://developer.apple.com/documentation/SwiftUI/UIHostingController
* https://developer.apple.com/documentation/uikit/views_and_controls/using_swiftui_with_uikit
* https://developer.apple.com/documentation/SwiftUI/SwiftUI-Views-Displayed-by-Other-UI-Frameworks
* https://developer.apple.com/documentation/uikit/uiviewcontroller