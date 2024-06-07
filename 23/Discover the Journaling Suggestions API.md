Find out how the new Journaling Suggestions API can help people reflect on the small moments and big events in their lives though your app — all while protecting their privacy. Learn how to leverage the API to retrieve assets and metadata for journaling suggestions, invoke a picker on top of the app, let people save suggested content, and more.

Benefits:
* wellbeing
* mental health

difficult to know where to start.  iOS can sense meaningful events and purpose a starting point.
iOS sensed journaling suggestions

Private access picker.  Runs in a separate process, rendered on top of it.  Only what a person selects is passed back.  Can review iOS generated iteas before choosing and sending a suggestion to your app.

# Journaling Suggestions API

recommended, recent.

advanced ML techniques.  Photo memories, weekly summaries, multiday trips.

Recent tab -> more temporal manner.  

review assets in alrger size.  About to share the data with your app.  List view allows them to review all the metadata associated with each asset.

Select/unselect any asset and curate what should be sent.  Even edit the suggestion title.

Confirmation button to deliver content.  No permissions required because we run out of process.

# Invoking the picker
* add journaling suggestions capability
* import framework
* Create picker instance
* Show the picker

`JOurnalingSuggestionsPicker`.  SwiftUI only?

# Retrieve suggestion details

9 asset types.

* workout (with routes or not)
* contact (name, URL)
* location (place name, city name, coordinates, location)
* song (name, artist, album name, artwork url, timestamp)
* podcast (name, episode, artwork, timestamp)
* photo (url, timestamp)
* livePhoto (preview url, video url, timestamp)
* video (video url, timestamp)
* motion activity (step count, icon url, timestamp)

retrieved as AsyncImage.  


# Generic assets retrieval

some assets can be retrieved as UIImage or Image.
contact, song, podcast, photo, livephoto.  Use URL content to save image to disk.  

`suggestion.content(for Type: UIImage.self)`.  Pass to `UIImageWrapper`.

# Configure sheet content

customize suggestions.  UI that has toggles for
* activity
* media
* contacts
* photos
* significant locations

configuration applies to all applications, in settings app.  Privacy & security, journaling suggestions.

# Wrap up
* experiences that help reflect
* Present picker on top of your app
* Retrieve suggestion assets and details
* All in a privacy-preserving manner!

