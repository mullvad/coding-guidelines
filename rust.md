# Rust coding guidelines

## Formatting

All code should be formatted with the latest release of rustfmt. The [rustfmt configuration] in the
mullvadvpn-app repository is the reference configuration, in order to not have to repeat it here.

Also see the [main page] for general file format standards to follow.

## Code/API design

Follow the standard [Rust API guidelines] as much as possible.

Run clippy on all code and follow the recommendations it prints in all places where it is reasonable
to follow.

There is an [unofficial guide to design patterns in Rust] that it can be worth looking at.

There is an [unofficial book about Rust macro design] that can provide good design patterns for
macros.

## Imports and modules

* Avoid wildcard imports. They generally make it harder to trace where a type comes from.
  The exceptions are:
  * Third party crate preludes - But prefer importing each item unless there are too many.
  * Importing all variants of an enum in a small scope in order to match over all the variants
    without prefixing every branch with the enum name. But prefer prefixing with enum name
    unless doing so is very inconvenient.
  * Importing `super::*` in a local test module.

* Avoid importing standalone functions and macros directly into the namespace in most cases.
  The module prefix can often both add valuable context and clearly show it's a
  function defined elsewhere. So prefer `log::error!(...)` over `use log::error; error!(...)`.

### Other/custom design guidelines

* Crate names should be kebab-case
* Don't have debug formatted (`{:?}`) strings in messages shown to a user, except in `debug!` and
  `trace!` log entries if needed
* When converting a path to a string intended for user viewing, use `Path::display()`
* Desired argument order for assertions is `assert_eq!(actual, expected)`

### Generic Rust type improvement tips

* Avoid unnecessary conversions that introduce either extra possible error states or loss of data/
  precision if possible.
  ```rust
  // DONT: Will return an error if the content is not correct UTF-8.
  let path = env::var("A_PATH_VARIABLE").map(PathBuf::from);

  // DO: Does not enforce any encoding on the path.
  let path = env::var_os("A_PATH_VARIABLE").map(PathBuf::from);
  ```
* The appropriate borrowed versions of `Vec<T>`, `String` and `PathBuf` are `&[T]`, `&str` and
  `&Path`. Not `&Vec<T>`, `&String` and `&PathBuf`.
  ```rust
  // DONT
  fn sum(data: &Vec<u32>) -> u32 { ... }
  fn read_file(path: &PathBuf) -> Vec<u8> { ... }

  // DO
  fn sum(data: &[u32]) -> u32 { ... }
  fn read_file(path: &Path) -> Vec<u8> { ... }
  ```

## Converting between types

### Integers (avoid `as`)

Avoid converting between integer types using `as`. It can silently cause values to be truncated
and introduce bugs if types change or depend on external factors, such as the machine it's
built on. Consider the follwing code:

```rust
fn compute_index() -> u16 {...}

api_with_u32_indexes(compute_index() as u32);
```

This is sound code right now. But what if `compute_index` is changed to return `u64`s in the
future? Then the casting can truncate values and call `api_with_u32_indexes` with an invalid value.
The compiler won't say a thing. To catch this potential issue at compile time, always prefer:

```rust
api_with_u32_indexes(u32::from(compute_index()));
```

This will fail to compile if the return type of `compute_index` ever changes to something that
is not guaranteed to fit into a `u32`.

When dealing with integer conversion from wider to narrower types (`From` is not implemented),
use `TryFrom` to trigger an error if an overflow happens, instead of continuing the execution
with incorrect values. The exception is of course if truncating too large values is the desired
behavior.

### Prefer `From` over `Into`

When converting a `Foo` into a `Bar`, prefer `Bar::from(foo)` over `foo.into()`. The
former is easier to understand and less automatic inference is involved.
With `.into()` it can be really hard to figure out what the output type is going to be.

The exception is generic code where using `From` is not really possible. Such as:

```rust
fn connect(address: impl Into<SocketAddr>) {
    let address = address.into();
    ...
}
```

The same applies to `TryFrom` vs `TryInto`.

## Unsafe code

### Writing unsafe code (`unsafe fn`)

When you write an `unsafe fn` function it should always have a `# Safety` section in its
documentation stating what the user must do in order to not cause undefined behavior
when calling it.

### Calling unsafe code (`unsafe {...}`)

When you use an `unsafe {}` block, it means you as a caller must uphold some contract/
invariant that the code you call can't express in such a way that the compiler enforces it.
This should be documented with a `// SAFETY:` comment just above the unsafe block,
except for trivial cases.

Keep each `unsafe {}` block as small as possible without complicating the code too much.
This allows you to better document all the invariants you uphold individually.

Prefer:
```rust
/// SAFETY: The pointer points to a valid StoreHere struct.
unsafe { fetch_current(&mut store_here) };
/// SAFETY: All arguments are powers of two
unsafe { unsafe_alloc_special(4, 512) };
```

Over:
```rust
// SAFETY: The arguments are sane
unsafe {
    fetch_current(&mut store_here);
    unsafe_alloc_special(4, 512);
}
```


[rustfmt configuration]: https://github.com/mullvad/mullvadvpn-app/blob/master/rustfmt.toml
[main page]: ./README.md
[Rust API guidelines]: https://rust-lang-nursery.github.io/api-guidelines/
[unofficial guide to design patterns in Rust]: https://github.com/rust-unofficial/patterns
[unofficial book about Rust macro design]: https://danielkeep.github.io/tlborm/book/
