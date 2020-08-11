#swiftui 

Previews benefit your app as a whole.  By structuring your app to make it more previewable, you make it more understandable, etc.

We have 4 ways to structure your app (topics?)

* previewing multiple files
* App life cycle
* Sample data
* Structuring your views

# Example app
# Multiple files
## pinning
We can pin preview, so that when we edit some small control, we can pin the containers so we see the control in context.

Recommended way to customize for darkmode is with an asset in the asset catalog.  Pinning control while we do that.

"Extra previews" file.  Can bring in previews for a specific situation like accessibility sizes.

# App life cycle
Apps do some kind of initialization at startup.  Work is expensive.  We want to avoid this during previews.

It's important to think about defining dependencies.  

Hold play button and choose "debug preview" from the menu!  This will enable breakpoints and let us debug preview issues.  In this demo, we refer to the debug gauge.

Use `@StateObject` on our model.  This will a) only initialize the state once, b) allow swiftui to react to changes?

For *previews*, `@StateObject` indicates we don't initialize (???).  Great opportunity to do work only when we need to run our app.  

[[App essentials in SwiftUI]]

If we're not initializing our data models, how do we get data into our previews?  2 parts.

# Sample data

During development, I want photo examples.
Asset catalog with various images.  I don't want to ship these in my app.  "Development Assets" phase.  Xcode only includes these in "development version of your app".  I guess debug config?

Can also add swift files to this?  I guess this avoids `#if DEBUG` guards?

# structing views
"Rich datatype".  Can be coredata, cloudkit, etc.  At the end of the day, our users interact with our data with simple datatypes.  String in textfield, boolean in toggle.

In-between are our views.  This translates the rich datatype into the simple datatype.  Where in our view hierarchy should we do this translation?

The sooner we do it, the more reusable, testable, and previewable our app is.

We want to make it easy to add previews, so we can test our app in various configurations.  4 examples.

## Collaborator cell (immutable inputs)
Identify the minimum amount of data that you need to pass from your model into your view.  Maybe you odn't need the whole model.

e.g., `name`, `image`, `connectonStatus` as separate types.  Now can preview in various configs without making fake models etc.

When you can, make the inputs to your view simple, immutable datatypes.
## Inspector (mutable inputs)
Pass in simple datatypes that are read-write.

We want to avoid potentially bringing in CloudKit into our previews.

Evidently when we need a binding we can pass in `.constant(BindingType())` which is some way of saying the value cannot be changed?

Another approach is to define an intermediate view (container) that hosts the value, and take the binding from that.

Maybe give our view struct a closure ivar, to let "someone else" decide how a complex edit in the models is performed in response to a view action (e.g. a button tap).  This lets us supply a different version in the xcode previews.

On-device previews.  As you make changes, they're seamlessly pushed to devices.  The first time you use, an icon appears that says "xcode previews".  You can get back to the last preview you were looking at, even if you were disconnected from xcode.  Easier to do hallway usability testing.

## Collage editor (generic inputs)
Use generics to pass data into the main editor.

Once again, we have a CKRecord in here that's annoying to deal with.  We could do what we did previously, and use bindings to pass in the data.    Problem here is the image slot is a rich type?

In this example, we define a protocol as an abstraction.  Here we create a protocol with an associatedtype for the slot data, which is also a protocol abstraction.  Now we can define a design-time implementation.

Then our view is generic over this protocol, so we can supply a different implementation at design-time.  Second, we can be generic over an image picker view, so we can apply something else at design-time than a real photo picker.


## Sync status (environment inputs)
In SwiftUI, you need a value for the environment higher in the hierarchy.  When creating previews, you also need to provide a value for each preview.

## unnamed example
Putting the whole app in the preview.
Edits reflect on all devices at the same time.

# Recap
* How to preview multiple files at once so you can edit a view in larger context
* relationship between previews and swiftui app lifecycle, only initializing data when you need it
* where to define sample data and how to keep it out of release product by using development assets
* structuring your views, by making them more previewable.

# Big idea
The investments to make your app more previewable benefit your app as a whole.  A previewable app is more understandable, testable, and maintainable.

