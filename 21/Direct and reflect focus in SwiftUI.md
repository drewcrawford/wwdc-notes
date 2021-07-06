#swiftui 
Built-in components base default behavior on SwiftUI knowledge of platform conventions.

Focus.  System that lets your app take input from keyboards, remotes, game controllers, accessible switch controls, etc.

Not tied to specific screen coordinates.

Often, focused view is drawn with special embellishments.

There are however, some cases when you want a more accelerated focus experience than the default.

When you select "new note", we want focus to automatically move to the newly created note.  Requires custom implementation.

Move focus from button on the bottom left side to content near the top.  SwiftUI cannot guess where to move focus.

Want keyboard to go away where user displays an event.  Now you can do this in SwiftUI.

# Move and detect focus

Present an email, password, and sign in field.  
Programmatically move focus back to email field.

```swift
import SwiftUI
import AuthenticationServices

struct ContentView: View {

    @FocusState private var focusedField: Field?
    @State private var email: String = ""
    @State private var password: String = ""

    var body: some View {
        ZStack {
            Image("backgroundImage")
                .resizable()
                .opacity(0.7)
                .ignoresSafeArea()

            VStack(alignment: .center) {
                Text("Vacation Planner")
                    .font(.custom("Baskerville-SemiBoldItalic", size: 60))
                    .foregroundColor(.black.opacity(0.8))
                    .frame(alignment: .top)

                Spacer(minLength: 30)

                TextField("Email", text: $email)
                    .submitLabel(.next)
                    .textContentType(.emailAddress)
                    .keyboardType(.emailAddress)
                    .padding()
                    .frame(height: 50)
                    .background(Color.white.opacity(0.9))
                    .cornerRadius(15)
                    .padding(10)

                SecureField("Password", text: $password)
                    .submitLabel(.go)
                    .padding()
                    .frame(height:50)
                    .textContentType(.password)
                    .background(Color.white.opacity(0.9))
                    .cornerRadius(15)
                    .padding(10)

                Spacer().frame(height: 20)

                HStack {
                    Rectangle().frame(height: 1)
                    Text("or").bold().padding()
                    Rectangle().frame(height: 1)
                }
                .foregroundColor(.black.opacity(0.7))
                
                Spacer().frame(height: 20)

                SignInWithAppleButton(.signIn) { request in
                    request.requestedScopes = [.fullName, .email]
                } onCompletion: { result in
                    switch result {
                    case .success (_):
                        print("Authorization successful.")
                    case .failure (let error):
                        print("Authorization failed: " + error.localizedDescription)
                    }
                }
                .frame(height: 50)
                .cornerRadius(15)

                Spacer().frame(height: 20)

            }
            .frame(width: 280, height: 500, alignment: .bottom)
        }
    }

}
```

