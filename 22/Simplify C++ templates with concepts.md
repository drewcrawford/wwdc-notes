#cpp

New c++20 features supported in xcode 14

# Write generic code
## Templates enable generic algorithms
`isOdd(int) -> bool`

If I pass a uint64_t?  Concrete function does not behave correctly with 64-bit.  Gets truncated.  To fix this, I can make it a function template.

`isOdd<T>(T) -> bool`.  Compiler generates specialization that works correctly.  I don't have to write two versions that operate on two different types.  Use C+= tempaltes to write generic functions, and container classes as well.

```cpp
template<class T>
bool isOdd(T x) {
    return (x % 2) == 1;
}
```

Note however that I can call this with `1.1`.  

Here, template requirements are not explicit.  Only a doc comment that states I must call this with integer types.  Prior to c++20, there was not a good way to specify template requirements.  Documentation comments, parameter names, complicated enable_if checks, etc.

Now we have *concepts*.  Validate template requirements in your generic c++ code.

```cpp
#include<concepts>
template<std::integral T>
bool isOdd(T x) {
    return (x % 2) == 1;
}
```

Compiler can provide clearer diagnostic for passing `1.1`.  Turns out, `1.1` is a double which does not satisfy the concept.


# Use concepts
Reusable core concepts provided by the standard library

```cpp
#include <concepts>

//core concepts
static_assert(std::floating_point<double>);
static_assert(std::convertible_to<long,int>);
static_assert(std::move_constructible<std::vector<int>>);

//comparison concepts
static_assert(std::equality_comparable<std::string>);

```

many more concepts, object concepts, invocable concepts...


```cpp
template<std::integral T>
bool isOdd(const T &x) {}
```

constrain to multiple concepts

```cpp
template<class T>
requires std::equality_comparable<T> && std::default_constructible<T>
bool isDefaultValue(const T &value) {
	return value == T();
}
```

Will only be specialized with supported types.

## Review
* Use concepts to constrain templates
* Use concepts library to test core type behavior
* Use `requires` clause to test multiple concepts


# Create concepts
## Identify behavioral requirements
various shapes.  Circle.  
For each pixel in the rendered image, compute distance to surface.

circle => - distance inside the circle, + distance inside the circle.

We can make a crescent by subtracting one shape from another.

Will use a function template instead for performance reasons.  Avoid virtual call overhead.

```cpp
template<class T>
Color computePixelColor(const T &shape, float x, float y) {
	float dist = shape.getDistanceFrom(x,y);
	if dist <= 0.0) return Color::white();
	return Color::trasnaprent();
}
```

Verify that shapes can be filled in correctly.  Any shape type, e.g. crescent, circle, matching type.  Even though a template works well, I would like to use concepts to constrain the type that can be passed to the function.  Clearer diagnostics when a mismatch occurs.

Also allows me to add more overloads of the function.

We will create a concept for `Shape`.  Identify requirements.  We only call `shape.getDistanceFrom`.

You can use `requires` to test if type behaves in a specific manner.

```cpp
template<class T>
concept Shape = requires (const T &shape) {
	//this will not be executed, only validated at compile time.
	shape.getDistanceFrom(0.0f,0.0f)
};
```

example validating return

```cpp
template<class T>
concept Shape = requires (const T &shape) {
	//this will not be executed, only validated at compile time.
	{shape.getDistanceFrom(0.0f,0.0f)} -> std::same_as<float>;
};
```

now we can say `template<Shape T>`.

Suppose some shapes have computePixelColor to find a color at a given pixel.  We want to overload to create multipel variants.  Each variant must be constrained using different concepts.

```cpp
template<class T>
concept GradientShape = Shape<T> && requires (const T &shape) {
	{ shape.getGradientColor(0.0f,0.0f) } -> std::same_as<Color>;
};
```

This ensures that a class that satisfies `GradientShape` also satisfies `Shape`.

This template only works with shape classes with a gradient.  It is constrained by `GradientShape` concept.

Try rendering a circle with a gradient.  here I'm rendering a graidentcircle.

Compiler picks the overload that is constrained with the `GradientShape` concept as it is more specific than the first overload.  

## Recap
* Create concepts by identifying requirements in generic code
* Use `requires` expression to validate behavior of types
* Use concepts to provide overloaded variants for generic functions


# Compile-time evaluation
## Benefits of compile-time evaluation
Reduce the runtime cost of initialization for variables
Verify your constants before running your code

## ex
```cpp
Color Color::fromHexCode(std::string::string_view hexCode) { ... }

const std::array<Color, 3> colorPalette = {
	Color::fromHexCode("#FF5733"),
	Color::fromHexCode("#FF5733"),
	Color::fromHexCode("#FF5733"),
};
```

Each color is initialized with athe string literal, etc.  Currently, `fromHexCode` needs to parse 3 string literals.  Complicated constant intiialization can have a measurable impact on the launch time of my app.  I can use compile-time code evaluation to ensure that this array is initialized with constant color values instead.

`constexpr` *enables* compile-time code evaluation.  I must add it in order to *ensure* that it is constant.

Make your functions `constexpr` when you want them to be evaluated at compile-time.
```cpp
constexpr Color Color::fromHexCode(std::string::string_view hexCode) { ... }

constexpr const std::array<Color, 3> colorPalette = {
	Color::fromHexCode("#FF5733"),
	Color::fromHexCode("#FF5733"),
	Color::fromHexCode("#FF5733"),
};
```

if statements, primitive operations, etc.  All can be evaluated at copmile-time.  This function makes calls to `hexToInt`.  I have already annotated `hexToInt` with constexpr, so calls can be evaluated at compile-time.

Since all calls from `fromHexCode` are evaluated at copmile time, it now contains a constant array literal.  Which means my appd oesn't have to parse this at runtime.

Reduces the amount of work this C++ library has to do at startup.  make your c++ variables `conexpr` when you want to ensure they are initialized with constant values.

Xcode 14 has substantially improved stdlib support for compile-time evaluation.  We support several different stdlib types and algorithms.

In addition, xcode 14 has greatly improved its c++20 standard support.  All features shown here can be used in c++20 mode.

Switch to c++20 mode today if you haven't done so already.  Use the c++ language dialect setting in your xcode project to upgrade.  Switching will let you use concepts in your code.  C++20 does not require a minimum depmloyment target.  Ship your code in same OS verison you're currently targeting.  Try c++20 today.

Thanks, enjoy

* https://en.cppreference.com/w/cpp/concepts
* https://en.cppreference.com/w/cpp/language/constraints
