#swift
# Swift Algorithms

```swift
// Raw loop:
var selectedMessages: [Message] = []
for indexPath in indexPathsForSelectedRows {
    selectedMessages.append(messages[indexPath.row])
}

// Using `map` makes this clearer and faster.
indexPathsForSelectedRows.map { messages[$0.row] }
```

Map is faster because it avoids intermediate allocations due to array resizing.

```swift
// Raw loop:
var attachments: [Attachment] = []
for message in messages {
    if let attachment = message.attachment {
        attachments.append(attachment)
    }
}

// The above is just a `map` and a `filter`.
messages
    .filter { $0.attachment != nil }
    .map { $0.attachment! }

// This pattern is so common we have a special name and algorithm for it.
messages.compactMap { $0.attachment }
```

```swift
extension Message {
    func makeMessageParts() -> [TranscriptItem]
}

messages // [Message]
    .map { $0.makeMessageParts() } // [[TranscriptItem]]
    .joined() // [TranscriptItem]

// This pattern is so common that we have another special kind of map for it.
messages // [Message]
    .flatMap { $0.makeMessageParts() }  // [TranscriptItem]
```

Any given message might correspond to multiple items in the transcript.

```swift
// Raw loop:
var photos: [PhotoItem] = []
for item in transcript.reversed() {
    if let photo = item as? PhotoItem {
        photos.append(photo)
        if photos.count == 6 {
            break
        }
    }
}

// The above can be expressed more concisely by chaining together algorithms.
transcript
    .reversed() // [TranscriptItem]
    .compactMap { $0 as? PhotoItem } // [PhotoItem]
    .prefix(6) // [PhotoItem]

// This gives us more flexibility to express this code more clearly.
transcript
    .compactMap { $0 as? PhotoItem } // [PhotoItem]
    .suffix(6) // [PhotoItem]
    .reversed() // [PhotoItem]
```

If each step allocates an intermediate array, won't it be slow?

Turns out `joined` doesn't return an array.

```swift
extension Message {
    func makeMessageParts() -> [TranscriptItem]
}

messages
    .map { $0.makeMessageParts() } // [[TranscriptItem]]
    .joined() // FlattenSequence<[[TranscriptItem]]>
```

This is effectively free to create and is lazy so it processes elements on-demand.  

Note that `.compactMap` has to opt-in to being lazy with `.lazy`.  Lazy chains are a great fit for these usecases where you're only processing a small number of elements from a large collections.

Sometimes you need or want an array, in which case you can `Array(foo)`.

github.com/apple/swift-algorithms

The purpose is to provide a low-friction venue for us to incubate new families of missing algorithms for inclusion in stdlib.

* combinations/permutations
* iteration by groups
* N smallest/largest elements
	
## iteration tools

sliding window of fixed size
```swift
import Algorithms

let x = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

for window in x.windows(ofCount: 3) {
    print(window)
}

// Prints [0, 1, 2]
// Prints [1, 2, 3]
// Prints [2, 3, 4]
// Prints [3, 4, 5]
```

windows of count 2 are particularly common, `adjacentPairs`.  This vends a tuple allowing for more convenient element access.

