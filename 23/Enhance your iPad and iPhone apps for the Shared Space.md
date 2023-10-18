Get ready to enhance your iPad and iPhone apps for the Shared Space! We'll show you how to optimize your experience to make it feel great on visionOS and explore Designed for iPad app interaction, visual treatments, and media.

Most iPad/Iphone apps run great without any changes.  

See [[Run your iPad and iPhone apps in the Shared Space]] to learn about systems of builtin behaviors, differences, testing setup, etc.

# Interaction

Key components: new natural input technique.

tap allows peopel to looka t a button, then tap their finger to interact.  Tap to ggle, tap, hold, swipe.

Direct touch requires reaching out to the app and touching the button in the space with one finger.  Regardless of interaction method, the button provides ocntinuous visual feedback to help with accuracy.

In this video, the cursor represents where the person is looking.  Highlight tints color.

Controls that are inactive to not get hover effects.  System controls take care of hover effects for you.

If you're building custom controls, your hover effects may need some tuning.

Same app in simulator.  Because the menu button is a system control, it's working as expected.  But each card is a vstack and does not receive hover.

Hover effects needed to communicate it's tappable.
## Tappable VStack with hover effect: 3:02
```swift
struct TappableCard: View {
   // Sample card
   var imageName = "BearsInWater"
   var headline = "Bear Fishing"
   var timeAgo = "42 Minutes ago"
   
   var body: some View {
      VStack {
         VStack(alignment: .leading) {
            Image(imageName)
               .resizable()
               .clipped()
               .aspectRatio(contentMode: .fill)
               .frame(width: 300, height: 250, alignment: .center)
            Text(headline)
               .padding([.leading])
               .font(.title2)
               .foregroundColor(.black)
         }
         Divider()
         HStack {
            HStack {
               Text(timeAgo)
                  .frame(alignment: .leading)
                  .foregroundColor(.black)
            }
            .padding([.leading])
            Spacer()
            VStack(alignment: .trailing) {
               Button { print("Present menu options") } label: {
                  Image(systemName: "ellipsis")
                     .foregroundColor(.black)
               }
            }
         }
         .padding(EdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5))
      }
      .frame(width: 300, height: 350, alignment: .top)
      .hoverEffect()
      .background(.white)
      .overlay(
         RoundedRectangle(cornerRadius: 10)
            .stroke(Color(.sRGB, red: 150/255, green: 150/255, blue: 150/255, opacity: 0.1), lineWidth: 3.0)
      )
      .cornerRadius(10)
      .onTapGesture {
         print("Present card detail")
      }
   }
}
```

We use `.hoverEffect()` to make it interactable.  

If we zoom into our example

## 4:08 - Custom player with tap targets that are larger than the hover effect bounds
```swift
struct ContentView: View {
   var body: some View {
      VStack {
         // Video player
         HStack {
            Button { print("Going back 10 seconds") } label: {
               Image(systemName: "gobackward.10")
                  .padding(.trailing)
                  .contentShape(.hoverEffect, CustomizedRectShape(customRect: CGRect(x: -75, y: -40, width: 100, height: 100)))
                  .foregroundStyle(.white)
                  .frame(width: 500, height: 834, alignment: .trailing)
            }
            Button { print("Play") } label: {
               Image(systemName: "play.fill")
                  .font(.title)
                  .foregroundStyle(.white)
                  .frame(width: 100, height: 100, alignment: .center)
            }
            .padding()
            Button { print("Going into the future 10 seconds") } label: {
               Image(systemName: "goforward.10")
                  .padding(.leading)
                  .contentShape(.hoverEffect, CustomizedRectShape(customRect: CGRect(x: 0, y: -40, width: 100, height: 100)))
                  .foregroundStyle(.white)
                  .frame(width: 500, height: 834, alignment: .leading)
            }
         }
         .frame(
              minWidth: 0,
              maxWidth: .infinity,
              minHeight: 0,
              maxHeight: .infinity,
              alignment: .center
         )
      }
      .frame(
           minWidth: 0,
           maxWidth: .infinity,
           minHeight: 0,
           maxHeight: .infinity,
           alignment: .topLeading
      )
      .background(.black)
   }
}

struct CustomizedRectShape: Shape {
   var customRect: CGRect
   
   func path(in rect: CGRect) -> Path {
      var path = Path()
      
      path.move(to: CGPoint(x: customRect.minX, y: customRect.minY))
      path.addLine(to: CGPoint(x: customRect.maxX, y: customRect.minY))
      path.addLine(to: CGPoint(x: customRect.maxX, y: customRect.maxY))
      path.addLine(to: CGPoint(x: customRect.minX, y: customRect.maxY))
      path.addLine(to: CGPoint(x: customRect.minX, y: customRect.minY))
      
      return path
   }
}
```

we can use a custom shape which will be smalelr than the tappable region.

Hover effect
* custom button
* custom shaped hover effect
* opt out


