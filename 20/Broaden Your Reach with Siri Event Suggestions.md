# New categories and platforms
## categories
* restaurants
* movies
* ticketed events
* car rentals
* lodging
* flights
* train
* bus (new) -> Intended for trips that are a specific time.
* boat (new)

## platforms
(new) macOS big sur.

[[Integrating with Siri Event Suggestions - 19]]

# Using markup for websites/emails
## Siri event suggestison markup
Open standard markup vocabulary
Support for JSON-LD and microdata
Apple documentation and guidelines.

### json-ld
```html
<script type="application/ld+json">
{
  "@context": "http://schema.org",
  "@type": "FoodEstablishmentReservation",
  "reservationStatus": "http://schema.org/ReservationConfirmed",
  "reservationId": "IWDSCA",
  "partySize": "2",
  "reservationFor": {
    "@type": "FoodEstablishment",
    "name": "EPIC Steak",
    "startDate": "2020-06-26T19:30:00-07:00",
    "telephone": "(415)369-9955"
    "address": {
      "@type": "http://schema.org/PostalAddress",
      "streetAddress": "369 The Embarcadero",
      "addressLocality": "San Francisco"
      "addressRegion": "CA",
      "postalCode": "95105",
      "addressCountry": "USA"
    }
  }
}
</script>
```

### microdata
```html
<div itemscope itemtype="FoodEstablishmentReservation"> 
  <link itemprop="reservationStatus" href="http://schema.org/ReservationConfirmed"/>
  <meta itemprop="reservationId" content="IWDSCA"/>
  <meta itemprop="partySize" content="2"/>
  <div itemprop="reservationFor" itemscope itemtype="FoodEstablishment">
    <meta itemprop="name" content="EPIC Steak"/>
    <meta itemprop="startDate" content="2020-06-26T19:30:00-07:00"/>
    <meta itemprop="telephone" content="(415)369-9955"/>
    <div itemprop="address" itemscope itemtype="PostalAddress">
      <meta itemprop="streetAddress" content="369 The Embarcadero"/>
      <meta itemprop="addressLocality" content="San Francisco"/>
      <meta itemprop="addressRegion" content="CA"/>
      <meta itemprop="postalCode" content="95105"/>
      <meta itemprop="addressCountry" content="USA"/>
    </div>
  </div>
</div>
```

As long as you keep the same reservation ID across updates, siri will be able to understand that a reservation changed.

Can also use cancelled to tell siri the event was cancelled.

### requirements
* Must register your domain with Apple.
* Source verification (DKIM, HTTPS)

For development, can disable the verification.

Open developer settings and enable "Allow any domain".

`defaults write com.apple.suggestions SuggestionsAllowAnyDomainForMarkup -bool true`

"Allow unverified sources" maps to

`defaults write com.apple.suggestions SuggestionsAllowUnverifiedSourceForMarkup -bool true`

### Demo
Looks like this is added to calendar in some kind of "pending" state by default.
Wonder how well that works for spam?  I wonder if that's why they have the app review process?

### guidelines
* respect for format and type of each property
	* Use appropriate `reservationStatus`
	* Use ISO8601 format for dates and times
* keep `reservaionId` consistent for updated and cancelled reservations

### next steps
1.  Markup
2.  Test
3.  Register

developer.apple.com/contact/request/siri-events


# Donations best practices
* Reservation number -> identify a reservation across app, website, email
* container reference -> uniquely identifies the reservation *within your app*
* item reference -> multiple items within the reservation.  e.g., each leg of flight.  The system may launch your app to display a single item reservation

## single part reservation
reservation number -> use for container reference as well
also use for item reference
`.spokenPhrase` -> "Table at ACME Eatery"

## Round trip travel
reservation number -> identifies all (2) parts of reservation.
Also use for container reference.
2 train reservation objects.  Pick an identifier to each part.

## Multiple reservations
e.g. a travel app might offer bundles of travel, lodging, and so on.  Let's say roundtrip flights, plus hotel.

* Reservation
	* Flight 1
	* Flight 2
	* Hotel

## How does this work if displaying on a calendar on a device with no app installed?
* New URL property
* "Show in Safari" button

## other

Now, system will intelligently group notifications when multiple donations are made together.

ideal place to do this is when your app is displaying a list of reservations.

We also recommend you donate when viewing a single reservation, or a part (such as 1 ticket for a movie).  System will only display notifications the first time an item is donated.

Also good to donate when a reservation is changed (e.g. push notification).  

[[Advances in App Background Execution - 19]]

# Debugging tips

We added logging (Console.app).  category: `siri-event-suggestions`.

Siri won't process donations until you acknowledge "what's new in calendar"

