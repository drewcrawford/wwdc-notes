# Meet the Contact Access Button

Learn about the new Contacts authorization modes and how to improve Contacts access in your app. Discover how to integrate the Contact Access Button into your app to share additional contacts on demand and provide an easier path to Contacts authorization. We'll also cover Contacts security features and an alternative API to be used if the button isn't appropriate for your app.

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

### Appearance options - 6:10
```swift
ContactAccessButton(queryString: searchText)
  .font(.system(weight: .bold))
  .foregroundStyle(.gray)
  .tint(.green)
  .contactAccessButtonCaption(.phone)
  .contactAccessButtonStyle(ContactAccessButton.Style(imageWidth: 30))
```

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

# Resources
* [Accessing a person’s contact data using Contacts and ContactsUI](https://developer.apple.com/documentation/Contacts/accessing-a-person-s-contact-data-using-contacts-and-contactsui)
* [Forum: App & System Services](https://developer.apple.com/forums/topics/app-and-system-services?cid=vf-a-0010)
* [HD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10121/4/A4253FF7-546D-4248-9DFA-DACBFB567A90/downloads/wwdc2024-10121_hd.mp4?dl=1)
* [SD Video](https://devstreaming-cdn.apple.com/videos/wwdc/2024/10121/4/A4253FF7-546D-4248-9DFA-DACBFB567A90/downloads/wwdc2024-10121_sd.mp4?dl=1)