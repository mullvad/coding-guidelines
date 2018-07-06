# Mullvad coding guidelines

These documents aim to be the source of information for how we architect, design and format
program code in various languages and tools.

## Formatting

See the [EditorConfig settings] in the main Mullvad VPN app repo for file format standards to
follow.

In general this is what to follow:

* File encoding should be UTF-8 and not start with [BOM] bytes.
* Files should end with newlines.
* Don't have trailing spaces on any lines.
* Indent only with spaces. Except for languages where it is customary to use tabs.
* Only commit LF (`\n`) line endings to git. If you configure git to convert to and from CRLF
  (`\r\n`) in the working directory that is fine, but only commit LF line endings.

## Language specific guidelines

* [Rust](rust.md)


## Git

### Commit messages

Here is a generally nice guide to how to write good commit messages:
https://chris.beams.io/posts/git-commit/

So sum it up the most important parts are:

* Capitalize the subject line.
* Do not end the subject line with a period.
* Use the imperative mood in the subject line.

### History

Prefer to let the history represent how the project evolves rather than exactly how the developer
experimented to get there.

So for a given feature branch, each line of code should usually only have to be changed in one
commit. The developer might experiments back and fourth with different values or correct
spelling errors etc. But the end result of the work should be the addition of the correct values
and the correctly spelled versions of what was added. When looking back in history, the intermediate
"backups" of the code the developer did does not add much value.

Basically instead of this history:
```
| * ae83f10 (HEAD -> add-feature) Fix malformed linker argument
| * 8888af9 Lower TCP timeout to 30 seconds
| * c8e7e68 Add linker argument to CI config
| * 281a8e8 Add a timeout to TCP connections
|/
* 4d3ca65 (origin/master, master) Merge branch 'previous-feature'
```
Consider squashing it to something like this before merge:
```
| * 6e84f10 (HEAD -> add-feature) Add linker argument to CI config
| * 381a8e9 Add a 30 second timeout to TCP connections
|/
* 4d3ca65 (origin/master, master) Merge branch 'previous-feature'
```


## Changelogs

Keep a changelog for every project/library according to the guidelines and rules set up at
[keep a changelog]. For inspiration see how we manage it in the main Mullvad VPN app repo
[here](https://github.com/mullvad/mullvadvpn-app/blob/master/CHANGELOG.md).

* Keep the categories for every release in the same order as on the website:
  * _Added_ - for new features.
  * _Changed_ - for changes in existing functionality.
  * _Deprecated_ - for soon-to-be removed features.
  * _Removed_ - for now removed features.
  * _Fixed_ - for any bug fixes.
  * _Security_ - in case of vulnerabilities.
* Add new items to the end of the list of changes under the appropriate category. This way they
  automatically end up in chronological order within each category.
* Use the imperative mood in the entries. This is to stay consistent within the changelog itself
  and to stay consistent with how git commit messages are written. In other words use "increase",
  not "increased". Use "add" not "added". The exception to this are the titles for the categories.
* Make sure only items that changed since the last release are present. For example:
  ```
  ### Changed
  - Increase connection timeout to 30 seconds.
  ...
  - Increase timeout for connections from 30 to 45 seconds.
  ```
  Would not be very intuitive for a reader. Because comparing the previous release with this one
  all that happened is that the timeout increased to 45 seconds. That it was set to 30 at some
  point between does not matter. When multiple changes happen to the same thing within a single
  release like this, join the items:
  ```
  ### Changed
  - Increase connection timeout to 45 seconds.
  ```
* If the project has many changes that are platform specific, prefer to add sub-headings for the
  platform under the relevant categories over writing "... on \<platform\>" in many entries.
  Example:
  ```
  ## Fixed
  - Malformed log entry on Linux.
  - Wrong string delimiter in parser.
  - Invalid Linux syscall number.
  ```
  Then instead use:
  ```
  ## Fixed
  - Wrong string delimiter in parser.

  ### Linux
  - Malformed log entry.
  - Invalid syscall number.
  ```


[EditorConfig settings]: https://github.com/mullvad/mullvadvpn-app/blob/master/.editorconfig
[BOM]: https://en.wikipedia.org/wiki/Byte_order_mark
[keep a changelog]: https://keepachangelog.com/en/1.0.0/
