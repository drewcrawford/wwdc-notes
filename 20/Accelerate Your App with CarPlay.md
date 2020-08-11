#carplay

Until now, CarPlay templates have only been availble to navigation apps.
Today we're offering a new set of templates, plus improving existing templates.

# Design principles
* Designed for the driver, not for passengers
* Streamline interactions
* Reuse app configuration
* Launch first in CarPlay.  Eliminate any logic in your app that depends on being launched in iPhone.  Easy to do with `UIScene`, and in fact your app must adopt this isntead of `UIWindow` / `UIApplicationDelegate`.


# Audio apps
`PlayableContent` will be deprecated.  It's a metadata-based API, and the system assembles a tree-style UI on your behalf.  On iOS 13 and earlier, PC and audio templates can co-exist.  On iOS 13 and earlier, the system will launch your app with PlayabaleContent, and on 14 it will launch with templates if available.

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UISceneConfigurations</key>
	<dict>
		<key>CPTemplateApplicationSceneSessionRoleApplication</key>
		<array>
			<dict>
				<key>UISceneClassName</key>
				<string>CPTemplateApplicationScene</string>
				<key>UISceneConfigurationName</key>
				<string>MyApp—Car</string>
				<key>UISceneDelegateClassName</key>
				<string>MyApp.CarPlaySceneDelegate</string>
			</dict>
		</array>
	</dict>
</dict>
```
Note `UISceneDelegateClassName`.  We're specifying the name of a class for scene and scene delegate.

```swift
// CarPlay App Lifecycle

import CarPlay
//calls this when the app is launched
class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?
   
    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {

		//hold onto this object, because you'll need it later
        self.interfaceController = interfaceController
        let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
        let section = CPListSection(items: [item]) 
		//set template as my root template
        let listTemplate = CPListTemplate(title: "Albums", sections: [section])
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

  func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didDisconnect interfaceController: CPInterfaceController) {
	//release controller that we were holding
    self.interfaceController = nil
}
```

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let listTemplate = CPListTemplate(title: "Albums", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

```swift
//handle section in a list item
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
item.listItemHandler = { item, completion, [weak self] in
    // Start playback, then...
    self?.interfaceController.pushTemplate(CPNowPlayingTemplate.shared, animated: true)
	//if you don't immediately call the completion block, it's ok.  CarPlay will display a spinner to let the user know that your app is busy.
	//calling completion signals we can remove the spinner.
    completion()
}

// Later...
//we added support for changing list items this year.
//podcasts uses these dynamic updates to keep the progress indicator updated.
item.image = ...
```

## Tab bar template
* Can display several of your app's templates in a tab-bar style interface.

```swift
// CPTabBarTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let favorites = CPListTemplate(title: "Albums", sections: [section])
favorites.tabSystemItem = .favorites
favorites.showsTabBadge = true

let albums: CPGridTemplate = ...
albums.tabTitle = "Albums"
albums.tabImage = ...

//create a tabbar template by providing an array of templates, each becomes a tab.
let tabBarTemplate = CPTabBarTemplate(templates: [favorites, albums])
self.interfaceController.setRootTemplate(tabBarTemplate, animated: false)

// Later...
//(dynamic updates).  Can add/remove/rearrange tabs.
//here we show/hide badge
favorites.showsTabBadge = false
tabBarTemplate.updateTemplates([favorites, albums])
```
## `CPListImageRowItem` -> brings an artwork grid.  
```swift
// List Items for Audio Apps

import CarPlay

let gridImages: [UIImage] = ...
let imageRowItem = CPListImageRowItem(text: "Recent Audiobooks", images: gridImages) 

//just like the other list items, this one has a handler
//called whern the user selects the title
//in this example I might push a new item to show even more of the user's audiobooks
imageRowItem.listItemHandler = { item, completion in
    print("Selected image row header!")
    completion() //make sure to call completion handler
}

//each artwork is individually selectable.  So we also specify handler for this
//take an action in response to the specific audiobooks that the user selected
imageRowItem.listImageRowHandler = { item, index, completion in
    print("Selected artwork at index \(index)!")
    completion()
}