## 5:14 - Button with custom buttonStyle, then adding a hover effect to the button
```swift
struct ContentView: View {
    var body: some View {
        VStack {
         Button("Howdy y'all") { print("🤠") }
            .buttonStyle(SixColorButton())
        }
        .padding()
    }
}

struct SixColorButton: ButtonStyle {
   func makeBody(configuration: Configuration) -> some View {
      configuration.label
         .padding()
         .font(.title)
         .foregroundStyle(.white)
         .bold()
         .background {
            // Background color bands
            ZStack {
               Color.black
               HStack(spacing: 0) {
                  // GREEN
                  Rectangle()
                     .foregroundStyle(Color(red: 125/255, green: 186/255, blue: 66/255))
                     .frame(width: 16)
                  // YELLOW
                  Rectangle()
                     .foregroundStyle(Color(red: 240/255, green: 187/255, blue: 64/255))
                     .frame(width: 16)
                  // ORANGE
                  Rectangle()
                     .foregroundStyle(Color(red: 225/255, green: 137/255, blue: 50/255))
                     .frame(width: 16)
                  // RED
                  Rectangle()
                     .foregroundStyle(Color(red: 200/255, green: 73/255, blue: 65/255))
                     .frame(width: 16)
                  // PURPLE
                  Rectangle()
                     .foregroundStyle(Color(red: 134/255, green: 64/255, blue: 151/255))
                     .frame(width: 16)
                  // BLUE
                  Rectangle()
                     .foregroundStyle(Color(red: 75/255, green: 154/255, blue: 218/255))
                     .frame(width: 16, height: 500)
               }
               .opacity(0.7)
               .rotationEffect(.degrees(35))
            }
         }
         .cornerRadius(10)
         .hoverEffect()
   }
}
```

custom styling turns off hover effect  We need to re-add `.hoverEffect()` to re-enable it.




## 5:46 - Honey comb app with custom shape buttons, then adding hover effects that clip to bounds of the honey comb shape

```swift
struct ContentView: View {
    var body: some View {
      VStack {
         Button { print("🐝") } label: {
            // Button label
            HoneyComb()
               .fill(.yellow)
               .frame(width: 300, height: 300)
               .contentShape(.hoverEffect, HoneyComb())
            }
         }
         .frame(width: 400, height: 400, alignment: .center)
         .background(.black)
         .padding()
      }
    }
}

struct HoneyComb: Shape {
   func path(in rect: CGRect) -> Path {
      var path = Path()
      path.move(to: CGPoint(x: rect.minX + (rect.width * 0.25), y: rect.minY))
      path.addLine(to: CGPoint(x: rect.maxX - (rect.maxX * 0.25), y: rect.minY))
      path.addLine(to: CGPoint(x: rect.maxX, y: rect.midY))
      path.addLine(to: CGPoint(x: rect.maxX - (rect.maxX * 0.25), y: rect.maxY))
      path.addLine(to: CGPoint(x: rect.minX + (rect.width * 0.25), y: rect.maxY))
      path.addLine(to: CGPoint(x: rect.minX, y: rect.midY))
      path.addLine(to: CGPoint(x: rect.minX + (rect.width * 0.25), y: rect.minY))
      return path
   }
}
```

Default system-provided hovereffect covers the entire frame. By passing the custom shape, the hover effect will trim to the button's bounds.

Now when people look at individual buttons, the hover effect is trimmed.

## Opt out
* infrequent
* no global opt out
* overrhide over effect
* `.hoverEffectDisabled(Bool)`

Maximum of 2 simultaneous inputs.  Custom gesture recognizers are also supported, but may need to be updated to run smoothly.

Games or other apps that need rapid/simultaneous input need to support game controllers.  Long been supported on iOS, even more critical on this platform.

By including, `GCSupportsControlelrUserInteraction` and adding game controller capabilitiy, it adds a badge to product page

[[Advancements in Game Controllers]]
[[Build great games for spatial computing]]



# Visuals
In most cases, iPad light appearance looks great.  System optimizes rendering, using dynamic content scaling
* vector images
* prefer 2x size
prompts are presented modally.  Prompt must be interacted with before continuing.

**On this new platform, prompts do not present modally**.  Do not require handling before continuing.  These create their own window experience.  Apps should be aware where they get the prompt, but don't immediately get callbacks.

# Media

Some differences to be aware of.  Multiple external/internal cameras

**many cameras not available for app use**

Use discover sessions to detect which cameras are available for use.

* Availability API
* Request access
* Prompt text

AVCaptureDiscoverySession results.

Microphone -> single `.front` mic
Camera -> two cameras.  `.back` returns a black camera, nonfunctional camera.  Don't use.
front -> single composite camera.  "If no spatial persona is found, then no camera frames will return to apps."

Media playback
* custom players
	* AVRoutePickerView
	* Picture-in-picture
* not supported?  don't show controls?

Background audio
* Apps that utilize backgruond audio will no longer get background mode when device is locked, and will be fully suspended
alternative media sources
* cloud-based media
* pickers (document,photo)
* VNDocumentCameraViewController
	* automatically capture with continuity camera on a nearby device

# Next steps
* Hover effects
* Game controllers
* Availability check

[[Explore App Store Connect for spatial computing]]
[[Run your iPad and iPhone apps in the Shared Space]]
[[Build great games for spatial computing]]
[[Advancements in Game Controllers]]






