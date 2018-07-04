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


[EditorConfig settings]: https://github.com/mullvad/mullvadvpn-app/blob/master/.editorconfig
[BOM]: https://en.wikipedia.org/wiki/Byte_order_mark
