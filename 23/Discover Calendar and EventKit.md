#calendar #eventkit
Discover how you can bring Calendar into your app and help people better manage their time. Find out how to create new events from your app, fetch events, and implement a virtual conference extension. We'll also take you through some of the changes to calendar access levels that help your app stay connected without compromising the privacy of someone's calendar data.

# Integrate with calendar
Different roles that app can fill.  When put together, these different roles provide for a richer experience.

1.  Making reservations, buying tickets, arranging meetups.  Add events.
2. Displaying events
3. Modify events (bidirectional)
4. Extend virtual conferencing

# Frameworks overview
EventKit -> used to work directly with calendar data
EventKitUI -> iOS/catalyst that provides VCs for showing calendar UI.

EKEventStore -> main point of contact for calendar data.  Request access, only have one
EKEvent -> specific event.  Properties like title, start date, end date, and location
EKCalendar -> title, color.  Useful for coloring the events
EKSource -> collection of calendars.  Sources are useful for keeping calendars in your UI.

EventKitUI
* EKEVentEditViewController -> event editor.  Use this to add new events or make changes to existing events.
* EKEventViewController -> shows event details.  Use this to show information in your app about existing events
* EKCalendarchooser -> shows calendar lists and supports single or multiselection.  Use this to let people choose a calendar to add an event to or choose which calendar should be visible.

System prevents apps from reading/writing calendar events without permission.  3 level of access

| foo                                                  | no access | write only access | full access |
| ---------------------------------------------------- | --------- | ----------------- | ----------- |
| add events with eventkitui or siri event suggestions | yes (new) | yes (new)         | yes         |
| add events with EventKit                             | no        | yes (new)         | yes         |
| fetch or update existing events                      | no        | no                | yes         |
| Access or create calendars                           | no        | no                | yes            |


# Adding events
Add new events.  Events can be added in a few different ways.
* eventKitUI
* siri event suggestions
adds one event at a time.
* EventKit -> more events?

Present EKEventEditViewController
The user decides whether to save the event
No access needed (new)
* EventKitUI runs in a separate process
1.  Create an event store
2. create an event
3. create a VC
4. Present the VC
## Adding an event with EventKitUI - 5:49
```swift
// Create an event store
let store = EKEventStore()

// Create an event
let event = EKEvent(eventStore: store)
event.title = "WWDC23 Keynote"
let startDateComponents = DateComponents(year: 2023, month: 6, day: 5, hour: 10)
let startDate = Calendar.current.date(from: startDateComponents)!
event.startDate = startDate
event.endDate = Calendar.current.date(byAdding: .hour, value: 2, to: startDate)!
event.timeZone = TimeZone(identifier: "America/Los_Angeles")
event.location = "1 Apple Park Way, Cupertino, CA, United States"
event.notes = "Kick off an exhilarating week of technology and community."

// Create a view controller
let eventEditViewController = EKEventEditViewController()
eventEditViewController.event = event
eventEditViewController.eventStore = store
eventEditViewController.editViewDelegate = self

// Present the view controller
present(eventEditViewController, animated: true)
```

Use date components to make the start date.  Calculate the end date.

Use these offset APIs to pick end dates, to avoid weird DST issues.
Default timezone is the current system timezone.
Set a location.  Full address or mapkit handle.
Add notes to add extra detail.

Use delegate to find out whether people add events or not.

For a more complete example, see the sample code.

## Siri suggestions
for reservations made in your app.  From the Intents framework
No access needed.
No UI in your app
Events appear in the calendar inbox

* use for reservations
	food lodging
	travel
	ticketed events
Can be updated

1.  Create an INReservation
2. Create an intent and response
3. Create an INInteraction
4. Donate the interaction to the system


## Siri Event Suggestions - 9:17
```swift
// Create an INReservation
let spokenPhrase = “Lunch at Caffè Macs”
let reservationReference = INSpeakableString(vocabularyIdentifier: "df9bc3f5",
                                             spokenPhrase: spokenPhrase,
                                             pronunciationHint: nil)
let duration = INDateComponentsRange(start: myEventStart, end: myEventEnd)
let location = CLPlacemark(location: myCLLocation,
                           name: "Caffè Macs",
                           postalAddress: myAddress)
let reservation = INRestaurantReservation(itemReference: reservationReference,
                                          reservationStatus: .confirmed,
                                          reservationHolderName: "Jane Appleseed",
                                          reservationDuration: duration,
                                          restaurantLocation: location)

// Create an intent and response
let intent = INGetReservationDetailsIntent(reservationContainerReference:
    reservationReference)
let intentResponse = INGetReservationDetailsIntentResponse(code: .success, userActivity: nil)
intentResponse.reservations = [reservation]

// Create an INInteraction
let interaction = INInteraction(intent: intent, response: intentResponse)

// Donate the interaction to the system
interaction.donate()
```

