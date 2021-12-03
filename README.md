# markdown-lint Configurations

* Rules List - <https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md>

## Reference - Current SSDR configuration

* Sample file: [Reference_umd-fcrepo-local-development-setup.md](Reference_umd-fcrepo-local-development-setup.md)

Reference "markdownlint.config" settings

```json
  "markdownlint.config": {

    "MD013": false,
    "MD014": false,
    "MD029": false,
    "MD040": false,
    "MD046": { "style": "fenced" }
  }
```

## Config1 - markdown-lint extension default configuration

* Sample File: [Config1_umd-fcrepo-local-development-setup.md](Config1_umd-fcrepo-local-development-setup.md)

Config1 "markdownlint.config" settings

```json
  "markdownlint.config": {
    "MD013": false,
  },
```

### Config1 Issues

* This configuration enforces [MD029][MD029]
    "ol-prefix - Ordered list item prefix"

    Forces ordered list numbering in the raw file to always be "1" (can't use
    "1", "2", "3", must be "1", "1", "1"

    Note the numbering in the "Quick Start" section, which is badly broken by
    this rule.

    Even using a default of "ordered" or "one_or_ordered" is problematic,
    because the rule does not appear to recognize list items separated by
    code blocks (as in Quick Start).

* See [MD014 Concerns](#MD014-concerns)

## Config2 - Minimal acceptable configuration?

* Sample File: [Config2_umd-fcrepo-local-development-setup.md](Config1_umd-fcrepo-local-development-setup.md)

Config1 "markdownlint.config" settings

```json
  "markdownlint.config": {
    "MD013": false,
    "MD029": false,
  },
```

### Config2 Issues

* See [MD014 Concerns](#MD014-concerns)

----

## MD014 Concerns

The [MD014][MD014] "Dollar signs used before commands without showing output"
suggests not using dollar signs before commands if the code block consists
only of commands.

One of the rationales put forward for this is that it makes it easy to copy and
paste multiple commands all at once.

The following are some reasons why the current SSDR configuration (which
disables this rule), and that the policy of always marking commands with
an initial "$ " is to be preferred.

### 1) Harder to read

Both of the sample Config1 and Config2 files contains two code blocks that are
supposed to be placed in a file, instead of run on the command line. Which
ones are they?

### 2) Copy and paste possibly harmful

Consider the possibility of copying and pasting the wrong thing:

For example, suppose there is a code block with the following commands:

```bash
cd /tmp/foobar
rm -rf *
```

and when you copy and paste, you either don't capture the first line, or capture
only part of the first line, so that the actual pasted commands are:

```bash
d /tmp/foobar
rm -rf *
```

The first command prints an error, but the second command still runs, so the
current directory is destroyed.

**Note:** In a non-exhaustive search that I did of the GitHub repositories I
have checked out on my laptop, we don't have any Markdown files with `rm -rf *`
used in this manner, although <https://github.com/umd-lib/handle-custom> has
`rm -rf *.xml.versionsBackup`, so you could (in theory) still destroy stuff
unintentionally by failing to copy the entire line.

### 3) Not as convenient as hoped?

Consider the following commands:

```bash
sleep 1
sleep 10
sleep 50
```

Copy and paste the above commands into a terminal. In my terminal, at least,
the last command is _not_ run automatically, because there is no carriage
return on the end. So the first two commands have run, but the third command
is still waiting for the "Enter" key to be pressed. So there is still a
50-second wait for the final command (once you realize it hasn't run and hit
"Enter").

In order for this to work, you need to paste the code, and then remember to hit
"Enter" so that the final command runs. On my terminal, the last command is then
printed, meaning it is interspersed into the output of whatever command is
currently running.

**Note:** It does not seem possible to get around this by adding an additional
blank line at the end of the code block, and GitHub seems to ignore them.
For example:

```bash
sleep 1
sleep 10
sleep 50

```

### 4) Errors get hidden

Depending on the list of commands, a command earlier in the list may fail in
such a way that the output from later commands obscures the error.

For example, in the sample document, there are the following Docker commands:

```bash
docker build -t docker.lib.umd.edu/fcrepo-fuseki fuseki
docker build -t docker.lib.umd.edu/fcrepo-fixity fixity
docker build -t docker.lib.umd.edu/fcrepo-mail mail
```

It is possible that one of the first two commands could fail, and the error
get scrolled off the screen by the following commands. So you wouldn't know
anything was wrong until you tried to use them.

### 5) Metavariables

Some commands are written with "metavariables" (typically in ALL CAPS, possibly
enclosed brackets, such as "VERSION" in the following command:

```bash
docker build -t docker.lib.umd.edu/handle-custom:<VERSION> -f Dockerfile .
```

If this were part of a multi-command block, copying and pasting the commands
would not have the expected result.

----
[MD014]: https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md014
[MD029]: https://github.com/DavidAnson/markdownlint/blob/main/doc/Rules.md#md029