chunks of count (don't overlap)
```swift
import Algorithms

let x = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

for chunk in x.chunks(ofCount: 3) {
    print(chunk)
}

// Prints [0, 1, 2]
// Prints [3, 4, 5]
// Prints [6, 7, 8]
// Prints [9]
```

if a collection isn't evenly divisble, the last element will contain the remainder.  These are subsequences of the base collection, making them cheap to create.

Sometimes you want to chunk a collection into runs of "like elements"

```swift
import Algorithms

let x = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

for (isPrime, chunk) in x.chunked(on: \.isPrime) {
    print((isPrime, chunk))
}

// Prints (false, [0, 1])
// Prints (true, [2, 3])
// Prints (false, [4])
// Prints (true, [5])
```

This iterates on chunks of elements that return the **same** value for `isPrime`.  For convenience, this vends a tuple of value and element.

Chunked also handles the case of doing work if the previous/next elements differ.

*Times in the transcript whenever more than hour has passed between messages.*

```swift
import Algorithms

extension Message {
    func makeMessageParts() -> [TranscriptItem]
}

transcript = Array(
    messages
        .lazy
        .flatMap { $0.makeMessageParts() }
        .chunked { $1.date.timeIntervalSince($0.date) < 60 * 60 }
        .joined { DateItem(date: $1.first!.date) }
)
```
In this situation, we are passed 2 elements and determine if they belong in the same group or not.

Algorithms package has a new joined that allows you to compute separator from prev/next chunks.

All computed on-demand with `lazy`.   

Note that laziness is not a silver bullet.  When you're only iterating a sequence a single time, computing on-demand can save work and avoid allocations.

In cases like this, still use lazy.  But as a last step, save your work into an array.

Chaining together algorithms makes clearer, faster, and more correct.  Next time you're doing a loop, consider what else you could do.  Look at documentations or guides.

# Swift Collections
stdlib implements 3 major structures.
* Array
* Set
* Dictionary

They all implement CoW value semantics providing efficient in-place mutations while also ensuring that collection values are safe to pass around without mutations.

However, there are so many more datastructures out there!  It would be useful to have a larger collection.

github.com/apple/swift-collections

New collection types for inclusion in swift stdlib.  By importing collections, we get access to additional types.

* Dequeue
* OrderedSet
* OrderedDictionary

Variants of the same theme.  That said, these new times aren't replacements for the existing ones.  They're complementary.

For some usecases, new types are the better fit, in other cases not.  Learn how these differ from existing types.

## Deque

Queues pop up everywhere that an arbitrary number of items need to be processed 1x1.
async tasks in an application.  Queues provide 2 major operations:
* Push items to back of queue
* pop elements off the front

DEQ makes these symmetric.  Efficiently access both sides.

"Deck", like a deck of cards.  Similar to array type and implements many protocols, e.g. `ExpressibleByArrayLiteral`.  `RandomAccessCollection`, etc.

`RangeReplaceableCollection`.  

```swift
var items: Deque = ["D", "E", "f"]
print(items[1])  // "E"
items[2] = "F"
items.insert(contentsOf: ["A", "B", "C"], at: 0)
print(items[1])  // "B"
```

If you use an array to store items, `insert` at the front needs to realloc and copy.  Arrays keep elements in contiguous buffer.

If the array is large, this makes prepending new elements relatively expensive.  

Dequeue wraps storage buffer around its boundaries.  Indices are still offset from the start of the collection, prepend new elements without moving existing ones.  Storage buffer
[DEF....ABC]

Dequeue needs to do work between logical indices and storage positions.  But accessing elements is still quite efficient.  And because we don't slide members, can perform this quite quickly.  Switching to the right ds can make all the difference.  

When removing subrange, can close the gap by moving the preceding elements, decreasing the elements that have to be moved.  This isn't as drastic as an improvement, but it makes things 2x on average.


# OrderedSet
Unordered sets are effectively random.  Ordered sets are ordered.

Equality must have the same order.  If you just need to know if they contain the same elements in any order, than use the `unordered` 
```swift
var items: OrderedSet = ["E", "D", "C", "B", "A"]
items[3]  // "B"
items.append("F")         // (inserted: true, index: 5)
items.insert("B", at: 1)  // (inserted: false, index: 3)
items.remove("E")
items.sort()
items.shuffle()
```

Note that they can't conform to `SetAlgebra` because that requires that the order does not matter.  `unordered` does though.

OrderedSet stores elements in a regular array.  Still uses hashtable implementation, but in this case the table only stores integer indices.  The range is bound by the size of the hashtable, so we can pack into few bits.

Lookup performance is competitive to a standard set.  Array is slow because it compares each element.

Appending an element to an ordered set is similar to inserting into standard set.  This needs to hash the new item, and a check if the element exists, so it's more complex than appending to array, but it still takes constant time.

However, because it needs to quickly look up existing elements and append new ones, it cannot efficiently implement inserting an item at the front or middle.  Need to renumber indices in the hashtable.  Therefore removals/insertions turn into operations with linear complexity making them slower than usual set.

# Ordered dictionaries
```swift
var dict: OrderedDictionary = [2: "two", 1: "one", 0: "zero"]

print(dict[1])  // Optional("one")
print(dict)     // [2: "two", 1: "one", 0: "zero"]

dict[3] = "three"
dict[1] = nil
print(dict)     // [2: "two", 0: "zero", 3: "three"]
```

follows the order in which the keys were originally inserted.  We can assign the value to a new key to append.  We can remove by assigning nil to an existing key.

Ordered dictionaries use array-like integer indices but this introduces an interesting issue.  In our dictionary, the index subscript conflicts with the key subscript.  When we subscript 0, do we mean key 0, or the key-value-pair at offset 0?

We think that the key-based subscript is the primary operation for a dictionary type, so to prevent the ambiguity it always means key.

Ordered dictionaries doesn't provide an indexing subscirpt at all.  This means that OrderedDictionary can't be Collection, because Collection requires an indexing subscript.

Therefore, OrderedDictionary only conforms to Sequence.  HOwever, for cases where Collection is desireable, it provides `elements` view.  This has an indexing subscript with key-value pair.

Implementation: regular dictionary uses keys and values as separate hashtables.  Ordered dictionary uses single compressed hashtable and two parallel arrays instead.  This can save even more space than orderedsets.

By using these constructs, we can boost the performance of our apps or reduce memory use, or express new constraints.



Other packages
* ArgumentParser
* Numerics
* SystemAtomics
* Collections
* Algorithms

