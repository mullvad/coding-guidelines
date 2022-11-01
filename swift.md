# Swift coding guidelines

## Formatting

All code should be formatted with the latest release of SwiftFormat.
The reference [SwiftFormat configuration] can be found in the mullvadvpn-app repository.

## General design guidelines

### Prefer whole classes to extension galore

Do not abuse extensions for code organization purposes within the same unit. Extensions do not
support stored properties and instance variables will have to be added into the class or struct
definition anyway.


```swift
// BAD
struct Foo {
    var value: Int?
    var otherValue: Int?
}

extension Foo {
    func printValue() {
        /* implementation */
    }
}

extension Foo {
    func printOtherValue() {
        /* implementation */
    }
}

extension Foo: CustomDebugStringConvertible {
    var debugDescription: String {
        /* implementation */
    }
}

// GOOD
struct Foo: CustomDebugStringConvertible {
    var value: Int?
    var otherValue: Int?

    func printValue() {
        /* implementation */
    }

    func printOtherValue() {
        /* implementation */
    }

    var debugDescription: String {
        /* implementation */
    }
}
```

### Prefer explicit `return`

Prefer explicit `return` in closures and functions except when used in single line closures with
positional arguments.

```swift
let foo: Int?

// GOOD
foo.map { $0 }

// BAD
func bar() -> Int {
    0
}

// GOOD
func bar() -> Int {
    return 0
}
```

### Prefer named arguments in complex closures

```swift
let rawPortRanges: [[UInt16]] = [/* initialization */]

// BAD
rawPortRanges.compactMap {
    guard $0.count == 2 else { return nil }

    let startPort = $0[0]
    let endPort = $0[1]

    if startPort <= endPort {
        return (startPort ... endPort)
    } else {
        return nil
    }
}

// GOOD
rawPortRanges.compactMap { inputRange in
    guard inputRange.count == 2 else { return nil }

    let startPort = inputRange[0]
    let endPort = inputRange[1]

    if startPort <= endPort {
        return (startPort ... endPort)
    } else {
        return nil
    }
}
```

### Prefer explicit access level on extension members

```swift
// BAD
public extension Foo {
    struct Bar {
        public var baz: Int?
    }

    func qux() {}
}

// GOOD
extension Foo {
    public struct Bar {
        public var baz: Int?
    }

    public func qux() {}
}
```

### Prefer implicit `self`

Use implicit `self` unless it's necessary in order to distinguish between local variable and
instance member.

```swift
// BAD
func foo(bar: Int) {
    self.bar = bar
    self.baz = 42
}

// GOOD
func foo(bar: Int) {
    self.bar = bar
    baz = 42
}
```

<!-- # Links -->

[SwiftFormat configuration]: https://github.com/mullvad/mullvadvpn-app/blob/master/ios/.swiftformat
