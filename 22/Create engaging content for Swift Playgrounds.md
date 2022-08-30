A number of tutorial example code things.  Building dynamic apps, customizing views, more advanced topic like async data fetch, etc.

# Overview

```swift
let numOfTiles = 100
let squareLength = 150.0

    // Dance floor
    ForEach(0 ..< numOfTiles, id: \.self) { index in
        let i: CGFloat = CGFloat(index / 10)
        let j: CGFloat = CGFloat(index % 10)

        let x = (squareLength * i) - (squareLength)
        let y = (squareLength * j) - (squareLength * 2)

        Rectangle()
            .frame(width: squareLength, height: squareLength)
            .border(.black, width: 3)
            .position(x: x, y: y)
            .randomizedColorEffect(startAnimation: startParty)
    }
    .blur(radius: 15)
    .opacity(startParty ? 1.0 : 0.0)
```


```swift
ForEach(data.creatures) { creature in
    Text(creature.emoji)
        .resizableFont()
        .animatedScalingEffect(startAnimation: startParty)
        .randomizedOffsetEffect(startAnimation: startParty,
                                x: midX * 0.6,
                                y: midY * 0.6)
        .animatedRotationEffect(startAnimation: startParty)
        .opacity(startParty ? 1.0 : 0.0)
}
```

```swift
Text("🪩")
    .resizableFont()
    .animatedRotationEffect(startAnimation: startParty)
    .opacity(startParty ? 1 : 0)
```

How to explain custom view modifiers?  could send them to documentation.  But can teach alonside the project code.  New system designed to help authors like you create engaging in-app experiences.

When a learner first opens, introduce with a welcome message.

At the top of the project editor.  On the righthand side is the learning center.  Designated area to add images, instructional text, etc.

Learning center contains a section of task.  Help guide learners, etc.  By tapping a task button, our instructional system will render a card.  This card can contain a series of pages with text, images, code snippets.  Later, marcus wil cover over two task types, walkthroughs and experiments.  

Build a compelling educational experience.


# The guide module
By default, the file structure keeps all files at its root.  Change file structure to create an App module.  Move all our project's sourcecode into App.  Package.swift stays at the root.

Create a Guide module at the same level of App.  Inside there, we have a Guide file.  This contains the prose of our content.

```swift
@GuideBook(title: "Creature Party!", icon: icon.png, background: background.png, firstFile: CreatureDance.swift) {
    
}
```

This acts as the main container for all our directives.  Title, icon/background, and the file you first want opened.

Then we have a welcome message.

```swift
@WelcomeMessage(title: "Welcome to Creature Party!") {
		In Creature Party, you'll take this app of dancing creatures to the next level with the help of colors, shapes, animations, and plenty of emoji!
}
```

Optional, presented to learner where they open the project
```swift
@Guide {
        @Step(title: "Pump up the jams") {
            
        }
    }
}
```

# Walkthroughs
Step directive is where our content will live.  To make a step, need to fill with two other directives.

```swift
@ContentAndMedia {
  	Tonight, the creatures are gonna party like it's 2022. 🐙💃🦝🕺🦦 
}
```

Longer prose and larger images that might help cover your topic.  While it's small in this example, view can extend further down.  Great place to write longer bits of prose and co mplex content such as diagrams.

```swift
@TaskGroup(title: "Walkthroughs") {
		Here are the walkthroughs! These will help explain all of the new code.                 
}
```

optional directive to put inside your steps if you want to clump tasks together.  if you have content which covers the same topics across multilpe files.

```swift
@Task(type: walkthrough, id: "partyMode", title: "Setting up the Party", file: CreatureDance.swift) {
}
```

what UI to generate when displaying the task.

Every task needs an ID.  Every ID in the guide must be unique.  title parameter is also as tring, this can be whatever you want and does not have to be unique.

Title sits inside the button and file of the walkthrough is written above it.

```swift
@Page(id: "1.modifier", title: "") {
  	This is a [view modifier](https://developer.apple.com/documentation/swiftui/viewmodifier). Modifiers let you create unique versions of a view in SwiftUI. 
}
```

Title paramter behaves a lot like the one for tasks, but when you leave the title frme empty, it lets the system know...


```swift
ForEach(data.creatures) { creature in
		Text(creature.emoji)
			.resizableFont()
			/*#-code-walkthrough(1.modifier)*/
			.animatedScalingEffect(startAnimation: startParty)
			/*#-code-walkthrough(1.modifier)*/
      .randomizedOffsetEffect(startAnimation: startParty,
      												x: midX * 0.6,
				                      y: midY * 0.6)
      .animatedRotationEffect(startAnimation: startParty)
      .opacity(startParty ? 1.0 : 0.0)
}
```

