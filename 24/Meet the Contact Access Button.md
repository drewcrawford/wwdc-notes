# Meet the Contact Access Button

Learn about the new Contacts authorization modes and how to improve Contacts access in your app. Discover how to integrate the Contact Access Button into your app to share additional contacts on demand and provide an easier path to Contacts authorization. We'll also cover Contacts security features and an alternative API to be used if the button isn't appropriate for your app.

# Limited access

now has the option to only share a portion of the contacts db.  

Contacts auth prompt comes in two stages.
* whether to share contacts or not
* select contacts, allow full access

Four authorization levels.

| authorization level | read | write |
| ------------------- | ---- | ----- |
| full                | yes  | yes   |
| limited             | some | yes   |
| Not determined      | no   | no    |
| Denied              | no   | no    |

Fits into existing UI and can grant access to contacts with a single tap.
# Contact access button

Introducing the best way to handle limited access.

Streamlined way to receive access for additional contacts in your app.
Shows contact search result.
Seamlessly blends into your UI.
When the button has a unique match, it only takes one tap to grant you access to the contact.  Full-featured contact picker without requiring full access.
Incremental access

* Request access when required
* INtuitive and obvious need
contact access button shows intuitive benefit.
Easier than upfront request
Fast onboarding

Shows simplified prompt to request limited access.  Since this occurs immediately after tapping contact search result, easy to understand why you want access.



### Using ContactAccessButton - 5:15
```swift
// Using ContactAccessButton

@Binding var searchText: String
@State var authorizationStatus: CNAuthorizationStatus = .notDetermined

var body: some View {
    List {
        ForEach(searchResults(for: searchText)) { person in
            ResultRow(person)
        }
        if authorizationStatus == .limited || authorizationStatus == .notDetermined {
            ContactAccessButton(queryString: searchText) { identifiers in
                let contacts = await fetchContacts(withIdentifiers: identifiers)
                dismissSearch(withResult: contacts)
            }
        }
    }
}
```

we only show if limited or not determined authorization status.
When I initialize the button I pass in the text for the search field.  Receive an array of identifier string.

Button is designed to be tailored to fit the appearance of your app with swiftui modifiers.  Controls appearance of upper line of text and trailing action label.

Can style, etc.

* font
* foregroundStyle
* tint
* contactAccessButtonCaption
### Appearance options - 6:10
```swift
ContactAccessButton(queryString: searchText)
  .font(.system(weight: .bold))
  .foregroundStyle(.gray)
  .tint(.green)
  .contactAccessButtonCaption(.phone)
  .contactAccessButtonStyle(ContactAccessButton.Style(imageWidth: 30))
```

Contents are private
Taps are securely verified
Contents always legible

Button legibility.  Button must be legible and unobstructed, otherwise it won't work.
see things like contrast ratio, clipping.  

# Accessing contacts

Contacts.framework has 3 other ways to access data.

CNContactStore - primary gateway to contact data
Presents access prompt on use.
Reports authorization status
Read/write requires authorization.
Notifies when data changes

Limited access and CNContactStore
Limited status changes which contacts are accessible

`CNAuthorizationStatus.limited`.
Useful for showing alternate UI
Use with `ContactAccessButton` returned identifiers.
### Fetching contacts with CNContactStore - 10:11
```swift
// Fetching contacts with CNContactStore

func fetchContacts(withIdentifiers identifiers: [String]) async -> [CNContact] {
    return await Task {
        let keys = [CNContactFormatter.descriptorForRequiredKeys(for: .fullName)]
        let fetchRequest = CNContactFetchRequest(keysToFetch: keys)
        fetchRequest.predicate = CNContact.predicateForContacts(withIdentifiers: identifiers)
        var contacts: [CNContact] = []
        do {
            try CNContactStore().enumerateContacts(with: fetchRequest) { contact, _ in
                contacts.append(contact)
            }
        } catch {
            // ...
        }
        return contacts
    }.value
}
```

`CNContactPickerViewController`

System ui for picking one or more contacts from the contact database
Delivers snapshot of data
No authorization required
Implicit permission via interaction
Good for one-off tasks

Contact access picker - manage limited access set
Best for bulk access changes
Access picker **not** contact picker.

### Using contactAccessPicker - 12:47
```swift
// Using contactAccessPicker

@State private var isPresented = false

var body: some View {
    Button("Show picker") {
        isPresented.toggle()
    }.contactAccessPicker(isPresented: $isPresented) { identifiers in
        let contacts = await fetchContacts(withIdentifiers: identifiers)
        // use the new contacts!
    }
}
```

four different ways to access contacts

| authorization level | CNContactPickerViewController | CNContactStore | ContactAccessButton | contactAccessPicker |
| ------------------- | ----------------------------- | -------------- | ------------------- | ------------------- |
| full                | yes                           | yes            | not needed          | not needed          |
| limited             | yes                           | partial        | yes                 | yes                 |
| not determined      | yes                           | prompt         | yes                 | no                  |
| denied              | yes                           | no             | no                  | no                  |


# Next steps
* Test your apps with limited access
* adopt the contact access button
* Use the best contacts access method for your app

# Resources
* [Accessing a person’s contact data using Contacts and ContactsUI](https://developer.apple.com/documentation/Contacts/accessing-a-person-s-contact-data-using-contacts-and-contactsui)
* [Forum: App & System Services](https://developer.apple.com/forums/topics/app-and-system-services?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10121/4/A4253FF7-546D-4248-9DFA-DACBFB567A90/downloads/wwdc2024-10121_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10121/4/A4253FF7-546D-4248-9DFA-DACBFB567A90/downloads/wwdc2024-10121_sd.mp4?dl=1)