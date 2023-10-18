Simplify your SwiftUI data models with Observation. We'll share how the Observable macro can help you simplify models and improve your app's performance. Get to know Observation, learn the fundamentals of the macro, and find out how to migrate from ObservableObject to Observable.

# What is observation?

Normal swift types, transform them with macros.

###  Using @Observable - 1:26
```swift
@Observable class FoodTruckModel {    
    var orders: [Order] = []
    var donuts = Donut.all
}
```
Make UI respond to changes in your data models.  Make models simpler than ever.  This uses the new macro system in swift.  `@Observable` tells the compiler to transform your code.

Makes the type able to be observed.  Use observable types to power your SwiftUI views.
###  SwiftUI property tracking - 2:12
```swift
@Observable class FoodTruckModel {    
  var orders: [Order] = []
  var donuts = Donut.all
}

struct DonutMenu: View {
  let model: FoodTruckModel
    
  var body: some View {
    List {
      Section("Donuts") {
        ForEach(model.donuts) { donut in
          Text(donut.name)
        }
        Button("Add new donut") {
          model.addDonut()
        }
      }
    }
  }
}
```

SwiftUI knows that the model accesses specific properties when executing the body call.  
###  SwiftUI computed property tracking - 3:12
```swift
@Observable class FoodTruckModel {    
  var orders: [Order] = []
  var donuts = Donut.all   var orderCount: Int { orders.count }
}

struct DonutMenu: View {
  let model: FoodTruckModel
    
  var body: some View {
    List {
      Section("Donuts") {
        ForEach(model.donuts) { donut in
          Text(donut.name)
        }
        Button("Add new donut") {
          model.addDonut()
        }
      }
      Section("Orders") {
        LabeledContent("Count", value: "\(model.orderCount)")
      }
    }
  }
}
```

Computed properties follow the same rules.  The UI updates.  In the newly added content, the orderCount accesses orders.  So when orders change, that text will be updated.


* macro
* Tracks access
* Property changes cause UI updates

[[Write Swift Macros]]
[[Expand on Swift macros]]

# SwiftUI property wrappers
`@State` `@Environment` and `@Bindable`.  We’ve covered the case where you don’t need them.

###  Using @State - 4:41
```swift
struct DonutListView: View {
    var donutList: DonutList
    @State private var donutToAdd: Donut?

    var body: some View {
        List(donutList.donuts) { DonutView(donut: $0) }
        Button("Add Donut") { donutToAdd = Donut() }
            .sheet(item: $donutToAdd) {
                TextField("Name", text: $donutToAdd.name)
                Button("Save") {
                    donutList.donuts.append(donutToAdd)
                    donutToAdd = nil
                }
                Button("Cancel") { donutToAdd = nil }
            }
    }
}
```

View needs to have its own state stored in a model.  Here we have the `@Observable` model.  When the sheet is presented, Donuś state variable is used to bind values to the editable fields.

Managed by the lifetime fo the view it’s contained in.

@Environment.  Propagate values as globally accessible.  Types work fantastically here since updates created by them are based upon access.  

When username will change, menu view will update.

###  Using @Environment - 5:14
###  Using @State - 4:41
```swift
struct DonutListView: View {
    var donutList: DonutList
    @State private var donutToAdd: Donut?

    var body: some View {
        List(donutList.donuts) { DonutView(donut: $0) }
        Button("Add Donut") { donutToAdd = Donut() }
            .sheet(item: $donutToAdd) {
                TextField("Name", text: $donutToAdd.name)
                Button("Save") {
                    donutList.donuts.append(donutToAdd)
                    donutToAdd = nil
                }
                Button("Cancel") { donutToAdd = nil }
            }
    }
}
``````swift
@Observable class Account {
  var userName: String?
}

struct FoodTruckMenuView : View {
  @Environment(Account.self) var account

  var body: some View {
    if let name = account.userName {
      HStack { Text(name); Button("Log out") { account.logOut() } }
    } else {
      Button("Login") { account.showLogin() }
    }
  }
}
```

`@Bindable`.  Lightweight.  All it does is allow bindings to be created from that type.
* connect reference to UI.
* Use $ syntax to create bindings.
###  Using @Bindable - 6:27
```swift
@Observable class Donut {
  var name: String
}

struct DonutView: View {
  @Bindable var donut: Donut

  var body: some View {
    TextField("Name", text: $donut.name)
  }
}
```

Ex here we write back to the binding.  To make bindings to the donut, we need to use @Bindable.  This allows us to use $donut.name.

1.  Part of the view?  State var
2. Global to application?  Environment
3. Just need bindings?  Bindable
4. Or var!
# Advanced uses

Because SwiftUI tracks access to fields per instance, can use arrays, ptionals, or any type that contains your observable models.

###  Storing @Observable types in Array - 7:53
```swift
@Observable class Donut {
  var name: String
}

struct DonutList: View {
  var donuts: [Donut]
  var body: some View {
    List(donuts) { donut in
      HStack {
        Text(donut.name)
        Spacer()
        Button("Randomize") {
          donut.name = randomName()
        }
      }
    }
  }
}
```

SwiftUI detects access to the property on a per-instance basis.  So here when the donut name is changed, the view updates accordingly.  Build your models how you want.  

General rule is, for observable, if a property that is used changes, the view changes.

but if a computed property does not have any stored property, then two extra steps need to be taken to make it work with observation.  Only when the property tat would be observed needs to be changed not via composition of stored properties in observable types.

Tell observation when the property is accessed and when it changes.

###  Manual Observation - 9:18
```swift
@Observable class Donut {
  var name: String {
    get {
      access(keyPath: \.name)
      return someNonObservableLocation.name 
    }
    set {
      withMutation(keyPath: \.name) {
        someNonObservableLocation.name = newValue
      }
    }
  } 
}
```

Here we’ve written out the expanded version manually.  Most of the time you don’t need this, because most of the time the properties of the models in question are composed of other stored properties.  But if you need that advanced capability, you can do thi son your own.

* SwiftUI tracks access
* Composed properties
* Manual control when needed

ObservableObject.  Achieve someo f the same things we did previously.  If you have an app that uses SwiftUI today, be in a very similar situation.

# ObservableObject

Observable macro can simplify your code and has a decent performance boost.  

Remove the conformance to ObservableObject, remove @published, and conform to macro.

In all cases of `@Observed`, disappeared or needed just bindings.  Changed to new `@Bindable`.  

Changing from observable object to macro was mostly just deleting annotations or simplifying down to property wrappers.

# Harness the magic
* macro for observing changes
* New projects use `@Observable`
* Simplify code for new features






