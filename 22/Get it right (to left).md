#localization 

You've localized your app in various languages.  nwo you want to localize for arabic/hebrew.  Good choice, arabic is one of the 10 most used languages.

Brings new challenges.  That's waht this talk is about.  How to develop your app so it can be localized into these languages

# RTL languages
In hebrew, characters run right to left.  Arabic.  etc.

Apple has keyboard support for 15 RTL languages.

* aligned on right
* line-ending punctuation
* number runs left to right.
* english words left to right.
* Many text is bidirectional.

Entire page is right to left.  
All UI from right to left, including safari's toolbar.
All elements in numbers have also flipped.  Sidebar ont he right, document itself, etc.  Mac menubar, mac dock.

this can be complicated.  We do most of the heavy lifting for you.  There are things to keep in mind.

# Text
## Writing direction
English: left to right
Hebrew: right to left.
Mixed: individual components keep writing direction.

When we say the english sentence has left-to-right writing direction.

English: left-aligned
Hebrew: right-alined

The good news is that most of the time you don't have to worry about either of these things.  Core Text takes care of arranging properly.  
All our UI frameworks automatically set writing direction of alignmenta s well.

* Natural writing direction
	* Follows UI language
* Natural alignment
	* Follows writing direction

Most of the time this is what you want.  But you can override the default.

| UI direction                         | LTR (english,chinese,etc) | RTL (arabic,hebrew,etc) |
| ------------------------------------ | ------------------------- | ----------------------- |
| absolute direction                   | Left -> Right             | Left -> Right           |
| Text alignment and writing direction | natural                   | natural                        |

# Images
Having text read in the opposite direction has a profound influence on things other than text.
Pages toolbar in english and arabic.  Many icons are the same.  Either they're symmetrical or direction isn't tied to the language.

Other buttons, like view/document, flip their mirror images.  e.g. sidebar shows icon on opposite side.  Document shows the user turn the opposite direction.

Icon changes completely.  Letter on text button changes to a different letter to reflect the language.

fix images in asset catalog.  Ask for this in xcode's iamge settings.  Direction=>{Fixed, 2 mirror options, or Both}.  The two mirrors are related to which one is your development language.

SF Symbols.  These flip automatically.  In this case the bulleted list has different RTL And LTR versions.

Localization feature can go beyond just mirroring for RTL.  In arabic, question mark is the reverse.  

Arrows and other directional indicators.  If we look at the two that point to the left, `arrow.backward.circle` flips.  `arrow.left.circle` does not flip.  SFSymbols follows this naming convention for icons you may or may not want to flip.  Forward/backward flips, left/right do not.

| UI direction                         | LTR (english,chinese,etc) | RTL (arabic,hebrew,etc) |
| ------------------------------------ | ------------------------- | ----------------------- |
| absolute direction                   | Left -> Right             | Left -> Right           |
| Text alignment and writing direction | natural                   | natural                 |
| Image direction                      | Backward,Forward          | Forward,Backward                        |

# Control orientation
Notice that everythign has flipped appearance for RTL.  Popup buttons moved to the left.  Checkboxes.  Etc.  Opacity slider.

Buttons with textual label and an icon.  Arrow on the preview button flips.  While arrow on animation direction does not.
```swift
struct ContentView: View {
    var body: some View {
    VStack(alignment: .leading) {
        Button(action: {}) {
            Label("Preview", systemImage: "arrowtriangle.forward.fill")
        }.labelStyle(IconOnRightLabelStyle())
            
            HStack() {
                Button(action: {}) {
                    Label("Left", systemImage: "arrow.left")
                }.labelStyle(TitleAndIconLabelStyle())

                Button(action: {}) {
                    Label("Right", systemImage: "arrow.right")
                }.labelStyle(IconOnRightLabelStyle())
            }.environment(\.layoutDirection, .leftToRight)
        }.padding()
    }
}
```
as we saw before, choose eitehr an icon that reverses or doesn't based on `forward` vs `left`.

Next question: how do you control which sideof the label the icon goes on?  `.labelStyle`.  By default, in reading direction.  

This requires a custom style.  For example

```swift
struct IconOnRightLabelStyle : LabelStyle {
    func makeBody(configuration: Configuration) -> some View {
        HStack {
            configuration.title
            configuration.icon
        }
    }
}
```

Label style has to make an HStack and add the title/icon itno it.  As with any HStack, the order you add is order of display, and it is automatically reverses?

You don't want the icon to change size on the right, you want it on the right regardless.  Views on SwiftUI pick up their directionality from the environment which you modify.  So use `.layoutDirection` `.leftToRight` for the layout.  This will inject the value.

