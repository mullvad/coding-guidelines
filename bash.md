# Bash coding guidelines

This document outlines best practices for writing Bash shell scripts. Except the things in this
document it is recommended to use [`shellcheck`] in order to find potential bugs and write
nice code.

[`shellcheck`]: https://www.shellcheck.net/

## Filename

Bash script files should follow the `kebab-case.sh` style convention unless special circumstances
mandates something else.

## Shebang

Start a Bash script with the following to make it parse consistently.
Without it it might be ran by either `bash` or `sh` depending on the environment,
and the outcome might be different.

```bash
#!/usr/bin/env bash
```

## Error handling

It's often good to start a Bash script with `set -eu` to make it sanity check itself
while executing.

The `-e` part makes the script exit if any command exits with a non zero exit code.
This is often desired, so the script aborts if anything goes wrong
instead of continue trying subsequent steps.

The `-u` part makes the script exit with an error if an undefined variable is referenced.

## Current working directory independence

You generally want `./scripts/foo.sh` and `cd scripts && ./foo.sh` to behave the same.
If a script uses relative paths the current working directory can affect the outcome in
undesirable ways. To mitigate this start the script by computing the path to the script
and `cd` to it:

```bash
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd "$SCRIPT_DIR"
```

The above is the most cross platform way of obtaining the directory of the script.

The exception to the above is if the script is intended to do something in the current
working directory. For example if an argument to the script is the directory to perform
some work on, and that argument defaults to the current working directory. In these cases,
prefer to make the argument mandatory and pass `.` instead.

## Variables

### Constants vs other variables

* Constant variables use `SCREAMING_SNAKE_CASE` names.

  This means variables that are assigned once and then never modified, and that only depend on
  other constant variables. The exception to "never modified" is where multiple statements are
  used to compute the content of the "constant" (see `COMPILE_FLAGS` in example at the end).

* Non-constant variables use `snake_case` names.

  This means variables that are modified throughout the execution of the script or that depend
  on other non-constant variables. For example loop variables and variables concatenating other
  variables where at least one element is a non-constant.

Environment variables and arguments to the script are considered constants. But arguments in
functions are not constants.

## Conditions and branching

Use `[[` for conditions, not `[`.

### Check if variable is defined

Since the script likely uses `set -u` it's an error to access undefined variables. The proper way
to check if a variable is set or unset is with `[[ -z ${my_variable+x} ]]`. Example:

```bash
if [[ -z ${product_version+x} ]]; then
    echo "Please pass product version as first argument"
    exit 1
fi
```

Prefer the above over defining the variable as the empty string, do some processing and then check
if it's still empty or not.

### Namespace

Use `local` for variables inside functions unless they need to be global for some reason
(They most likely should not be). This stops execution of functions from polluting the global
namespace. Do this both for constants and other variables.

```bash
function cowsay {
    local COW_PREFIX="The cow says"
    local message="$1"
    echo "$COW_PREFIX: $message"
}
```

## Functions

* Functions use `snake_case` names.
* Functions are declared with the `function` keyword and no parenthesis:
  ```bash
  function name_of_function {
      ...
  }
  ```

## Formatting

Place function opening braces on the same line as the function declaration, and branch
conditioning "`; then`"/"`; do`" etc on the same line as the `if`/`for` etc.

Use four (`4`) spaces for indentation.

```bash
function foo {
    if [[ "${1:-""}" != "--dev-build" ]]; then
        echo "Production build!"
    fi
}
```

## Parsing command-line arguments

When parsing command-line arguments that have flags, use a `while` loop containing a `case`
statement that matches on `$1`. Each case should set the appropriate settings for the flag and
optionally parse further flag specific arguments by using `$2`, `$3`, etc.
After parsing additional flag specific arguments make sure that you `shift` by that amount of
arguments. Because we almost always use `set -u` it is important that all variables that are set
during parsing have been initialized before parsing, otherwise they will not have a default value.
Finally make sure that the `case` statement is followed by a `shift`.

```bash
foo_set="false"
bar_arg_one="default value"
bar_arg_two="default value"
while [ "$#" -gt 0 ]; do
    case "$1" in
        "--foo")
            foo_set="true"
            ;;
        "--bar")
            bar_arg_one=$2
            bar_arg_two=$3
            shift 2
            ;;
        *)
            echo "Unknown argument: $1"
            exit 1
            ;;
    esac
    shift
done
```

## Example

Here is a script that does nothing useful, but it demonstrates the preferred formatting
and variable naming for many scenarios.

```bash
#!/usr/bin/env bash

set -eu

SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
cd "$SCRIPT_DIR"

# Base compile flags valid all the time
COMPILE_FLAGS="disable-foo enable-bar"
# Platform specific compile flags
if [[ "$(uname -s)" != "Darwin" ]]; then
    COMPILE_FLAGS+=" disable-apple"
fi

# Script arguments count as constants
OUTPUT_FILENAME="$1"
OUTPUT_PATH="$SCRIPT_DIR/$1"

function clear_tmp {
    local TMP_DIR="/tmp"
    local filename_pattern="$1"
    rm -f "$TMP_DIR/$filename_pattern"
}

for log_filename in *.log; do
    log_path="$SCRIPT_DIR/$log_filename"
    clear_tmp $log_filename
done
```
