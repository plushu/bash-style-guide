# Plushu Bash Style Guide

## Boilerplate

The following 2 lines begin every Bash script in Plushu and its plugins:

```bash
#!/usr/bin/env bash
set -eo pipefail; [[ $PLUSHU_TRACE ]] && set -x
```

The following is the rationale for each part of this.

### Line 1 (shebang)

    #!/usr/bin/env bash

We use `/usr/bin/env` to get the path to `bash` because certain systems may be set up so that a newer-installed version of `bash` is placed higher in `$PATH` than `/usr/bin/bash`. This is common in Mac OS X environments, which historically ship with an old (3.*) version of bash in `/usr/bin`, where developers often install a more recent one in their home directory (via `brew` or `macports`) to use when testing a script's correctness. 

### Line 2 (options)

    set -eo pipefail

We `set -e` so that any uncaught error in the script will cause it to exit, and we `set -o pipefail` so that an error in the middle of a pipeline will read as a failure as well (normally, the success or failure of a pipeline is decided by the last command in it).

Without exiting on uncaught errors, a script may continue execution after an unforeseen problem has arisen, further compounding whatever issue may have caused it to fail.

Using `set -e` as a standard practice is not universally agreed upon, but [when used properly](http://fvue.nl/wiki/Bash:_Error_handling), it comes out better than the alternative.

### Line 2, continued (tracing)

    [[ $PLUSHU_TRACE ]] && set -x

This line sets the script to output a trace of every command executed, for debugging. This allows a user to set "export PLUSHU_TRACE=true" in `.plushurc` to turn debug mode on for every action taken when running a command in Plushu.

(The `plushu` script itself places the second part of line 2 farther down in the script's execution, after the `.plushurc` that would set it has been sourced.)

## Quoting

Except for assignment, variables in Plushu are *always* quoted unless whitespace splitting or globbing is *explicitly needed*.

Even if a variable will never have a value with whitespace or glob characters, it is still quoted, for consistency. Also, you never know when some outside factor will break your assumptions about the "expected" value (see [stuartpb/dokku-rethinkdb-plugin#4](https://github.com/stuartpb/dokku-rethinkdb-plugin/issues/4), where an unexpected `"<no value>"` resulted in a check against both `"<no"` and `"value>"`).

## Echo

Plushu only uses `echo` to output messages, never variables (see [issue #22](https://github.com/plushu/plushu/issues/22) for the history and reasoning for this).
Plushu uses Bash herestrings to pass a variable to a command via stdin (rather than piping a command), and `printf '%s\n'` to output a variable to stdout or a file.

## Tests

All tests use the `[[` builtin "extended test" construction. Comparison tests use "==" (as used for comparison in virtually every other language) rather than "=" (which is easily confused with the assignment operator).

Testing for empty or non-empty strings is done with `-z` and `-n`, respectively, rather than letting the string be tested without a flag (for essentially the same reason as why `echo` is not used for writing variables).

Using an && conditional as a short test is frowned upon (with the exception of the PLUSHU_TRACE test in the header), as it can result in side-effects like [this](https://github.com/plushu/plushu-app-docker-opts/commit/29695d9ddf86fb47bf3c22ccf20620ce45adc1b4) when used as the last command of a script (which determines its exit code).

## Command Substitution

All command substitution should use `$()` rather than backticks. On top of being POSIX-standard and simpler to nest, `$()` constructs use the same backslash escaping rules as surrounding code. Backtick constructions have their own crazy rules about backslash escaping (see http://mywiki.wooledge.org/BashFAQ/082 and [the manual](http://www.gnu.org/software/bash/manual/bashref.html#Command-Substitution)).
