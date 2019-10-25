# Mullvad development guidelines

These documents aim to be the source of information for how we architect, design and format
program code in various languages and tools.


## Code review and PR process

As a remote team, a lot of work is done by individuals without too much interaction.
It is recommended to discuss solutions and problems early and often. Show proof of concept code to
evaluate if the chosen abstraction seems correct before going through with major refactoring using
it. Doing so avoids too long-drawn PRs where the entire solution is new to the
reviewer when the PR is submitted. Mostly relevant for larger changes of course, and changes
where which solution to use is not obvious.

### Assigning / assigned reviewer

Most PRs should have one reviewer. But sometimes it makes sense to have multiple. When creating
a PR, assign someone who is knowledgeable in the parts of the code that is touched. If someone
on the team is working on a high priority issue with a tight schedule, try to avoid choosing them
unless they are the only one who can review the particular code in question.

If you are not the assigned reviewer you are of course still welcome to read and comment on code.
After all, all merged code becomes everyone's code. But since you have probably not been in the
loop regarding this change, you might not be as informed as the author and the assigned reviewer.
Please bear this in mind and don't drive-by comment on things you are not familiar with. If you
really want to understand please ask the author via some other medium if they have the time
to explain the code in question, don't let it slow down the PR process.

### Work in progress (WIP) PRs

If you want to show your code to the reviewer early on, and you prefer a PR where comments
can be left over just sharing a remote branch. Then it's fine to submit a WIP PR. But it
should be marked as such. Add the WIP label and contact the reviewer and explain the
situation/intent.

You can also mark the PR as a draft on Github.

## UI design

At the time of writing this we don't have anyone dedicated fully to UI and UX. We don't have any UX
professional on the team. We solve all UI design by iterating and discussing in the team.
This generally works out quite well. But there is some friction to reach the end results, and it
takes time. Since we know from history that most UI changes will receive a fair amount of
feedback and be reworked a couple of times, it is even more important to start gathering feedback
on UI design even earlier than on code and programming solutions.


## Code formatting

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
experimented to get there. There is value in trying to keep the history relatively clean and
understandable. However, we should not spend a considerable amount of time just grooming history.
So the following advice is on a best-effort basis. If the history becomes hard to rearrange, just
clean the easy parts and squash or leave the rest as is.

For a given feature branch, there should generally not be "fix" commits fixing things from
earlier commits in that same branch. The developer might experiment back and forth with different
values or correct spelling errors etc. But the end result of the work should be the addition of the
correct values and the correctly spelled versions of what was added. When looking back in history,
the intermediate "backups" of the code the developer did does not add much value.

Example, instead of this history:
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

## Dead code

Don't keep dead code laying around generally. The cost of dead code is the extra complexity of
understanding the code base as well as every developer coming there thinking "Oh, maybe we can
remove this" and goes through the same hoops just to realize it's kept for some reason not in
the code itself.

Exceptions do exist of course. If one PR adds dead code and you plan on using it in the upcoming
PR right after or in the coming days. Then it might be fine. That might sometimes be preferred over
waiting with the first PR until everything is done and submitting one huge PR. But it's a case
by case basis and is best resolved between developer and reviewer.

Consider adding comments to dead code explaining why it is justified leaving it there.


[EditorConfig settings]: https://github.com/mullvad/mullvadvpn-app/blob/master/.editorconfig
[BOM]: https://en.wikipedia.org/wiki/Byte_order_mark
[keep a changelog]: https://keepachangelog.com/en/1.0.0/
