# Mullvad development guidelines

These documents aim to be the source of information for how we architect, design and format
program code in various languages and tools.

## Complexity, edge cases and problem scope

When reading specifications for a feature and trying to come up with a solution, or when
you are implementing something and realize new edge cases: Think critically and reason
about the situation and the actual users of this program. Reach out to other developers
or product owners in case of doubt instead of assuming too much.

Be smart about where corners can be cut. Don't try to cover every possible strange edge
case directly in the PoC implementation. Make the unlikely scenarios result in errors
and keep the code simple over writing something that is very generic or complex to
support cases you don't yet know if they will happen.

Don't make things a lot more generic than they need to be. If you need to generate a firewall
rule that allows `foo`. Then call it `create_allow_foo_rule()` and not
`create_rule(Action, Subject)` unless you actually need multiple rules where the code reuse
outweighs the added complexity of being generic.

## Code quality and documentation

When you change or add something, leave the code base in an equally good, or better state
than you found it in. Don't reduce the percentage of documentation by adding undocumented
code. Don't rush to publish a PR without critically examining your own code and reflect on
your chosen variable and function names, the level of documentation etc.
Spending a few hours polishing stuff before having it reviewed and merged
will definitely pay off by not having a lot of other people spend more time than that on trying
to understand what it does.

Productivity in the long term is more important than productivity in the short term. It is fine
that the current feature or bugfix takes a bit longer to implement if it means that the
developers coming here after you can spend less time doing what they need to do.

### What to document

Try to document code so it *adds* information and value. Don't just restate what the code
already tells the reader. The best is of course to have the code be self explanatory.
But some aspects of functionality is hard to explain in code only.

Consider the following function signature:
```rust
fn spawn_cmd(command: String) -> CmdHandle
```

Adding `/// Spawns the command as a subprocess` adds almost no value to the reader. It's
almost obvious from the function signature. What I as a reader and user of the function
want *do want to know* is:
* What happens if I drop the handle? Will the subprocess be killed?
* Can `command` include the arguments? Or can I just run programs without arguments?
* What will `stdin`, `stdout` and `stderr` of the child process be connected to?

See the pattern? What does the function do that is not obvious from the signature?

Not every single internal function needs documentation. As stated above, only things that the
code does not express by itself should be documented. Functions and constants which are 100%
obvious what they do or are used for does not need a comment.


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

For UI design, we prefer to iterate quickly and collaboratively. To agree upon the design before
writing any code for it. This is to avoid double-work where the implemented design will need to
be reworked after receiving feedback too late.

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

## Variable naming

### Files, paths and directories

Having variables that in some way represent files in the filesystem is very common.
If they always follow the same naming pattern the developer does not need to get creative,
and the reader will be familiar more often.

* `foo_path` - Variables containing a path to a **file**, both absolute and relative paths.
  Examples include `/tmp/lol.data`, `./output.log`.

* `foo_dir` - Variables containing a path to a **directory**, both absolute and relative paths.
  Examples include `/tmp`, `./output`, `../foo/bar`. Their values should (usually) not end with
  a path separator (`/`).

* `foo_filename` - Variables containing filenames that are not paths. Usually used to build
  paths with by concatenating it with a `_path` or `_dir`.
  Example: `foo.log`.

* `foo_dirname` - Variables containing a directory name that is not a path.

In the cases where a variable contains just parts of a path or filename (`".exe"`),
or a formatting string that will compute to a path (`"/var/log/my_program.{}.log"`).
Then include `prefix`, `suffix`, `template` or similar relevant identifier in the end of the
variable name.

## Language specific guidelines

* [Rust](rust.md)
* [Bash](bash.md)


## Git

### Commit messages

Here is a nice guide on how to write good commit messages:
https://chris.beams.io/posts/git-commit/

So sum it up the most important parts are:

* Capitalize the subject line.
* Do not end the subject line with a period.
* Use the imperative mood in the subject line, starting with a verb.
* Limit the subject line to 72 characters*
* Separate the subject from the body with two regular newlines ('\n')

\* The only exceptions to this length limit are merge and revert commits. They should have the
commit message git inserts by default (`Merge branch '<name_of_branch>'` and
`Revert "<subject_of_reverted_commit"`) even if the total length exceeds the guideline.

### Branch naming

Use kebab case git branch names with lowercase letters. For example `add-links-to-readme`.

An exception to the style is if a part in the branch name requires a specific way of writing to
not lose its meaning/technical context. Examples include:

* `suppress-CVE-2026-3872` - "CVE-2026-3872" is a unique identifier. Changing it could change its
  meaning and/or make it less recognizable.
* `fix-multicast-address-translation-IOS-123` - "IOS-123" being a hypothetical ticketing system
  identifier requiring a specific format for the automation to work.

Try to avoid adding exact programming terms that do not need to be in the title, without
reducing the information in the name too much. Instead of `implement-Clone-for-IpAddr` prefer
`make-ip-address-type-clonable`.

Try to describe what is being done in the branch, without being too verbose. High signal to noise
ratio is important. Some examples:
* Instead of `improve-readme` use `add-build-instructions`
* Instead of `fix-routing-table` use `fix-linux-lan-route-parsing`

There is no exact length limit on branch names, but try to stay relatively short.

Slashes in branch names are allowed if the team use them consistently and according to an agreed
upon system. Examples could include `android/fix-icon-rotation` or `linux/feature/add-multihop`.

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

### Co-authored commits

When multiple authors have made significant contributions to a commit&mdash;for instance, when
squashing large commits from multiple authors into one commit&mdash;add co-author information to the
commit message for the remaining authors:

```bash
$ git commit -m "A very good commit.

Co-authored-by: Name 1 <name1@example.com>
Co-authored-by: Name 2 <name2@example.com>"
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
