# Publishing to crates.io

This document outlines best practices and rules for publishing crates to [crates.io].
For guidelines on writing Rust, see: [rust](./rust.md).

To be added to the [`appteam-crates.io`] github team, an employee must have read
and understood this document.

Crates owned or maintained by Mullvad VPN's app team should in general follow common
[crates.io best practices] and have clean and correct metadata and documentation.
On top of that, make sure to follow the below guidelines both when publishing
a new crate, and when updating an existing one.

## Verify integrity of code to be published

Make sure the code you have checked out can be trusted by verifying commit signatures
all the way from the last release tag (or first commit) up to the commit you want to publish.

Publishing a crate should be done in a clean working directory,
so exactly what is represented in git is published. Never publish with `--allow-dirty`.

Since cargo prefers including extra files rather than excluding them,
sanity check what you are about to publish with `cargo package --list` first.

## MSRV (Minimum Supported Rust Version)

Bumping the MSRV should not be treated as a breaking change in the versioning. The alternative
would be catastrophic. There would be many major versions of most crates,
and the ecosystem would be immensely fractured.

The MSRV should be specified in the [`rust-version`] metadata field. This helps cargo give better
error messages and pick suitable versions of the crate. This mitigates some problems related to
publishing MSRV bumps as non-breaking changes.

## Readme

The git repository should have a readme with a friendly, understandable and up to date
introduction to what the crate does.

The `readme` field does not need to be set in `Cargo.toml`, since cargo by default picks
up `README.md` anyway.

## Changelog

All crates owned by Mullvad should maintain a changelog on the
["Keep a Changelog"] format in a file named `CHANGELOG.md` in the git repository.

## Vulnerability reporting

Github repositories for crates owned by Mullvad should have Github's
"Private vulnerability reporting" enabled.
This allows developers and security researchers to submit vulnerability reports without
them becoming public immediately.

The git repository should have a [`SECURITY.md`] file explaining how to report security
vulnerabilities. A good default can be found in [SECURITY.md.example](./SECURITY.md.example).

## Authors

The `authors` metadata field in `Cargo.toml` should be set to `["Mullvad VPN"]` for crates
developed fully in house. Individuals should generally not be listed.

If Mullvad takes over or forks an existing crate, `"Mullvad VPN"` should be added
to the `authors`, but the previous authors should be kept intact.

## License

Always both set the `license` field in `Cargo.toml` and commit a `LICENSE`
(or `LICENSE-<name of license>`) file to the repository.

The crate license should in general be `"MIT OR Apache-2.0"`, but other licenses can be used
in special cases, or when required due to forking an existing library.

If the crate is used in the [Mullvad VPN app], the library must of course be compatible with
the license of the app (`GPL-3.0-only`).

## Discoverability metadata

All the fields `description`, `categories` and `keywords` should be set with care, to improve
discoverability and give a professional impression.

## Linking metadata

The `repository` field should be set to the public URL of the repository where the crate
is developed.

The `homepage` field should not be set, unless there is a dedicated website for the crate,
that is not the repository.

## Git tag

After publishing, create a git tag on the commit used to publish the new version.
The tag should be of the format `v$VERSION`, for example `v1.0.3` for crate
version `1.0.3`.

The git tag should be PGP signed by the publisher.

## Ownership and publishing rights

The github team [`appteam-crates.io`] should be added as owner on crates.io,
to allow the team to publish updates.

However, *teams* have [limited permissions] on crates.io and cannot add or remove
other owners. So we need at least one user account to be owner as well. This should
be a tech lead or well-trusted developer.

## Yanking

The [cargo documentation on yanking] explains it well:

> Crates should only be yanked in exceptional circumstances, for example, an accidental
> publish, an unintentional SemVer breakages, or a significantly
> broken and unusable crate.

For most bugs and issues, yanking should not be used. Instead a fixed semver compatible version
should be published, and maybe a [RustSec] advisory should be created, if relevant.

[cargo documentation on yanking]: https://doc.rust-lang.org/cargo/commands/cargo-yank.html#when-to-yank
[RustSec]: https://rustsec.org/
[crates.io best practices]: https://doc.rust-lang.org/cargo/reference/publishing.html
[limited permissions]: https://doc.rust-lang.org/cargo/reference/publishing.html#cargo-owner
[crates.io]: https://crates.io
[`appteam-crates.io`]: https://crates.io/teams/github:mullvad:appteam-crates-io
[Mullvad VPN app]: https://github.com/mullvad/mullvadvpn-app/
[`rust-version`]: https://doc.rust-lang.org/cargo/reference/rust-version.html
["Keep a Changelog"]: https://keepachangelog.com
[`SECURITY.md`]: https://docs.github.com/en/code-security/getting-started/adding-a-security-policy-to-your-repository