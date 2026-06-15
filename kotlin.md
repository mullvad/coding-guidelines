# Kotlin coding guidelines

These guidelines apply to all Kotlin code we write, regardless of target platform.
Android-specific conventions do not belong in this document.

## Coding guidelines

We follow the official [Kotlin coding conventions].

## Linting and formatting

All code is formatted with [ktfmt] using `kotlinLangStyle` and the [editorconfig] in the
[mullvadvpn-app repository] is a reference configuration. We'll typically use additional
linters such as [detekt], but that can differ on a project basis.

Also see the [main page] for general file format standards to follow.

## CI

All repositories containing Kotlin should have a CI pipeline that fails on compilation errors,
formatter diffs, detekt findings, and test failures before code is merged to the main branch.

[Kotlin coding conventions]: https://kotlinlang.org/docs/coding-conventions.html
[ktfmt]: https://github.com/facebook/ktfmt
[detekt]: https://detekt.dev/
[mullvadvpn-app repository]: https://github.com/mullvad/mullvadvpn-app
[detekt configuration]: https://github.com/mullvad/mullvadvpn-app/blob/main/android/config/detekt.yml
[editorconfig]: https://github.com/mullvad/mullvadvpn-app/blob/main/android/.editorconfig
[main page]: ./README.md
