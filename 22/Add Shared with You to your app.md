#sharedwithyou

I'll introduce shared with you and how to adopt in your app.
* safari
* news
* music
* podcasts
* tv app
* photos

Content is received when we aren't ready to consume it.  ex, friend sends recommendation for TV show.  SWY makes it easy to discover this tv show later on when browsing tv shows.

pick up messages conversation right from within the app.  Allowing you to stay in context of shared content without leaving the app.

# Pinning in messages
* Gesture in messages
* elevates content in Shared with You and Search
* Starts the flow of automatic sharing content

We extended it to your app, links, and content.

# Design
Two parts.  
1.  Shelf
2. Attribution view

## Shelf
dedicated space in your app's browsing experience that highlights content shared in messages.  TV app's "watch now" tab has a shared with you shelf.  So does the listen now tab, music, and podcasts.

ranked and ordered list.  For each item, show a 
* rich preview for content
* thumbnail, title, and subtitle (if applicable)
* attribution view

Have a show more element that can expand the view and navigate to all shared with you content.
attribution view => out of process view that displays the names and avatars.  If the content was pinned in messages.
Show the attribution view in the details view of your content.

## Attribution view actions
* Interactive
* contextual menu
	* reply => similar to tapping the view
	* remove => tell SWY not to surface this content going forward
	* can add to your content's contextual menus.
	* title can be customized.
	* ex safari shows "remove link"
* custom context menu


# How it works
* Links shared by friends and family in messages
* at least one contact in group conversation
* Uses universal links

Users ahve granulary control.  Can choose what content is shared outside of messages on a conversation, app, or global basis.

Doesn't need to be requested in advance.  Happens "organically".

## Automatic sharing
* Pinned content is always available
* heuristics-based automatic sharing enablement
	* when automatic sharing is turned on, further content is automatically available

## content ordering
1.  Siri suggestions
2. Pinned links
3. Chronological order

siri suggestion signals:
* user viewed or interacted with content
* content pinned?
* presentation context
your app plays a part in providing this feedback.

In a conversation, when a link is shared multiple times, SWY surfaces only the most recent.  When a link is shared in multiple messages conversations, SWY visually references via attribution view.  Tapping on attribution view presents disambiguation menu.

## Privacy/security
* views are drawn out of process
* app content is only accessible to your app

apps do not have access to messages recipients and conversations!
# How to adopt SWY
* adopt universal links
* add the Shared with You capability
* Add Shared with You shelf
* Add attribution views for your content

## universal links
* create two-way association between apps and website
* update app delegate to respond to user activity object
[[What's new in Universal Links]]

## SWY framework
* SWHighlightCenter
	* helps you get SWY content for your app
* SWHighlight
	*  Model objecct that wraps your app's shared content
* SWAttributionView
	* connects yourt content back to a messages conversation
	* displays attribution information
## SWHighlightCenter
Highlights
* array of highlight objects
* Delegate
	* new highlights

SWHighlight => passes URL to app's content that was shared in messages.  refer to your content, render a rich preview, and navigate to the content in your app.

```swift
// Enumerate Shared with You shelf

class SharedWithYouViewController: UIViewController, SWHighlightCenterDelegate {
    let highlightCenter = SWHighlightCenter()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        highlightCenter.delegate = self
    }
    
    func highlightCenterHighlightsDidChange(_ highlightCenter: SWHighlightCenter) {
        for highlight in highlightCenter.highlights {
            let highlightURL = highlight.url
            // Generate a rich preview for the Highlight
        }
    }
}
```

Apps can keep previous list of highlights in order to diff that list against the latest list.

as easy as that.  How to add, customize the attribution view, for your app's SWY content.

## SWAttributionView
* highlight
	* triggers out of process rendering of attribution information.
* maximum width
* horizontal alignment


```swift
// Setting appearance of Attribution View

let attributionView = SWAttributionView()
attributionView.highlight = self.highlightCenter.highlights[index]
attributionView.preferredMaxLayoutWidth = maximumWidthForView
```

This expands the bottom of the shared content.  Use UIView to set `.minimumContentSizeCategory` etc.

View's height is dependent on preferredContentSize category.  If view's height is unnecessarily constrained, view might be clipped or not yet drawn.

`.horizontalAlignment` is set to `.leading` in this case.    also `.center` or `.trailing`.

```swift
// Horizontal Alignment for Attribution View

let attributionView = SWAttributionView()
attributionView.highlight = self.highlightCenter.highlights[index]
attributionView.preferredMaxLayoutWidth = maximumWidthForView
attributionView.horizontalAlignment = .leading
```

Customization:
* Display context
* background style

Default value for display context is "summary".  This shows the content is displayed "for consumption".  I guess like a list of things.

`.detail` => actively consuming content, watching a movie, etc.

```swift
// Display Context for Attribution View

let attributionView = SWAttributionView()
attributionView.highlight = self.highlightCenter.highlights[index]
attributionView.preferredMaxLayoutWidth = maximumWidthForView
attributionView.horizontalAlignment = .center
attributionView.displayContext = .summary
```

backgroundStyle = `.color` => over monochrome backgrounds
`.material` => over multicolor backgrounds.  This sets a background blur for the view's contents.  Safari's landing page has a background image.

## Context menus
* highlight menu
* custom title for remove
	* string should include the word "remove", localized correctly


```swift
// Add Shared with You Content Menu to your app’s content

let attributionView = SWAttributionView()
attributionView.highlight = self.highlightCenter.highlights[index]
attributionView.menuTitleForHideAction = "Remove Item"

let contextMenuConfig = UIContextMenuConfiguration(identifier: nil,previewProvider: nil) { [weak self] _ in
        let additionalMenu = attributionView.supplementalMenu
        // Append additionalMenu items to your content’s menu items
}
```

# Recap
* Adopt universal links
* add the SWY capability
* Import SWY framework