See docs to learn more.

[[Broaden Your Reach with Siri Event Suggestions]]

Write-only access
* show custom editing UI
* add multiple events at a time
* Add events without user interaction

NSCalendarsWriteOnlyAccessUsageDescription key.  Shown in the request prompt.

People may choose "Don't allow".
Cannot
* access existing events, including yours?
* access or create calendars

[[23/What's new in Privacy|What's new in Privacy]]

Add event with EventKIt.

## Adding an event with write-only access - 12:41
```swift
// Create an event store
let store = EKEventStore()

// Request write-only access
guard try await store.requestWriteOnlyAccessToEvents() else { return }

// Create an event
let event = EKEvent(eventStore: store)
event.calendar = store.defaultCalendarForNewEvents
event.title = "WWDC23 Keynote"
event.startDate = myEventStartDate
event.endDate = myEventEndDate
event.timeZone = TimeZone(identifier: "America/Los_Angeles")
event.location = "1 Apple Park Way, Cupertino, CA, United States"
event.notes = "Kick off an exhilarating week of technology and community."

// Save the event
guard try eventStore.save(event, span: .thisEvent) else { return }
```

When saving an event directly with eventkit, nothing will be filled in for you.  What you set is what will be saved.

Some properties are required
1.  Calendar.  Use `store.defaultCalendarForNewEvents`.
2. title, startDate, endDate
3. others are optional, but fill out as much as possible.

see sample code

# Full access

for the *very few* apps that need to read calendar data, there's full access.  Only request if you need to

* display events
* update or remove events
`NSCalendarsFullAccessUsageDescription` info plist key.

See HIG: accessing private data

1.  Create an event store
2. request full access
3. create a predicate
4. fetch the events

## Fetch events - 15:51
```swift
// Create an event store
let store = EKEventStore()

// Request full access
guard try await store.requestFullAccessToEvents() else { return }

// Create a predicate
guard let interval = Calendar.current.dateInterval(of: .month, for: Date()) else { return }
let predicate = store.predicateForEvents(withStart: interval.start,
                                         end: interval.end,
                                         calendars: nil)

// Fetch the events
let events = store.events(matching: predicate)

let sortedEvents = events.sorted { $0.compareStartDate(with: $1) == .orderedAscending }
```

See sample code.

## Supporting earlier releases
* Perform runtime check for API availability
Call new request API in iOS 17 and macOS sonoma
call legacy request API in earlier releases

before iOS17:
* NSCalendarsUsageDescription
* NSContactsUsageDescription to use EventKitUI
* If these are missing then the app will crash


# Virtual conferences
Two ways that these extensiona re used.  When adding a location to an event, custom virtual conference options will appear in the location picker.

tapping will add the virtual conference to the event.

Making a virtual conference extension takes just a few steps.

1.  Create the extension target
2. Implement `fetchAvailableRoomTypes` to provide
3. Implement `fetchVirtualConference` to provide the object for a selected room type.

## Virtual conference extension - 19:18
```swift
// Create the extension target
class VirtualConferenceProvider: EKVirtualConferenceProvider {
  
    // Provide the room types
    override func fetchAvailableRoomTypes() async throws ->
        [EKVirtualConferenceRoomTypeDescriptor] {
        let title = "My Room"
        let identifier = "my_room"
        let roomType = EKVirtualConferenceRoomTypeDescriptor(title: title, identifier: identifier)
        return [roomType]
    }

    // Provide the virtual conference
    override func fetchVirtualConference(identifier: EKVirtualConferenceRoomTypeIdentifier)
        async throws -> EKVirtualConferenceDescriptor {
        let urlDescriptor = EKVirtualConferenceURLDescriptor(title: nil, url: myURL)
        let details = "Enter the meeting code 12345 to enter the meeting."
        return EKVirtualConferenceDescriptor(title: nil,
                                             urlDescriptors: [urlDescriptor],
                                             conferenceDetails: details)
    }
  
}
```

Your app will appear in calendar app's location picker.  

Think about hwo your app can contribute

* use eventkitUI or siri event suggestions to add events without requresting access
* request the minimum access needed, when it's needed
* implement a virtual conference extension



# Resources
* https://developer.apple.com/documentation/eventkit/accessing_calendar_using_eventkit_and_eventkitui
* https://developer.apple.com/documentation/eventkit
* https://developer.apple.com/documentation/sirikit/siri_event_suggestions
* https://developer.apple.com/ios/universal-links/
