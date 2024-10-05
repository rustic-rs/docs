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
hooks = { run-before = [], run-after = [], run-failed = [], run-finally = [] }
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
run-before = ["sh -c  'echo Hello, world!'"]
```

In this example, the command `echo 'Hello, world!'` is executed before any
command is executed.

If you want to run multiple commands, you can add them to the list:

```toml
[global.hooks]
run-before = ["sh -c 'echo Hello, world!'"]
run-after = ["sh -c 'echo Goodbye, world!'"]
```

In this example, the command `echo 'Hello, world!'` is executed before any
command is executed, and the command `echo 'Goodbye, world!'` is executed after
any command is executed.

You can use that, e.g. to send a notification when a backup has finished:

```toml
[backup.hooks]
run-after = ["notify-send 'Backup finished'"]
```

You can also use the `--json` option to get the output of the command in JSON
format. This can be useful if you want to parse the output of a command in a
script.