```swift
struct ContentView: View {
    var body: some View {
    VStack(alignment: .leading) {
        Button(action: {}) {
            Label("Preview", systemImage: "arrowtriangle.forward.fill")
        }.labelStyle(IconOnRightLabelStyle())
            
            HStack() {
                Button(action: {}) {
                    Label("Left", systemImage: "arrow.left")
                }.labelStyle(TitleAndIconLabelStyle())

                Button(action: {}) {
                    Label("Right", systemImage: "arrow.right")
                }.labelStyle(IconOnRightLabelStyle())
            }.environment(\.layoutDirection, .leftToRight)
        }.padding()
    }
}
```

Any change you make to the environment are inherited by the child views.  This keeps both buttons from reversing the label.

1.  Left button has its icon on the left beacuse we used the built in TitleAndLabelStyle
2. Preview and right icon appear on the right because of the custom style
3. Left/Right don't change their order because we've added an environment modifier
4. Preview reverses because it has no modifier

In AppKit/UIKit, the situation is different.  Here the position is ocntrolled with the `Position` control in IB.  

Leading/Trailing => change based on UI direction
Left/Right => do not

In AppKit, you use `.imagePosition`, in UIKit it's `configuration.imagePlacement`.  Might need to set configuration first.

| UI direction                         | LTR (english,chinese,etc) | RTL (arabic,hebrew,etc) |
| ------------------------------------ | ------------------------- | ----------------------- |
| absolute direction                   | Left -> Right             | Left -> Right           |
| Text alignment and writing direction | natural                   | natural                 |
| Image direction                      | Backward,Forward          | Forward,Backward        |
| Layout direction                     | Leading Trailing          | Trailing Leading                        |

Most of the time, you want to use these instead of left and right, saving L/R only for things that are tied to an absolute direction.

This particular screenshot has 4 controls.  
* some reverse their segment order
* Other two segmented controls for alignment text, do not reverse their text.  They move thigns in absolute direction.
```swift
struct ContentView: View {
    var body: some View {
        VStack(alignment: .leading) {
            Picker(selection: $textStyle, label: Text("Text Style")) {
                Text("B").tag(TextStyle.bold)
                Text("I").tag(TextStyle.italic)
                Text("U").tag(TextStyle.underline)
                Text("S").tag(TextStyle.strikethrough)
            }.pickerStyle(.segmented)

            Picker(selection: $alignment, label: Text("Alignment")) {
                Image(systemName: "text.alignleft").tag(TextAlignment.left)
                Image(systemName: "text.aligncenter").tag(TextAlignment.center)
                Image(systemName: "text.alignright").tag(TextAlignment.right)
           }.pickerStyle(.segmented)
             .environment(\.layoutDirection, .leftToRight)
        }
    }
}
```

we use the same environment technique as before

In UIKit, you'll see a segmented control "Semantic".  Controls the "semantic content attribute".  Default is unspecified => control flips.
playback => control is a media playback control
spatial => spatial control.  Move things around in space, in absolute direction
force left-to-right
RTL

Doesn't just work for UISegmentedControl.  All have a content attribute.  For any standard UIKit view, the semantic content attribute will determine if it flips.

In AppKit, for all NSControls, the xcode attributes inspector has 2 attributes called layout and mirror.
Layout => whether the control should use LTR or RTL layout.  You don't normally change this, instead use Mirror.
Mirror => Always to flip.

Keep the layuot the same by setting this value to never.

Text.  Set document password dialog.  In arabic, everything is reversed.  But in english, they were right-aligned to be close to text field.  In arabic, they're left-aligned.

Getting this layout in swiftUI on the mac is easy

```swift
var body: some View {
   Form {
        TextField("Password:", text: $password)
        TextField("Verify:", text: $verifyPassword)
        TextField("Password Hint:\n(Recommended)", text: $passwordHint)
            .multilineTextAlignment(.trailing)
    }.padding()
}
```

But in our example, one thing is multiple line.  Two-line label is not aligned.  Bottom labe really is right-aligned, it's just that its bounding box is right-aligned, not the individual lines of text in the bounding box.  To fix this, we use `.multilineTextAlignment`.

For single line text objects, the bounding box tightly encloses the text.

To keep the alignment the same, change the alignment with an evironment modifier.

In UIKit, text is naturally aligned by default, but you can change to an absolute direction.

Alignment segmented control in IB.  Button on the far right with the dotted line gives you "natural" or "leading edge" alignment.  Follows semantic content attribute.  No builtin setting for trailing edge alignment, you have to do that in code.

