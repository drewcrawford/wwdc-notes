Use value types where possible.  Classes are reference types and if you decide to use them, swift manages them via ARC.

To write swift, need to understand how ARC works.  In this session, we do just that.

# Object lifetimes and ARC
* Object's lifetime begins at `init()` and ends at last use
* ARC deallocates an object after its lifetime ends
* ARC tracks object's lifetime with reference count
* Swift compiler inserts retain/release operations
* Swift runtime deallocates object with 0 refcount

```swift
class Traveler {
    var name: String
    var destination: String?
}

func test() {
    let traveler1 = Traveler(name: "Lily") //REF BEGIN, +1
	//retain to prepare for traveler2
	let traveler2 = traveler1 //REF BEGIN
	//REF END
	//release (init)
    traveler2.destination = "Big Sur"
	//REF end
	//release (traveler2 assignment)
    print("Done traveling")
}
```

Swift compiler infers retain when reference begin and release after last use.

Once refcount drops to 0, object can be deallocated.

Object lifetimes are use-based.  Begins at init and ends at last use.  Different from C++ in which an object is guaranteed to end at closing brace.

In practice, object lifetimes are dtermined by retain/release inserted by the swift compiler.  Observed object lifetimes may differ than the guaranteed minimum and may exceed the last use of the object.

In such cases, the object is deallocated beyond last use.  Usually this doesn't really matter, however, with language features like weak and unowned references and deinit side-effects, it's possible to observe lifetimes.

If you rely on observed object lifetimes you can end up with problems.  An emergent property of the swift compiler and can change when implementation details change

May remain hidden for a long time, uncovered at surprising times with a compiler update or source changes.

# Observable object lifetimes

* Language features
* Consequences and safe techniques

Unlike default strong references, weak and unowned references do not participate in reference counting.  For this reason they are commonly used to break reference cycles.  What are they?

```swift
class Traveler {
    var name: String
    var account: Account?
    func printSummary() {
        if let account = account {
            print("\(name) has \(account.points) points")
        }
    }
}

class Account {
    var traveler: Traveler
    var points: Int
}

func test() {
	//traveler +1
    let traveler = Traveler(name: "Lily")
	//account +1.  Since this refers to traveler, we retain traveler.
    let account = Account(traveler: traveler, points: 1000)
	//we also retain account
    traveler.account = account
    traveler.printSummary()
}
```

In the test function, we create traveler and account objects and then call `printSummary`.  Let's see what happens.

After all references go away, the reference count of the object remain 1 because of reference cycle.  Objects are never deallocated, causing a memory leak.

* Break reference cycles
* Referred object can be deallocated while in use
* Access to weak reference returns nil
* Access to unowned reference traps

Can be marked as weak or unowned to break the reference cycle.

Now with weak

```swift
class Traveler {
    var name: String
    var account: Account?
    func printSummary() {
        if let account = account {
            print("\(name) has \(account.points) points")
        }
    }
}

class Account {
    weak var traveler: Traveler?
    var points: Int
}

func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
    traveler.printSummary()
}
```

Now traveler drops to 0.  Traveler is deallocated, account is released, account is deallocated.

In this example we used the weak reference to break the reference cycle.  If a weak reference is used to access an object while its guaranteed lifetime has ended, and you are relying on the observed lifetime... you can have problems.

```swift
class Traveler {
    var name: String
    var account: Account?
}

class Account {
    weak var traveler: Traveler?
    var points: Int
    func printSummary() {
        print("\(traveler!.name) has \(points) points")
    }
}

func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
    account.printSummary()
}
```

What exactly whappens when `printSummary` is called?  It may print today, but this is only a coincidence.  Reference count can drop to 0.

You may be wondering, force-unwrap is the reason for the crash.  Optional binding?

However, optional binding is not the problem.  Without the crash, you may have a silent bug.  When an observed object changes for unrelated reason.

## Safe techniques
* `withExtendedLifetime()`
* redesign to access via strong reference
* Redesign to avoid weak/unowned reference


### withExtendedLifetime
```swift
func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
	//extend for duration of closure
    withExtendedLifetime(traveler) {
        account.printSummary()
    }
}

func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
    account.printSummary()
	//extend until hit here
    withExtendedLifetime(traveler) {}
}

func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
	//extended until end of current scope
    defer {withExtendedLifetime(traveler) {}}
    traveler.account = account
    account.printSummary()
}
```

