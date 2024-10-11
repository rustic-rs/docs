# Hooks

`rustic` supports hooks that are executed before and after certain commands.
This allows you to run custom scripts or commands before or after a backup,
restore, or other commands.

## Overview

`rustic` supports the following hooks (in order):

- `run-before` is executed either on a global level, before accessing the
  repository, or before a specific command (e.g. backup).
- `run-after` is executed either on a global level, after accessing the
  repository, or after a specific command (e.g. backup).
- `run-failed` is executed if a command before has failed.
- `run-finally` is executed after all other hooks of the same type have been
  executed, regardless of whether they succeeded or failed.

## Configuration

**Important**: For always up-to-date information, please make sure to check the
in-repository documentation for the config files available
[here](https://github.com/rustic-rs/rustic/blob/main/config/README.md).

Hooks are configured in the profile configuration file. Here is an example of a
configuration file with all possible hooks:

```toml
[global.hooks]
run-before = []
run-after = []
run-failed = []
run-finally = []

[repository.hooks]
run-before = []
run-after = []
run-failed = []
run-finally = []

[backup.hooks]
run-before = []
run-after = []
run-failed = []
run-finally = []

[[backup.snapshots]]
sources = []

[backup.snapshots.hooks]
run-before = []
run-after = []
run-failed = []
run-finally = []
```

Each hook is a list of commands that are executed in order. If you want to run a
script or command, you can add it to the list.

Hooks are also not executed in a shell, so you can't use shell features like
pipes or redirects. If you need to use shell features, you can run a shell
command that executes the desired command. Within the shell command, you can use
shell features.

For example:

```toml
[global.hooks]
run-before = [
  "sh -c  'echo Hello, > test.out'",
  "sh -c  'echo World! >> test.out'",
]
run-after = ["sh -c 'echo Goodbye, world! >> test.out'"]
```

In this example, the `run-before` hook is executed globally before any command
is executed. Within the hook, two commands are executed. The first command
writes `Hello,` to a file called `test.out`, and the second command appends
`World!` to the same file on a new line. The commands within the list of a hook
are executed in order.

So, after any other command is executed globally, the `run-after` hook is
executed. In this case, the command `echo 'Goodbye, world!'` is executed, and
the output is appended to the file `test.out` in a new line.

You can use hooks, e.g. to send a notification when a backup has finished:

```toml
[backup.hooks]
run-after = ["notify-send 'Backup finished successfully!'"]
```

## Use cases

Here are some use cases which might be interesting to use hooks for:

### Global hooks

- Send messages after successful / failed runs
- Feed your custom logging/monitoring with information about start/end of rustic
  calls

### Repository hooks

- Mount/umount the drive where the repository is located
- Sync your repo to some remote destination after each command
- Start extra integrity checks like checking the SHA256 against repo files

### Backup hooks

- Mount/umount a drive with data-to-backup
- Run commands to save some systems state, e.g. `dpkg --get-selections` on
  debian-based systems
- Dump a database into a file and remove the dump after backup (using
  `stdin-command` may be an alternative)

If you have a nice use-case yourself, please share it with others by making a
pull request to this documentation.