Can use any hashable type for `FocusState`.  Note that the value is optional.  In general, types must be `Hashable` and `Optional`.  `nil` is used for focus in unrelated parts of the screen.

 Creates a link between placement of focus and value of field.
 
 You can use current placement of focus for making other decisions.
 
 INitially, focusField is nil.  If you tap on email textfield, that gains focus
 
 Note that we use the `.focused($field, equals: .foo)` to match this to your custom enum.
 
 Link works both ways.  Means that we're not limited to reacting to focus changes, can move focus programmatically by updating the property.
 
 Set focus back to email and highlight email with invalid field.
 
 ```swift
 import SwiftUI
import AuthenticationServices

enum Field: Hashable {
    case email
    case password
}

struct ContentView: View {

    @FocusState private var focusedField: Field?
    @State private var email: String = ""
    @State private var password: String = ""
    @State private var submittedEmail: String = ""

    var body: some View {
        ZStack {
            Image("backgroundImage")
                .resizable()
                .opacity(0.7)
                .ignoresSafeArea()

            VStack(alignment: .center) {
                Text("Vacation Planner")
                    .font(.custom("Baskerville-SemiBoldItalic", size: 60))
                    .foregroundColor(.black.opacity(0.8))
                    .frame(alignment: .top)

                Spacer(minLength: 30)

                TextField("Email", text: $email)
                    .submitLabel(.next)
                    .textContentType(.emailAddress)
                    .keyboardType(.emailAddress)
                    .padding()
                    .frame(height: 50)
                    .background(Color.white.opacity(0.9))
                    .cornerRadius(15)
                    .padding(10)
                    .focused($focusedField, equals: .email)
                    .border(Color.red,
                            width: (focusedField == .email &&
                                    !isEmailValid) ? 2 : 0)

                SecureField("Password", text: $password)
                    .submitLabel(.go)
                    .padding()
                    .frame(height:50)
                    .textContentType(.password)
                    .background(Color.white.opacity(0.9))
                    .cornerRadius(15)
                    .padding(10)
                    .focused($focusedField, equals: .password)

                Spacer().frame(height: 20)

                HStack {
                    Rectangle().frame(height: 1)
                    Text("or").bold().padding()
                    Rectangle().frame(height: 1)
                }
                .foregroundColor(.black.opacity(0.7))
                
                Spacer().frame(height: 20)

                SignInWithAppleButton(.signIn) { request in
                    request.requestedScopes = [.fullName, .email]
                } onCompletion: { result in
                    switch result {
                    case .success (_):
                        print("Authorization successful.")
                    case .failure (let error):
                        print("Authorization failed: " + error.localizedDescription)
                    }
                }
                .frame(height: 50)
                .cornerRadius(15)

                Spacer().frame(height: 20)

            }
            .frame(width: 280, height: 500, alignment: .bottom)
            .onSubmit {
                submittedEmail = email
                if !isEmailValid {
                    focusedField = .email
                }
            }
        }
    }
  
    private var isEmailValid : Bool {
        let regex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format:"SELF MATCHES %@", regex)
        return submittedEmail.isEmpty || predicate.evaluate(with: submittedEmail)
    }

}
```
 
# Create navigation targets

Focus will only move if there is something adjacent and focusable in the direction.

Need to extend focusableArea to encompass something that is adjacent to the current field.

```swift
import SwiftUI
import AuthenticationServices


struct ContentView: View {

    @State private var email: String = ""
    @State private var password: String = ""

    var body: some View {
        HStack {
            VStack(alignment: .leading) {
                Spacer(minLength:60).frame(height: 150)
                Text("Vacation\nPlanner")
                    .font(.custom("Baskerville-SemiBoldItalic", size: 60))
                    .foregroundColor(Color.black.opacity(0.8))
                    .lineLimit(nil)
                    .multilineTextAlignment(.center)
                    .padding(.horizontal, 40)

                Spacer().frame(height:80)

                TextField("Email", text: $email)
                    .submitLabel(.next)
                    .textContentType(.emailAddress)
                    .keyboardType(.emailAddress)

                Spacer().frame(height:30)

                SecureField("Password", text: $password)
                    .submitLabel(.go)
                    .textContentType(.password)

                HStack {
                    Rectangle().frame(height: 1)
                    Text("or").bold().padding()
                    Rectangle().frame(height: 1)
                }
                .foregroundColor(Color.black.opacity(0.7))

                Spacer().frame(height: 20)

                SignInWithAppleButton(.signIn) { request in
                    request.requestedScopes = [.fullName, .email]
                } onCompletion: { result in
                    switch result {
                    case .success (_):
                        print("Authorization successful.")
                    case .failure (let error):
                        print("Authorization failed: " + error.localizedDescription)
                    }
                }
                .frame(height: 50)
                Spacer()
            }
            .frame(width: 350, alignment: .center)

            VStack {
                Image(photoName)
                    .resizable()
                    .frame(width: 1400)
                    .aspectRatio(contentMode: .fit)
                    .ignoresSafeArea(edges: [.trailing])
                BrowsePhotosButton()
            }
        }.preferredColorScheme(.light)
    }
}
```


1.  add `.focusSection()`.  Evidently we do this twice, not sure why

# Wrap up
* Think about focus for each platform
* SwiftUI has great default behavior
* Leverage powerful focus APIs

https://developer.apple.com/documentation/SwiftUI/View-Input-and-Events