let section = CPListSection(items: [imageRowItem]) 
let listTemplate = CPListTemplate(title: "Listen Now", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

## Now playing template
* Optionally enable the "playing next" and the "album artist" button
* Custom buttons with your own custom images
* Several special behaviors
	* A shared instance.  Configure the properties of the shared interest and, if you enable the optional buttons, add at least one observer for button actions
	* Configure this immediately at launch.  System may display this template on your behalf, e.g. the now playing button on the home screen.
	* Now playing bar button will be added to your app's navigation bar or tab bar.
	* If a different button becomes the now playing app, OS will remove this button from your view hierarchy
	* Only the list template may be pushed on top of the NP template.  e.g., show the upcoming playback queue.


```swift
// Now Playing Template

import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {
      	//configure shared template immediately
        let nowPlayingTemplate = CPNowPlayingTemplate.shared

        let rateButton = CPNowPlayingPlaybackRateButton() { button in
                                                           
            // Change the playback rate!
                                                           
        }
		//enable the playing next button and add observers for the now playing template buttons
        nowPlayingTemplate.updateNowPlayingButtons([rateButton])
    }
}
```

# Communication apps
Previously known as messaging and voip apps.
Traditionally work by leveraging SiriKit and callkit.
Must continue to use these APIs to provide voice/telephoney

But now, can take advantage of carplay framework to show contacts, display messages, etc.

## Contact template
Don't remember this one.

## screencast

This app uses a tab bar template.
Ability to show list of message threads.  Starting in iOS 14, we introduce a new list item subclass called CP..ListItem
Handlers are not called when tapped by the user.  Instead, Siri will be automatically invoked when the user taps the list item.  The user will do reply flow through siri.
Can display unread indicator, pin, star, icon on lefthand side of cells.
Trailing end can be configured with mute indicator, text, and optional image.
`CPMessageListItem` supports dynamic updates. Just update the properties.
Ability to show contact information.
Templates support up to 3 lines of text and 4 buttons.

# Quick food ordering apps

## screencast
no detailed menu, account management, etc.
Easy one-step access to the most common tasks.
iOS introduces POI template.  Provide a list of locations and user can pick one.
Template provides pan/zoom and will notify you of reason changes.
12 CPPointOfInterest instances.
* Title
* subtitle
* informative text

Remember to show only the most relevant info.  On iPhone, your app might show all possible locations, but in CarPlay you should limit your list.  CP will automatically group nearby locations.

We have updated the pin image to show a different color.
These buttons can be used for a variety of tasks.

```swift
//handle POI map region changes
// CPPointOfInterestTemplateDelegate

func pointOfInterestTemplate(_ template: CPPointOfInterestTemplate, 
                             didChangeMapRegion region: MKCoordinateRegion) {

    self.locationManager.locations(for: region) { locations in
        template.setPointsOfInterest(locations, selectedIndex: 0)
    }
}
```
CP will now update locations
```swift
// CPPointOfInterest creation

func locations(for region: MKCoordinateRegion, 
               handler: ([CPPointOfInterest]) -> Void) {
    var tempateLocations: [CPPointOfInterest] = []
        
    for clientModel in self.executeQuery(for: region) {
        let templateModel : CPPointOfInterest = self.locations[clientModel.mapItem] ??
                CPPointOfInterest(location: clientModel.mapItem,
                                  title: clientModel.title,
                                  subtitle: clientModel.subtitle,
                                  informativeText: clientModel.informativeText,
                                  image: clientModel.mapImage)
            
            
        tempateLocations.append(templateModel)
    }
    handler(templateLocations)
}
```

```swift
// Point of Interest Template location selection

//assign a primary button
let primaryButton = CPPointOfInterestButton(title: "Select") { button, [weak self] in
            let selectedIndex = ...
            
            if selectedIndex != NSNotFound {
                // Remove any existing selected state on previous location
                self?.selectedLocation.image = defaultMapImage
                // Change annotation for selected POI
                self?.selectedLocation = templateModel
                templateModel.image = selectedMapImage
                // Update the template with new values
                self?.pointOfInterestTemplate.selectedIndex = selectedIndex
            }
        }

let templateModel: CPPointOfInterest = ...

templateModel.primaryButton = primaryButton
```

Once the user has chosen the location or performed a task, you may need to show a summary or finalize the operation.

## Information template
Displays like an "Order Summary" type thing.  List of labels that may be configured in 1/2 columns, and a set of footer buttons.

For quick food ordering apps, may be used as order summary / confirmation.


# EV charging and parking apps
Information template can be used to show information about a charging station.

# wrap up
## Entitlements
* CarPlay entitlement required
* Single category only.  Will need to choose the category you are supporting and use a single entitlement.
* Entitlement determines template availability.

developer.apple.com/carplay
CarPlay App Programming Guide
Sample template apps

[[Carplay Audio and Navigation Apps - 18]]
[[Introducing Siri MediaKit Intents - 19]]

