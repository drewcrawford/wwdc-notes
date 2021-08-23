#voiceover #accessibility 


*Too much data* may overwhelm

*Too little data* will result in an incomplete experience.

# Accessibility custom content API
class `AXCustomContent`
protocol `AXCustomContentProvider`
Available on all platforms
Deliver content in measured portions
Present content only when needed

"More content available".  

## VO rotor
* Navigate applications efficiently
* Several built-in uses
* *More content rotor*

Twist two fingers on the screen.  Down/up to get next/previous additional content.

Control+option + cmd + /
or capslock + cmd + /


Settings=>voiceover=>verbosity=>More content=>Speak,play sound,change pitch, etc.

On macOS< use "voiceover utility preferences"

```swift
import UIKit
import Accessibility

class DogTableViewCell: UITableViewCell, AXCustomContentProvider {

    var coverImage: UIImageView!
    var name: UILabel!
    var type: UILabel!
    var desc: UILabel!
    var popularity: UILabel!
    var age: UILabel!
    var weight: UILabel!
    var height: UILabel!
    
    override var accessibilityLabel: String? {
        get {
            guard let nameLabel = name.text else { return nil }
            guard let typeLabel = type.text else { return nil }
            return nameLabel + ", " + typeLabel
        }
        set { }
    }

    var accessibilityCustomContent: [AXCustomContent]! {
        get {
            let notes = AXCustomContent(label: "Description", value: desc.text!)
            let popularity = AXCustomContent(label: "Popularity", value: popularity.text!)
            let age = AXCustomContent(label: "Age", value: age.text!)
            let weight = AXCustomContent(label: "Weight", value: weight.text!)
            let height = AXCustomContent(label: "Height" , value: height.text!)
            return [age, popularity, weight, height, notes]
        }
        set { }
    }
}
```

Can set `.importance = .high`.  Content will always be presented when user focuses on item without cycling through additional elements.

New, we support this same capability in swiftui

```swift
struct SampleView: View {
    var body: some View {
        VStack {
            Text(name)
            Text(description)
        }
        .accessibilityElement(children: .combine)
        .accessibilityCustomContent("Description", description, importance: .high)
    }
}
```

```swift
import SwiftUI
import Accessibility

struct DogCell: View {
    var dog: Dog
    var body: some View {
        VStack {
            HStack {
                dog.image
                    .resizable()
                VStack(alignment: .leading) {
                    Text(dog.name)
                        .font(.title)
                    Spacer()
                    Text(dog.type)
                        .font(.body)
                    Spacer()
                    Text(dog.description)
                        .fixedSize(horizontal: false, vertical: true)
                        .font(.subheadline)
                        .foregroundColor(Color(uiColor: UIColor.brown))
                        .accessibilityHidden(true)
                }
                Spacer()
            }
            .padding(.horizontal)
            HStack(alignment: .top) {
                VStack(alignment: .leading) {
                    HStack {
                        Text(dog.popularity)
                        Spacer()
                        Text(dog.age)
                        Spacer()
                        Text(dog.weight)
                        Spacer()
                        Text(dog.height)
                    }
                    .foregroundColor(Color(uiColor: UIColor.darkGray))
                    .accessibilityHidden(true)
                }
                Spacer()
            }
            .padding(.horizontal)
            Divider()
        }
        .accessibilityElement(children: .combine)
        .accessibilityCustomContent("Age", dog.age, importance: .high)
        .accessibilityCustomContent("Popularity", dog.popularity)
        .accessibilityCustomContent("Weight", dog.weight)
        .accessibilityCustomContent("Height", dog.height)
        .accessibilityCustomContent("Description", dog.description)
    }
}
```

```swift
import SwiftUI
import Accessibility

extension AccessibilityCustomContentKey {
    static var age: AccessibilityCustomContentKey {
        AccessibilityCustomContentKey("Age")
    }
}

struct DogCell: View {
    var dog: Dog
    var body: some View {
        VStack {
            HStack {
                dog.image
                    .resizable()
                VStack(alignment: .leading) {
                    Text(dog.name)
                        .font(.title)
                    Spacer()
                    Text(dog.type)
                        .font(.body)
                    Spacer()
                    Text(dog.description)
                        .fixedSize(horizontal: false, vertical: true)
                        .font(.subheadline)
                        .foregroundColor(Color(uiColor: UIColor.brown))
                        .accessibilityHidden(true)
                }
                Spacer()
            }
            .padding(.horizontal)
            HStack(alignment: .top) {
                VStack(alignment: .leading) {
                    HStack {
                        Text(dog.popularity)
                        Spacer()
                        Text(dog.age)
                        Spacer()
                        Text(dog.weight)
                        Spacer()
                        Text(dog.height)
                    }
                    .foregroundColor(Color(uiColor: UIColor.darkGray))
                    .accessibilityHidden(true)
                }
                Spacer()
            }
            .padding(.horizontal)
            Divider()
        }
        .accessibilityElement(children: .combine)
        .accessibilityCustomContent(.age, dog.age, importance: .high)
        .accessibilityCustomContent("Popularity", dog.popularity)
        .accessibilityCustomContent("Weight", dog.weight)
        .accessibilityCustomContent("Height", dog.height)
        .accessibilityCustomContent("Description", dog.description)
    }
}
```

# Wrap up
* identify data-rich parts of your app
* Implement `AXCustomContentProvider`
* Move supplementary information to `accessibiltiyCustomContent`
* Set importance property to `.high` to always present info

* https://developer.apple.com/accessibility/

