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

### Getting more context

Hooks provide environment variables to external commands, depending on the the
hook's type.

| Variable name           | Content                                                  | Type                 |
| ----------------------- | -------------------------------------------------------- | -------------------- |
| `RUSTIC_HOOK_TYPE`      | `run-before`, `run-after`, `run-failed` or `run-finally` | global               |
| `RUSTIC_ACTION`         | `repository`, `backup` or `source-specific-backup`       | repository or backup |
| `RUSTIC_BACKUP_LABEL`   | label of snapshot                                        | backup               |
| `RUSTIC_BACKUP_SOURCES` | comma-separated sources                                  | backup               |
| `RUSTIC_BACKUP_TAGS`    | comma-separated tags                                     | backup               |

Note that `global` variables are provided to all commands.

### Hook command failure

By default, if a hook command fails, `rustic` will log the error and stop. If
you would like `rustic` to continue its process with just a warning in the logs
or nothing, you can use the `on-failure` field.

The `on-failure` field takes `"error"` by default and can take `"warn"` and
`"ignore"` as well.

A first way to write it:

```toml
[backup.hooks]
run-after = [
  { command = "notify-send", args = ["Backup finished successfully!"], on-failure = "warn" },
]
```

An alternative way possible with the toml format:

```toml
[[backup.hooks.run-after]]
command = "notify-send"
args = ["Backup finished successfully!"]
# optional
on-failure = "warn" # possible values are "error" (default), "warn",  "ignore"
```

### Example: heartbeat monitoring with Watchgoose

[Watchgoose](https://watchgoose.com/docs/) alerts you when a scheduled backup
fails or does not run at all. It uses the same ping URLs as healthchecks.io:
`/start` when the job starts, the bare URL on success and `/fail` on failure.
Replace the UUID with the one of your check:

```toml
[backup.hooks]
run-before = [
  { command = "curl", args = ["-fsS", "-m", "10", "--retry", "3", "-o", "/dev/null", "https://watchgoose.com/ping/addffa72-da17-40ae-be9c-ff591afb942a/start"], on-failure = "warn" },
]
run-after = [
  { command = "curl", args = ["-fsS", "-m", "10", "--retry", "3", "-o", "/dev/null", "https://watchgoose.com/ping/addffa72-da17-40ae-be9c-ff591afb942a"], on-failure = "warn" },
]
run-failed = [
  { command = "curl", args = ["-fsS", "-m", "10", "--retry", "3", "-o", "/dev/null", "https://watchgoose.com/ping/addffa72-da17-40ae-be9c-ff591afb942a/fail"], on-failure = "warn" },
]
```

`on-failure = "warn"` makes sure that an unreachable monitoring server does not
abort the backup.

Backup hooks only run once the repository is open. If `rustic` fails earlier,
e.g. because of a wrong password or an unreachable repository, no `/fail` ping
is sent and Watchgoose alerts you when the expected ping does not arrive. To
report these failures immediately, put the same hooks in `[repository.hooks]` of
a profile that you only use for the scheduled backup, as repository hooks also
run for other commands like `snapshots`.

Watchgoose stores the first 10,000 bytes of a POST body as the ping log. To
attach the end of the `rustic` log to a failure, set a log file and send it with
`run-failed`:

```toml
[global]
log-file = "/var/log/rustic.log"

[backup.hooks]
run-failed = [
  { command = "sh", args = ["-c", "tail -c 10000 /var/log/rustic.log | curl -fsS -m 10 --retry 3 -o /dev/null --data-binary @- https://watchgoose.com/ping/addffa72-da17-40ae-be9c-ff591afb942a/fail"], on-failure = "warn" },
]
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
