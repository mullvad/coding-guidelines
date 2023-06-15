# Swift coding guidelines

## Coding guidelines
We've chosen to stand on the shoulders of giants, and follow [Google's Swift styleguide] and [Swift API guidelines],
with 2 additional rules:

1. We should try and use whatever language/syntax features the latest stable Swift compiler allows for.

2. Constructors should be as side-effect free as possible to aid with testing. Specifically, we should not do the
   following in a constructor:
   - Heavy I/O
   - Starting timers
   - Scheduling work

  Instead, an object should only start to operate after an explicit `start()` invocation.

## Formatting

All code should be formatted with the latest release of `swiftformat`. The [swiftformat configuration] in the
mullvadvpn-app repository is the reference configuration.

[swiftformat configuration]: https://github.com/mullvad/mullvadvpn-app/blob/main/ios/.swiftformat
[Google's Swift styleguide]: https://google.github.io/swift/
[Swift API guidelines]: https://www.swift.org/documentation/api-design-guidelines/

