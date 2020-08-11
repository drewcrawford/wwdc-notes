Original assessment on iPad

Prevent app switching
Disable home screen
etc.

Bring this to the mac as well.  

# Automatic assessment configuration
Not identical to iPad.  Preserve same testing environment on each platform.

## Assessment mode lifecycle

1.  Inactive
2.  Activating
3.  Active
4.  Deactivating
5.  Inactive

### Errors

1.  Activating may fail -> return to inactive
2.  Error could occur while the exam is running.  We do not tear down assessment mode, but we call it 'interrupted'.  You then deactivate on your own.

## Code

## Logic vs effects
Deciding to do a thing vs doing a thing

Deciding to enter assessment mode vs entering assessment mode

Your app owns the logic.  But assessment can make debuggin/testing very difficult.

## Protocol-oriented programming

Consider using protocols for mocks.

[[Protocol-oriented programming in Swift]]
[[Protocol and Value Oriented Programming in UIKit Apps]]

## AAC
Available on iOS and iPadOS, and mac.

Now configureable.

autocorrect,spellcheck, keyboard, shortcuts, activity continuation, etc.

Also available on catalyst.