Now can extend traveler lifetime.  While this may look like an easy way out, this can be fragile and transfers responsibility of correctness onto you.  You should ensure `withExtendedLifetime` is used every time a weak reference has a potential to cause bugs.

Redesign classes with better APIs.  e.g. hide the weak reference.

```swift
class Traveler {
    var name: String
    var account: Account?
    func printSummary() {
        if let account = account {
            print("\(name) has \(account.points) points")
        }
    }
}

class Account {
    private weak var traveler: Traveler?
    var points: Int
}

func test() {
    let traveler = Traveler(name: "Lily")
    let account = Account(traveler: traveler, points: 1000)
    traveler.account = account
    traveler.printSummary()
}
```

 It is important to stop and think: why are weak and  unowned references needed?  Are they used only to break reference cycles?  What if you avoid reference cycles?
 
 ## avoid weak
 
 It is not necessary for the account class to refer to the traveler class.
 
 ```swift
 class PersonalInfo {
    var name: String
}

class Traveler {
    var info: PersonalInfo
    var account: Account?
}

class Account {
    var info: PersonalInfo
    var points: Int
}
```

This is a definite way to eliminate all potential object lifetime bugs.

## Deinit side effects
Runs before deallocation
Observable side-effects
* global
* side-effects on exposed class details

```swift
class Traveler {
  var name: String
  var destination: String?
  deinit {
    print("\(name) is deinitializing")
  }
}

func test() {
    let traveler1 = Traveler(name: "Lily")
    let traveler2 = traveler1
    traveler2.destination = "Big Sur"
    print("Done traveling")
}
```

Here, deinit has a global side effect: printing.  Today the deinit may run after done is printed.  But since the last field is used, can run before.

```swift
class Traveler {
    var name: String
    var id: UInt
    var destination: String?
    var travelMetrics: TravelMetrics
    // Update destination and record travelMetrics
    func updateDestination(_ destination: String) {
        self.destination = destination
        travelMetrics.destinations.append(self.destination)
    }
    // Publish computed metrics
    deinit {
        travelMetrics.publish()
    }
}

class TravelMetrics {
    let id: UInt
    var destinations = [String]()
    var category: String?
    // Finds the most interested travel category based on recorded destinations
    func computeTravelInterest()
    // Publishes id, destinations.count and travel interest category
    func publish()
}

func test() {
    let traveler = Traveler(name: "Lily", id: 1)
    let metrics = traveler.travelMetrics
    ...
    traveler.updateDestination("Big Sur")
    ...
    traveler.updateDestination("Catalina")
	//note that traveler's last use is here.  So deinitializer can run here.
	//If so, `nil` is published, causing a bug.
    metrics.computeTravelInterest()
}

verifyGlobalTravelMetrics()
```

Different techniques 
### Safe techniques

#### withExtendedLifetime

Can extend until it's computed (end of scope etc).  This transfers the responsibility of correctness onto you.  

#### Redesign to limit visiblity of internal class details

```swift
class Traveler {
    ...
	//by making this private, we require interested
	//parties to go through us, so we can't be deallocated
    private var travelMetrics: TravelMetrics
    deinit {
        travelMetrics.computeTravelInterest()
        travelMetrics.publish()
    }
}

func test() {
    let traveler = Traveler(name: "Lily", id: 1)
    ...
    traveler.updateDestination("Big Sur")
    ...
    traveler.updateDestination("Catalina")
}
```

#### gEt rid of deinit side effects altogether

```swift
class Traveler {
    ...
    private var travelMetrics: TravelMetrics
     
    func publishAllMetrics() {
        travelMetrics.computeTravelInterest()
        travelMetrics.publish()
    }

    deinit {
				assert(travelMetrics.published)
    }
}

class TravelMetrics {
    ...
    var published: Bool
    ...
}

func test() {
    let traveler = Traveler(name: "Lily", id: 1)
    defer { traveler.publishAllMetrics() }
    ...
    traveler.updateDestination("Big Sur")
    ...
    traveler.updateDestination("Catalina")
}
```

deinitializer only performs verification.  by removing init side effects, we can eliminate potential lifetime bugs.

Important to throughly understand the language features that make object lifetimes observable.  Don't uncover bugs at surprising times.

A new experimental build setting called "Optimize Object lifetimes" is available.  This enables powerful ARC optimizations.  With this build setting turned on, you may see objects deallocated immediately more consistently, bringing observed object lifetimes closer to the guarnateed minimum.  This may expose hidden bugs.

You can follow the safe techniques to eliminate these bugs.


