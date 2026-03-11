# Swift coding guidelines

## Coding guidelines
We've chosen to stand on the shoulders of giants, and follow [Google's Swift styleguide] and [Swift API guidelines],
with a few additional rules:

### Compiler
We should try and use whatever language/syntax features the latest stable Swift compiler allows for.

### Constructors 
Constructors should be as side-effect free as possible to aid with testing. Specifically, we should not do the following in a constructor:

  - Heavy I/O
  - Starting timers
  - Scheduling work

Instead, an object should only start to operate after an explicit `start()` invocation.

### Abbreviations

Abbreviations that are commonly written in all uppercase should be kept that way, with a few exceptions.

#### Enum cases

Abbreviations at the beginning of an enum case name should be in lowercase, but not those anywhere else:

  - `.lwo`
  - `.protocolLWO`

#### Variables and functions

Abbreviations at the beginning of a varible or function name should be in lowercase, but not those anywhere else:

  - `var/func lwoObfuscation`
  - `var/func applyLWO`

#### Types

Abbreviations in type names should always be uppcased:

  - `LWOObfuscator`
  - `ProtocolLWO`

## Linting and Formatting

All code should be linted and formatted with `swift format`. The [swiftformat configuration] in the
mullvadvpn-app repository is the reference configuration.

[swiftformat configuration]: https://github.com/mullvad/mullvadvpn-app/blob/main/ios/.swift-format
[Google's Swift styleguide]: https://google.github.io/swift/
[Swift API guidelines]: https://www.swift.org/documentation/api-design-guidelines/
