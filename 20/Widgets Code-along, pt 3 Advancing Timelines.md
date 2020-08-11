# Widgets Code-along, pt 3 Advancing Timelines
URL sessions
Making network requests is one of the fundamental ways that a widget can get data.
Widgets have no app delegate.  So call `.onBackgroundURLSessionEvents` to get them.
# Using Link
Embed link able content in `Link`

# Widget bundles
By definition, there can only be one `@main` per process.  But we can make a `WidgetBundle` `@main`.  This is how we support multiple widgets.
# Dynamic configuration
What if we don’t know all the options up-front?  We can provide a dynamic list of options via an Intent extension.
Make sure your starting point is “None” in the add target wizard
Our config option is now a custom type instead of an `enum`.

[[Design Great Widgets]]
[[Build SwiftUI Views for Widgets]]
