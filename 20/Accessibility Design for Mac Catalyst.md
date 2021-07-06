Accessibility is one of our core values.  We have an array of technologies.

Ensuring the great work you've done on iOS are converted to Catalyst.  So accessibility will "just work".

# Keyboard usage
On macOS, keyboard isn't a supplementary method, it's primary
Make as much of your app accessible with the keyboard
## focus
Keyboard focus determines which UIElement is receiving events from the keyboard.  

System settings -> keyboard -> shortcuts -> "Use keyboard navigation to move focus between controls".

```swift
//ensures selection automatically triggers when focus moves to a different cell
myTableView.selectionFollowsFocus = true
```

[[20/What's new in Mac catalyst]]

## shortcuts
For people who use assistive technologies, finding onscreen UIs can become tedioius.  Would be nice to perform actions through a keyboard shorcut.

See apple guidelines at https://developer.apple.com/design/human-interface-guidelines/macos/user-interaction/keyboard

Check out existing apps like Safari and copy their shortcuts.

```swift
//creating a keyboard shortcut
extension AppDelegate {
  override func buildMenu(with builder: UIMenuBuilder) {
    super.buildMenu(with: builder)
    let shareCommand = UIKeyCommand(title: NSLocalizedString("Share", comment: ""),
                                    action: #selector(Self.handleShareMenuAction),
                                    input: "I",
                                    modifierFlags: [.command])
    let shareMenu = UIMenu(title: "",
                           identifier: UIMenu.Identifier("com.example.apple-samplecode.RoastedBeans.share"),
                           options: .displayInline,
                           children: [shareCommand])
    builder.insertChild(shareMenu, atEndOfMenu: .edit)
  }
  
  @objc func handleShareMenuAction() {
  }
}
```

```swift
//responding to raw keycodes
extension MyViewController {
  override func pressesBegan(_ presses: Set<UIPress>, with event: UIPressesEvent?) {
    switch presses.first?.key?.keyCode {
    case .keyboardLeftGUI:
      // Handle command key pressed
    case .keyboardB:
      // Handle B key pressed
    default:
    }
  }
}
```

## recap
* accessible keyboard controls
* keyboard focus
* keyboard shortcuts


# Navigation efficiency
voiceover
## accessibility groups
accessibility tree.  `isAccessibilityElement`.
However a flat hierarchy is difficult to navigate when it has many elements.

Set `accessibilityContainerType`.  This allows users to perform a touch gesture to navigate to the first element in the next container.

On Catalyst, we leverage the same API for a different behavior.  Containers on macOS will behave as an accessibility element, so we select the container itself.  Selecting the container will move to the next element.

Note on Catalyst, accessibility container becomes its own node on the accessibility tree.  Its accessibility elements become their own subtree.  

This closely aligns with the appkit situation.

Note that, because this is the case on catalyst, setting accessibility properties on containers (such as label) is useful/essential.

## what are accessibility containers?

"data tables" are when your container adopts `UIAccessibilityContainerDataTable` protocols.
`list` are  for ordered content, primarily webpages or PDF tables of contents
`landmark` is continer sspecifically for web pages or tvOS
`semanticGroup` is a general type.  Will be spoken the first time users focus on an element within that container (iOS) or focusing on container.  In our case we want to use this type.

Too much grouping can overcomplicate navigation.  Elements in the container are not discoverable until the user interacts with the group.

When elements be grouped?  Semantic.  How would you help someone over the phone?  Maybe you'd talk about different panes in the app.  This helps you identify functional sections.

by default
* toolbars
* navigation bars
* uitableview
* uicollectionview

are semantic groups by default

## navigation efficiency
* add a localized `accessibilityLabel` to accessibility containers

```swift
tableView.accessibilityLabel = NSLocalizedString("Coffee list", comment: "")
```

[[Writing Great Accessibility Labels - 19]]

```swift
//including state on accessibility labels
extension RBListViewController {
    
    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        let data = tableData[indexPath.row]
        let label = NSLocalizedString("Coffee list", comment: "")
        let selectedLabel = NSLocalizedString("%@ selected", comment: "")
        tableView.accessibilityLabel = label + ", " + String(format: selectedLabel, data.coffee.brand)
    }

}
```

```swift
//adopting accessibility containers to improve experience
let stackView = UIStackView()
stackView.axis = .vertical
stackView.translatesAutoresizingMaskIntoConstraints = false

let locationsAvailable = viewModel.locationsAvailable

let titleLabel = UILabel()
titleLabel.font = UIFont.preferredFont(forTextStyle: .body).bold()
titleLabel.text = NSLocalizedString("Availability: ", comment: "")
stackView.addArrangedSubview(titleLabel)

for location in locationsAvailable {
  let label = UILabel()
  label.font = UIFont.preferredFont(forTextStyle: .body)
  label.text = "• " + location
  label.accessibilityLabel = location
  stackView.addArrangedSubview(label)
}

stackView.accessibilityLabel = String(format: NSLocalizedString("Available at %@ locations", comment: ""), String(locationsAvailable.count))
stackView.accessibilityContainerType = .semanticGroup
```

# Testing on macOS
We've improved accessibility inspector for mac catalyst apps.

[[Auditing your Apps for Accessibility - 16]]
[[Accessibility Inspector - 19]]

Gives you a complete guide on how AI can find/fix issues across platforms.

cmd-control-up -> inspector moves up into the container

"automation type" tells you how XCUITest will perceive the item.

# Wrap up
* make controls accessible by keyboard and add meaningful shortcuts
* Group related elements in an accessibility container, adding a concise localized label
* Use accessibility inspector
* 