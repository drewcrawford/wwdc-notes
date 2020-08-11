#pencil 

[[What's new in pencilkit]]

# data model access
Access internals of PKDrawing
Synthesize new drawings
Inspect user-created drawings

## demo

## Inside `PKDrawing`
Each stroke represents an individual line that the user drew.

For strikes, the primary feature is the path.  You also have the ink, a transform, and masks.  Another usef property is `renderBounds`, which encompasses the entireity of the stroke.

Uniform cubic B-spline
`PKStrokePoints` control points

It means that the contents of the path are the control points for the b-spline.  The resulting points are not actually on the curve.

`path.interpolatedPoints(strideBy: .distance(50))` The last point is always generated, regardless of the stride.  Can stride by distance, time, or parametric value.

Non-uniform stepping
`parametricValue(parametricValue, offsetby:)` forwards-or-back.

## Masks
Cut a stroke into pieces
Each piece is a separate stroke

We want to use `maskedPathRanges` property, because stroke "paths" are not masked.

Strokes can have 0 or more masked ranges.