In appkit, you have the same alignment control.  But you also have Layout and Mirror.  If Mirror is automatically, the system sets layout direction to RTL, the meaning of all the Alignment settings reversed.  So "Left" Alignment becomes Right, and vice versa.

# UI layout
Arranging individual UI widgets on the screen.  Standard UIKit view and vc's respect the uI direction and also allow you to override

* UIStackView
* UICollectionView
* UINavigationBar/UINavigationController
	* changes direction of segue animations, etc.
* UITableView/UITableViewCell
* UITabBar/UITabBarController
* PageViewController
	* swipe gesture meanings, etc.

all these honor their semantic content attribute.

Similar for standard appkit
* NSStackView
* NSGridView
* NSCollectionView
* NSTableView

Standard swiftui views and VCs respect it as well
* HStack
* Form
* LazyHGrid/LazyVGrid
* List
* Menu

If you use AL, AL also reverses things for UI direction.  If you have horizontal constraints, you'll see that they connect to Leading and Trailing.  Different meanings depending on UI direction.  Turn off "Respect Language Direction" to get left/right.

```swift
myView.leadingAnchor.constraint(equalTo: mySuperView.leadingAnchor, constant:16)
```

just remember to use `leading` and `trailing` instead of `left` or `right`, unless you mean to.

## recap
 We do most of the work for you
| UI direction                         | LTR (english,chinese,etc) | RTL (arabic,hebrew,etc) |
| ------------------------------------ | ------------------------- | ----------------------- |
| absolute direction                   | Left -> Right             | Left -> Right           |
| Text alignment and writing direction | natural                   | natural                 |
| Image direction                      | Backward,Forward          | Forward,Backward        |
| Layout direction                     | Leading Trailing          | Trailing Leading                        |

# Numbers in arabic
Probably the first language you'll localize for that have different digits.

"Latin" digits vs Arabic-Indic digits.
Other languages have unique digits.  e.g. Devanagari

**Neither Arabic nor Hindi always use their native digits**.  Arabic depends on the country, e.g. SA vs UAE.  Individual users can also choose preferred digits.

For Hindi, we use latin digits by default, but users can elect to use native digits instead.

```swift
myLabel.string = String(localized: "There are \(peopleInChat) people in this chat.",
                        comment: "Label indicating number of chat participants")

Text("There are \(peopleInChat) people in this chat.",
     comment: "Label indicating number of chat participants")
```

Good news, this handles numbers correctly.  Also works right for text views in swiftui.  String interpolation with properly-localized digits.

Always use `String(localized:)` for user-visible strings.  don't use format, radix, etc.

Beware of static strings in a `localized:` argument.  Translators will give you latin numerals for some reason.  This is correct in a lot of places, but in SA and some other countries, you want local numerals.

```swift
myLabel.string = String(localized: "There are \(peopleInChat) people in this chat.",
                        comment: "Label indicating number of chat participants")

Text("There are \(peopleInChat) people in this chat.",
     comment: "Label indicating number of chat participants")
```

Could have separate localizations, but nobody does that and it would be wasteful.  In both arabic and hindi, the user can choose the digits they use, so you have to choose a localization based on user preferences, etc.

Solution is to have 1 arabic or hindi localization, but substitute the nubmer in `\(3)`.



```swift
myLabel.string = String(localized: "This application supports \(3) file formats.",
                        comment: "Label showing number of supported file formats
                        (number is always 3)")
```

Other elements like +,-,%,$ can change.  Turkish (LTR) has percent on left for some reason.  Keep in mind that if you're using native arabic digits, they use a different percent symbol.  So you really don't want to append % sign yourself.

```swift
myLabel.stringValue = String(localized: "\(percentComplete.formatted(.percent)) complete")
```

If part of a larger string, as in this example, String(localized) will ensure the substitution is surrounded with markup that keeps the writing direction of the forematted number in the surrounded messages from messing each other up.

# Testing in RTL

You don't have to have arabic or hebrew localizations to test your app in right to left.  Scheme editor.  Options.  App language.  At the bottom, choose the RTL pseudolanguage.\

# Summary
* Apple's frameworks handle RTL automatically
* You can opt out where needed
* Not all alnguages use the same characters for numbers
* Use the RTL pseudolanguage in xcode

shouldn't be hard to get thigns right... to left

* https://developer.apple.com/documentation/Xcode/localization
* https://developer.apple.com/localization/
* https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPInternational/Introduction/Introduction.html















