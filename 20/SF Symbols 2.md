# The basics
Seamlessly integrate with SF fonts.
Vertically centered with the cap height.  Still, there are typographic guidelines to consider.

Baseline -> imaginary line upon which the text rests.
But symbols are different.  Can be above or below.  This is intended and it's meant to optically balance the symbols.

scale
* small
* medium (default)
* large

While remaining weight-matched, can use symbol to adjust size.

small -> approx 20% smaller
large -> approx 30% larger

We make some weight compensation which allow us to make the same stroke width

Different scales may be appropriate based on vertical space.

## demo

SF Symbols app.  Updated this year.  Version of the system fonts suitable for design tools.

With symbols, we can make play and shuffle button more distinctive/recognizable.

We can just copy/paste from the symbols app into sketch.  Unclear to me how this works but it appears to work inline with text.

## demo (code)
```swift
// SF Symbols: simple usage and symbol configuration

import UIKit

class MainPlayerViewController: UIViewController {

    @IBOutlet weak var playButton: UIButton!
    @IBOutlet weak var shuffleButton: UIButton!
    @IBOutlet weak var playImageView: UIImageView!
    @IBOutlet weak var shuffleImageView: UIImageView!

    override func viewDidLoad() {
        super.viewDidLoad()
        setupButtons()
    }

    func setupButtons() {
        playImageView.image = UIImage(systemName: "")
        shuffleImageView.image = UIImage(systemName: "")
    }

    @IBAction func playAction(_ sender: Any) {
    }
    
    @IBAction func shuffleAction(_ sender: Any) {
    }
    
}
```
Can copy symbol name with "shift-cmd-c".  *If you copy the symbol itself, it won't work!*

```swift
// do NOT use symbol characters in code

let shuffleImage = UIImage(systemName: "􀊝")






// always use symbol names in code

let shuffleImage = UIImage(systemName: "shuffle")
```

## demo (sketch)
Can adjust symbol size, *not with points*, but instead with Text -> OpenType Features -> Alternative Stylistic Sets -> Square Symbols.

Symbol remains optically centered.

## demo (code)
```swift
// SF Symbols: simple usage and symbol configuration

import UIKit

class MainPlayerViewController: UIViewController {

    @IBOutlet weak var playButton: UIButton!
    @IBOutlet weak var shuffleButton: UIButton!
    @IBOutlet weak var playImageView: UIImageView!
    @IBOutlet weak var shuffleImageView: UIImageView!

    override func viewDidLoad() {
        super.viewDidLoad()
        setupButtons()
    }

    func setupButtons() {
        let buttonConfig = UIImage.SymbolConfiguration(scale: .small)
        playImageView.preferredSymbolConfiguration = buttonConfig
        playImageView.image = UIImage(systemName: "play.fill")
        shuffleImageView.preferredSymbolConfiguration = buttonConfig
        shuffleImageView.image = UIImage(systemName: "shuffle")
    }

    @IBAction func playAction(_ sender: Any) {
    }
    
    @IBAction func shuffleAction(_ sender: Any) {
    }
    
}
```

```swift
// SF Symbols: simple usage and symbol configuration

import UIKit

class MainPlayerViewController: UIViewController {

    @IBOutlet weak var playButton: UIButton!
    @IBOutlet weak var shuffleButton: UIButton!
    @IBOutlet weak var playImageView: UIImageView!
    @IBOutlet weak var shuffleImageView: UIImageView!

    override func viewDidLoad() {
        super.viewDidLoad()
        setupButtons()
    }

    func setupButtons() {
        let buttonConfig = UIImage.SymbolConfiguration(textStyle: .headline, scale: .small)
        playImageView.preferredSymbolConfiguration = buttonConfig
        playImageView.image = UIImage(systemName: "play.fill")
        shuffleImageView.preferredSymbolConfiguration = buttonConfig
        shuffleImageView.image = UIImage(systemName: "shuffle")
    }

    @IBAction func playAction(_ sender: Any) {
    }
    
    @IBAction func shuffleAction(_ sender: Any) {
    }
    
}
```

