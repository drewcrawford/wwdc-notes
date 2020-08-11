#watchos

# Watch apps are lightweight
# Keep it actionable
Consider filtering.
# Make actions discoverable
Previously, actions were in a gesture-based contextual menu (force-touch).
This year, we've moved these into the apps.

1.  Interactions should be discoverable and predictable
2.  Actions should always be visible
3.  Eliminate gesture-based contextual menus without removing functionality

Many of the actions we wanted to display were *secondary actions*

# Secondary Actions
not the primary reason you're using a watch app.  e.g. filtering, sorting, etc.

## Sort/filter
* sort by
* "view switcher"

```swift
var body: some View {
	List {
		Picker(selection: $viewing title: Text("Viewing")) {
			//viewing options
		}
		//stocks
	}
}
```

## Swipe actions
```swift
var body: some View {
	List {
		ForEach(model.locations) {
			ClockCell(location: $0)
		}
		.onDelete { deleteclock(index: $0)}
	}
}
```

## More options

Recommend adding at the bottom of list.  But what if it's not a list?  e.g., maps?

"More" button.

You can create your own more button with SFSymbol ellipsis and a circle.  Add transparent padding if needed to ensure it's tappable.

What if you have a single secondary option that you need to provide?  Create a button specifically for that action.

Never put primary actions inside a more button.  And be choosey about which secondary interactions you include there too.


## Actions in a scrolling view
May be the most discoverable and intuitive way to show actions in your app.

## Toolbar button
We designed this for compose in messages/mail.  Tucked beneath the navigation bar.
I likely didn't open this app to compose a new message.  Probably to look at a new message that came in.

But the compose button is there in the toolbar, and I can tuck it back away when I'm not using it.

```swift
var body: some View {
	ConversationList() {
		.toolbar {
			Button(action: newMessage) {
				Label("new message",systemImage:"square.and.pencil")
			}
		}
	}
}
```

Only use this in scrolling view.  Scrolling is what makes this button discoverable.

Only use this for actions that are essential, but may not be the primary action.


## Hierarchical navigation
In some cases, it makes sense to land one level in at launch.  e.g. mail.

App should remember the level I chose.  If your app doesn't have that level of permanence, this si not the right model to use.

