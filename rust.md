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