```swift
// SF Symbols in SwiftUI
import SwiftUI

struct ContentView: View {
    var body: some View {
        Image(systemName: "shuffle")
            .font(.headline)
            .imageScale(.small)
    }
}
```
New this year, we have `Label` which is a image + text.
```swift
// SF Symbols in SwiftUI
import SwiftUI

struct ContentView: View {
    var body: some View {
        Label("Sharing location", 
              systemImage: "location.fill")
    }
}
```

Can also do `Text("\(image) foo")`

```swift
// SF Symbols in SwiftUI
import SwiftUI

struct ContentView: View {
    var body: some View {
        let glyph = Image(systemName: "location.fill")
        return Text("\(glyph) Sharing location")
    }
}
```

[[20/what's new in swiftui]]

SFSymbols are now available on the mac.

AppKit:

```swift
// Using SF Symbols in AppKit

if let shuffleImage = NSImage(
    systemSymbolName: "shuffle", accessibilityDescription: "shuffle") {
    shuffleImageView.image = shuffleImage
  
    // Configure symbols
    let config = NSImage.SymbolConfiguration(textStyle: .body, scale: .small)
    let shuffleButtonImage = shuffleImage.withSymbolConfiguration(config)
}
```

# Additions and refinements
iOS 13: 1500 symbols
iOS 14: +750

Optically aligning images.

Template that allows you to create SFSymbols.
3 rows: small,medium, large
9 columns: for each weight
Margins are now lines instead of rectangles.  Therefore you can set positive/negative lines.

```swift
// loading symbols from Template V1 and V2

let shuffleImage = UIImage(systemName: "shuffle")
```

# Localization
English uses the latin alphabet, which is left-to-right.
In other locales, writing system is right-to-left.  SFSymbols are localized automatically.

If localized, app will show you what the variants are.  Can also implement for custom symbols on a per-svg basis.  Can also set "mirror" in asset catalog.

# Deployment targets
Some symbols were renamed/reordered this year.

If your app DT is iOS 13, look for symbols available in old OS.  Those symbols may have a different name, so you have to use the old name.

If your minimum DT is iOS 14, you can use the new name.  Xcode is aware of these name changes and will warn you accordingly.

# Colors
How to do multiple colors in a symbol?

We now have predefined-color symbols.  Can access them with multicolor.  Can choose between 2 preview modes, monochrome vs multicolor.

```swift
// Tinting symbols

if let folder = NSImage(
    systemSymbolName: "folder.badge.plus", accessibilityDescription: "add folder") {
    folder.isTemplate = true //default behaviof
}

if let folder = NSImage(
    systemSymbolName: "folder.badge.plus", accessibilityDescription: "add folder") {
    folder.isTemplate = false //multicolor
}
```
# Layout with SF Symbols
Symbols are *not designed to be constrained in frames to a specific size*.

Frees you from having to specify frames/alignment that will make it hard to leverage features.

Symbols have their own size, different amount of surface, but coherent releative to each other.  Beware of using `sizeThatFits`.  Instead rely on matching point size.

Use center alignments rather than aspectFit/scaleToFit.  And ensure `contentGravity` is `centered` if you're using CALayer.

(horiz) Align symbols to baseline, not center.

Vertically, use alignment guides to make the symbols/text... ?

# recap

```swift
// Using SF Symbols in AppKit

if let shuffleImage = NSImage(
    systemSymbolName: "shuffle", accessibilityDescription: "shuffle") {
    shuffleImageView.image = shuffleImage
  
    // Configure symbols
    let config = NSImage.SymbolConfiguration(textStyle: .body, scale: .small)
    let shuffleButtonImage = shuffleImage.withSymbolConfiguration(config)
}
```

```swift
// Tinting symbols

folder.isTemplate = true

folder.isTemplate = false
```