Let's open emoji app.  When you open the project, you have source editor on left and preview on right.  Welcome message.

```swift
@Page(id: "1.struct", title: "") {
  	Custom view modifiers are structures that contain code for explaining how to modify whatever view the given modifier is attached to.  
}
@Page(id: "1.body", title: "") {
  	The body method allows you to add custom view modifications. For example, here you're adding a scaling animation that grows and shrinks the `Creature` over a certain period of time. 
}
```

```swift
@Task(type: walkthrough, id: "protocol", title: "A Little More on Protocols", file: CreatureDance.swift) {
    @Page(id: "2.protocol", title: "") {
      All custom view modifiers implement the `ViewModifier` protocol. 
    }
    @Page(id: "2.body", title: "") {
      The `ViewModifier` protocol requires all structures that implement it to write the `body(content:)` method.   
    }
    @Page(id: "2.usage", title: "") {
      After you've written content for your `body(content:)` method, you can use it on any view you want. Here you'll use it on each `Creature` to add a rotation animation.
    }
}
```


# Experiments
Optional code learners can add if they're feeling extra curious or if they want to make the app unique to them.


```swift
@TaskGroup(title: "Experiments") {
    Time to set this party off! You can use experiments to add some extra pazazz to the dance floor.
    @Task(type: experiment, id: "colors", title: "Dancing in the Strobe Light", file: CreatureDance.swift) {

    }
}
```



```swift
@Page(id: "3.code", title: "", isAddable: true) {
    `\``
    .colorMultiply(creatureColor)
    `\`` 
}
```

isAddable => add code directly to the source editor.  Add button will appear.  Code must be wrapped in a code block with triple-backtick syntax.  Recommend keep lines to 10 or less.  Better that users don't have to scroll.



```swift
ForEach(data.creatures) { creature in
		Text(creature.emoji)
    		.resizableFont()
        /*#-code-walkthrough(1.modifier)*/
        .animatedScalingEffect(startAnimation: startParty)
        /*#-code-walkthrough(1.modifier)*/
        .randomizedOffsetEffect(startAnimation: startParty,
        												x: midX * 0.6,
        												y: midY * 0.6)
        /*#-code-walkthrough(2.usage)*/
        .animatedRotationEffect(startAnimation: startParty)
        /*#-code-walkthrough(2.usage)*/
        .opacity(startParty ? 1.0 : 0.0)
        //#-learning-task(colors)
}
```

After comes the pair of parentheses.  Inside we write the id of the experiment task.  And we have everything we need to test out the experiment task.  

```swift
ForEach(data.creatures) { creature in
		Text(creature.emoji)
    		.resizableFont()
        /*#-code-walkthrough(1.modifier)*/
        .animatedScalingEffect(startAnimation: startParty)
        /*#-code-walkthrough(1.modifier)*/
        .randomizedOffsetEffect(startAnimation: startParty,
        												x: midX * 0.6,
        												y: midY * 0.6)
        /*#-code-walkthrough(2.usage)*/
        .animatedRotationEffect(startAnimation: startParty)
        /*#-code-walkthrough(2.usage)*/
        .opacity(startParty ? 1.0 : 0.0)
        //#-learning-task(colors)
}
```

Before we add in our esecond experiment, let's add a page to the experiment that's already there.  It can be confusing for lsearners to add a blcok of code w/o knowing what it does.

Now we'll add our second task.  With that, we have a new piece of content to teach learners about custom view modifiers.


```swift
@Task(type: experiment, id: "timer", title: "Switch it Up", file: CreatureDance.swift) {
    @Page(id: "4.lights", title: "") {
      	Now that you have some colors, you can add some code to change the color of the creatures using a timer. Let's add one!
    }
    @Page(id: "4.code", title: "", isAddable: true) {
        `\``
        .onTapGesture {
          if let timer = timer {
            timer.invalidate()
            self.timer = nil
          } else {
            creatureColor = Color.randomColor
            timer = Timer.scheduledTimer(withTimeInterval: 2.0, repeats: true, block: { timer in
                                                                                       creatureColor = Color.randomColor
                                                                                      })
          }
        }
        `\`` 
    }
}
```

When I first open the project, welcome message animates in.  When I tap on the learn more butotn, it opens the learning center.  4 tasks added.  


* https://developer.apple.com/forums/tags/wwdc2022-110349
* https://developer.apple.com/forums/create/question?&tag1=237&tag2=532030
