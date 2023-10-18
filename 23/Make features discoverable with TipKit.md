#tipkit

# Feature discoverability

across various platforms
# Create a tip

```swift
struct FavoriteBackyardTip: Tip {

    var title: Text {
        Text("Save as a Favorite")
    }
    
    var message: Text {
        Text("Your favorite backyards always appear at the top of the list.")
    }
}
```

direct action phrases.

actionable, instructionable, and easy to remember.
Don't use tipkit for 
* promotional
* error message (ex sign in)
* not actionable (ex better feature)
* too complicated to read/remember
```swift
@main
struct BackyardBirdsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }

    // ...

    init() {
        TipsCenter.shared.configure()
    }
}
```

this enables key tipkit functionality.  Have them eprsist between appl aunches, etc.  A default shared instance is provided and is what I've added here.

icon, color, etc.

```swift
struct FavoriteBackyardTip: Tip {

    var title: Text {
        Text("Save as a Favorite").foregroundColor(.indigo)
    }
    
    var message: Text {
        Text("Your favorite backyards always appear at the top of the list.")
    }
    
    var asset: Image {
        Image(systemName: "star")
    }
    
    var actions: [Action] {
        [
            Tip.Action(
                id: "learn-more", 
                title: "Learn More"
            )
        ]
    }
}
```

you may want to add an acction to customize settings.  Take people directly to settings so they can make adjustments.

can also link to additional resources such as an onboarding flow.

* popover view.  Points to button or similar
	* on tvOS, this is the only option
* inline view
	* adjusts teh app's UI to fit around it, so no UI is blocked.

You want the tip close to the relevant button though

# Eligibility rules
Only if the feature is of interest and any education doesn't come off as spammy or irrelevant.  Favoritng a backyard is useful, but not for everyone.

in-app education can focus on those who benefit from it.  Whiel they're trying to accomplish something in an app.

## Rule types
parameter-based rules
* state and boolean comparison
event based rules
* user actions makes eligible


```swift
private let favoriteBackyardTip = FavoriteBackyardTip()

// ...

.toolbar {
    ToolbarItem {
        Button {
            backyard.isFavorite.toggle()
        } label: {
            Label("Favorite", systemImage: "star")
                .symbolVariant(
                    backyard.isFavorite ? .fill : .none
                )
        }
        .popoverMiniTip(tip: favoriteBackyardTip)
    }
}
```

Let's restrict to logged in.
```swift
struct FavoriteBackyardTip: Tip {

    @Parameter
    static var isLoggedIn: Bool = false

    // ...

    var rules: Predicate<RuleInput...> {

        // User is logged in
        #Rule(Self.$isLoggedIn) { $0 == true }
    }
}
```

when they've entered the detail view at least 3 times.
```swift
struct FavoriteBackyardTip: Tip {

    @Parameter
    static var isLoggedIn: Bool = false
  
    static let enteredBackyardDetailView: Event = Event<DetailViewDonation>(
        id: "entered-backyard-detail-view"
    )

    // ...
    
    var rules: Predicate<RuleInput...> {
        // User is logged in
        #Rule(Self.$isLoggedIn) { $0 == true }
            
        // User has entered any backyard detail view at least 3 times
        #Rule(Self.enteredBackyardDetailView) { $0.count >= 3 }
    }
}
```

donate when it appears

```swift
.onAppear {
     FavoriteBackyardTip.enteredBackyardDetailView.donate()
}
```

now we require 3x in the past 5 days, e.x. people regularly use the app.
```swift
// User has entered any backyard detail view at least 3 times in the past 5 days
#Rule(Self.enteredBackyardDetailView) { 
    $0.donations.filter { 
        $0.date > Date.now.addingTimeInterval(-5 * 60 * 60 * 24)
    }
    .count >= 3
}
```

another featuer is the ability to create custom donations by adding associated types to event donations, and querying the events based on those types.

We can refine so it matches only when someone has gone through a specific detail view.

```swift
// Create the associated type
extension BackyardDetailTip {
    struct DetailViewDonation: DonationValue {
        let backyardID: Int
    }
}

// Donate the unique id of the backyard detail being viewed
.onAppear {
     BackyardFavoriteTip.enteredBackyardDetailView.donate(
         with: .init(backyardID: backyard.id)
     )
}


struct FavoriteBackyardTip: Tip {

    // ...

    var rules: Predicate<RuleInput...> {
        // Update the rule to specify a backyardID
        #Rule(Self.enteredBackyardDetailView) {
            $0.donations.filter {
                $0.date > Date.now.addingTimeInterval(-5 * 60 * 60 * 24) 
            }
            .largestSubset(by: \.backyardID)
            .count >= 3
        }
    }
}
```

Keep in mind evaluation order for performance.

# Display and dismissal

limiting tips

```swift
// One tip per day.
TipsCenter.shared.configure {
    DisplayFrequency(.daily)
}

// One tip per hour.
TipsCenter.shared.configure {
    DisplayFrequency(.hourly)
}

// Custom configuration. Only show one tip every five days.
let fiveDays: TimeInterval = 5 * 24 * 60 * 60
TipsCenter.shared.configure {
    DisplayFrequency(fiveDays)
}

// No frequency control. Show all tips as soon as eligible.
TipsCenter.shared.configure {
    DisplayFrequency(.immediate)
}
```
Ignore display frequency per-tip.
```swift
struct FavoriteBackyardTip: Tip {

    // ...
  
    var options: [Option] {
        [.ignoresDisplayFrequency(true)]
    }
}
```

How to dismiss tips?  Maybe when the user does the action?

```swift
Button {
    backyard.isFavorite.toggle()

    // When user taps the favorite button, dismiss the tip
    favoriteBackyardTip.invalidate(reason: .userPerformedAction)
} label: {
    Label("Favorite", systemImage: "star")
        .symbolVariant(backyard.isFavorite ? .fill : .none)
}
.popoverMiniTip(tip: favoriteBackyardTip)
```

alternatively let's say they see it 5 times

```swift
struct FavoriteBackyardTip: Tip {

    // ...

    var options: [Option] {
        [.maxDisplayCount(5)]
    }
}
```

Syncs status to iCloud to ensure that tips seen on one device won't be seen on the other.

# Test tips

```swift
// Show all defined tips in the app
TipsCenter.showAllTips()

// Show some tips, but not all
TipsCenter.showTips([tip1, tip2, tip3])

// Hide some tips, but not all
TipsCenter.hideTips([tip1, tip2, tip3])

// Hide all tips defined in the app
TipsCenter.hideAllTips()

// Purge all TipKit related data
TipsCenter.resetDatastore()
```

also available as launch arguments.

```swift
// Show all defined tips in the app
com.apple.TipKit.ShowAllTips 1

// Show some tips, but not all
com.apple.TipKit.ShowTips tipID,otherTipID

// Hide some tips, but not all
com.apple.TipKit.HideAllTips 1

// Hide all tips defined in the app
com.apple.TipKit.HideTips tipID,otherTipID

// Purge all TipKit related data
com.apple.TipKit.ResetDatastore 1
```

# Wrap up
* explore discoverability opportunities
* keep tips short and actionable
* target the ideal audience
