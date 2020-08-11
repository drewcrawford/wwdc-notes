#swift 
# Swift numerics
Open-source Swift package
github.com/apple/swift-numerics

Building blocks for generic numerical computing in Swift

Basically we want to make code generic across different float widths.

We need to constrain `T: Real`.

## The `Real` protocol

Part of Swift Numerics package
Provides generic access to "standard fp capabilities".

`protocol Real: FloatingPoint, AlgebraicField, RealFunctions { ... }`

### stdlib protocols

* `AdditiveArithmetic`
* `Numeric`
* `SignedNumeric`
* `FloatingPoint`
* `BinaryFloatingPoint`
* `Float`,`Double`,`Float80`

#### `AdditiveArithmetic`

implements `+`, `-`,`.zero`

#### `SignedNumeric`
Also supports multiplication

#### `FloatingPoint`

comparison, `exponent`, `significand`,some constants

#### `AlgebraicField`

`:SignedNumeric`

adds division, `receiprocal`

#### `ElementaryFunctions`

`:AdditiveARithmetic`

Adds a large collection of common fp functions, `sin`,`cos`,`tan`,etc.

#### `RealFunctions`
`:ElementaryFunctions`
Less-used functions, `atan2`,`cos(piTimes:)`,`sin(piTimes:)`,etc.

#### `Real`
combines all this stuff.  


## The `Complex` type

Part of Swift Numerics
Generic over any `Real` type, so you can use various fp implementations

layout-compatible with C and C++ complex types

* `Complex<Double>` (Swift)
* `_Complex double` (C)
* `std::complex<double>` (C++)

### One caveat
Treatment of complex infinity and Nan is different than C or C++
* Can affect code ported from C or C++
* Simpler
* Significant performance gains for multiplication and division (1.3x faster for multiply, division 3.8x faster, if by constant, 10x)

## Numerics is a work in progress

# Float16

Float16 is IEEE&54 format new to Swift
Already available on iOS, iPadOS< tvOS< watchOS
Working with intel to land on x86

A normal floatingpoint-type
* Conforms to `BinaryFloatingPoint` and `SIMDScalar`
* Conforms to `Real` from Swift Numerics
* supports standard floating-point functions

## Tradeoffs

Pros
* Significant performance gains
* Interoperates with C/ObjC `__fp16` type

Cons
* Low-precision, small range



|                            | Double             | Float        | Float16     |
|----------------------------|--------------------|--------------|-------------|
| Size                       | 8 bytes            | 4 bytes      | 2 bytes     |
| `.leastNonzeroMagnitude`   | 4.9 * 10^-324      | 1.4 * 10^-45 | 5.9 * 10^-8 |
| `.greatestFiniteMagnitude` | 1.8 * 10^308       | 3.4 * 10^38  | 65504       |
| `1.nextUp`                 | 1.0000000000000002 | 1.000001     | 1.001       |

## Hardware support

Supported (and preferred) by apple gpus
Supported by apple's CPUs starting with A11 bionic
* Scalar performance is identical to `Float` or `Double`
* SIMD performance is 2x `Float`

`Float16` is simulated (using `Float` on older HW)

