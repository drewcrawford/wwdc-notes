#watchOS 

# Accessiblity on watchOS
## Assistive technologies
* VoiceOver
* AssistiveTouch

## Display accommodations
* Reduce motion
* Bold text
* Accessibility large text


# Visual accessibility
* Apps
* complications
* notifications

Don't forget!

Dynamic type.  Notice our font has a fixed font size.

Rather than that, use one of the 11 text styles provided.  These are displayed at the default system text size.  By using a text style, system will automatically adjust the font size.

```swift
struct PlantView: View {
    @Binding var plant: Plant
    
    var body: some View {
        VStack(alignment: .leading) {
            Text(plant.name)
                .font(.title3)
            HStack() {
                PlantImage(plant: plant)
                PlantTaskList(plant: $plant)
            }
            PlantTaskButtons(plant: $plant)
        }
    }
}
```

Set a line limit to the maximum number of lines you support, or remove it.  

Sometimes, layouts for larger text sizes, just need to be structured fiferently.  So use Environment
```swift
struct PlantContainerView: View {
    @Environment(\.sizeCategory) var sizeCategory
    @Binding var plant: Plant
    
    var body: some View {
        if sizeCategory < .extraExtraLarge {
            PlantViewHorizontal(plant: $plant)
        } else {
            PlantViewVertical(plant: $plant)
        }
    }
}
```

## Dynamic type
* Use text styles to size your fonts
* Wrap text, don't truncate
* Prefer vertical layouts for larger text styles

[[Building apps with Dynamic Type - 17]]
[[Make your app visually accessible]]

* Reduce the number of elements.  Each cell has 4 labels, four images, and 2 buttons.
* To get to the second plant, I have to navigate through each 
* Images separate elements from text
* Labels don't make sense
* Two buttons at the bottom are using the default labels

```swift
struct PlantCellView: View {
    @EnvironmentObject var plantData: PlantData
    var plant: Plant
    
    var plantIndex: Int {
        plantData.plants.firstIndex(where: { $0.id == plant.id })!
    }
    
    var body: some View {
        NavigationLink(destination: PlantEditView(plant: plant).environmentObject(plantData)) {
            PlantContainerView(plant: $plantData.plants[plantIndex])
                .padding()
        }
    }
}
```

```swift
struct PlantTaskLabel: View {
    let task: PlantTask
    @Binding var plant: Plant

    var body: some View {
        HStack {
            Image(systemName: task.systemImageName)
                .imageScale(.small)
            Text(plant.stringForTask(task: task))
                .accessibilityLabel(plant.accessibilityStringForTask(task: task))
        }
        .lineLimit(3)
        .font(.caption2)
    }
}
```

```swift
struct PlantButton: View {
    let task: PlantTask
    let action: () -> Void
    @State private var isTapped: Bool = false
    
    var body: some View {
        Button(action: {
            self.isTapped.toggle()
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                self.isTapped.toggle()
            }
            action()
        }) {
            Image(systemName: task.systemImageFillName)
                .foregroundColor(task.color)
                .scaleEffect(isTapped ? 1.5 : 1)
                .animation(nil, value: 0)
                .rotationEffect(.degrees(isTapped ? 360 : 0))
                .animation(.spring(), value: 0)
                .imageScale(.large)
        }
        .buttonStyle(BorderedButtonStyle())
        .accessibilityLabel("Log \(task.name)")
    }
}
```

With SwiftUI most accessibility comes for free.

Not the ideal experience.  `.accessiblityElement(.ignore)`.  

`.accessibilibyAdjustableAction` to allow incrementing or decrementing the value.
`.accessibilityLabel` with a single label.

```swift
struct PlantTaskFrequency: View {
    let task: PlantTask
    @Binding var plant: Plant
    let increment: () -> Void
    let decrement: () -> Void
    
    var value: Int {
        switch task {
        case .water:
            return plant.wateringFrequency
        case .fertilize:
            return plant.fertilizingFrequency
        default:
            return 0
        }
    }
    
    var body: some View {
        Section(header: Text("\(task.name) frequency in days"), content: {
            CustomCounter(value: value, increment: increment, decrement: decrement)
                .accessibilityElement()
                .accessibilityAdjustableAction { direction in
                    switch direction {
                    case .increment:
                        increment()
                    case .decrement:
                        decrement()
                    default:
                        break
                    }
                }
                .accessibilityLabel("\(task.name) frequency")
                .accessibilityValue("\(value) days")
        })
    }
}
```

As we saw,s wiftui makes it easy to make accessible.
Available on all apple platforms
[[Accessibility in SwiftUI - 19]]
[[SwiftUI Accessibility Beyond the Basics]]

## Complications and notifications
A high-traffic window into your app.  Also need to provide notification in an accessible way.

3 components:
* Text => accessiblity label should have the non-abbreviated version.
* image => Provide accessibility labels here too.
* symbol => May come with default labels, but make sure the label is the one that makes the most sense.

### Notifications
* Complex views
* Same accessibility support

# Motor accessibility

Before we dive in, let me give you a quick glance of new features.

## AssistiveTouch
* Full use of apple watch without touch
* Controlled by hand gestures or hand motions
* Access to functionality based on screen content

Primary way to use Assistivetouch is through hand gestures
* Clench
	* Tap
* Double-clench
	* Action menu
* Pinch
	* Next element
* Double-pinch
	* Previous element

### Motion pointer

alternative 
* Leverage wrist motion
* Onscreen pointer
* Dwell control (rest pointer for a long time)

## How it works
* Cursor
* Action menu
	* Performs system or custom actions

* Focusable elements
* Cursor frame
* Action menu

Focusable: built-in controls
Not focusable: static elements
Focusable: actionable elements
Not focusable: disabled user interaction elements

```swift
struct FreeDrinkView: View {
    @State var didCancel = false
    @State var didAccept = false
    @State var showDetail = false
    
    var body: some View {
        VStack(spacing:10) {
            FreeDrinkTitleView()
            
            FreeDrinkInfoView()
                .accessibilityRespondsToUserInteraction(true)
            
            HStack {
                CancelButton(buttonTapped: $didCancel)
                AcceptButton(buttonTapped: $didAccept)
            }
        }
        .onTapGesture {
            showDetail.toggle()
        }
        .sheet(isPresented: $showDetail, onDismiss: dismiss) {
            DrinkDetailModalView()
        }
    }
}
```

Use `.accessibiltyRespondsToUserInteraction(true)` to ensure your item can be focused.

Cursor frame.  Might clip the content inside.  Can add padding and border shape.

```swift
struct DrinkView: View {
    var currentDrink:DrinkInfo
    
    var body: some View {
        HStack(alignment: .firstTextBaseline) {
            DrinkInfoView(drink:currentDrink)
            
            Spacer()
            
            NavigationLink(destination: EditView()) {
                Image(systemName: "ellipsis")
                    .symbolVariant(.circle)
            }
            .contentShape(Circle().scale(1.5))
        }
    }
}
```

`.contentShape` here increases the size.

Custom actions: prioritized and shown at the beginning of list, so it's more convenient to interact.

# Wrap up
* Dynamic type
* VoiceOver
* ASsistiveTouch

* https://developer.apple.com/documentation/watchkit/create_accessible_experiences_for_watchos
* https://developer.apple.com/accessibility/


