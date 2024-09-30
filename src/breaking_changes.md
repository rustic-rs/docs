# Breaking Changes

This document lists all user facing breaking changes in `rustic` and provides
guidance on how to migrate from one version to another.

## 0.9.0

### Configuration File

#### Using String in `password-command` and `warm-up-command`

Using a String for `password-command` or `warm-up-command` have been re-allowed.

Note: The former version to give the command in an array is no longer supported.
Instead of

```toml
password-command = ["echo", "password"]
```

please use either of

```toml
password-command = "echo password"
password-command = { command = "echo", args = ["password"] }
```

#### Copy Command

Targets for the copy command must now be given by using existing rustic config
profile(s).

```toml
[copy]
targets = ["full", "rustic"]
```

#### Key Naming Conventions

Consistently apply singular and plural naming conventions for keys in the
configuration file that accept an array of values. This change affects the
following keys:

For `[global]`:

- `use-profile`-> `use-profiles` in config profile

For `[snapshot-filter]` and `[forget]`:

- `filter-host` -> `filter-hosts` in config profile
- `filter-label` -> `filter-labels` in config profile

For `[backup]`:

- `glob` -> `globs` in config profile
- `iglob` -> `iglobs` in config profile
- `glob-file` -> `glob-files` in config profile
- `iglob-file` -> `iglob-files` in config profile
- `custom-ignore-file` -> `custom-ignore-files` in config profile
- `tag`-> `tags` in config profile
- `source` -> `sources` in config profile
- `[[backup.sources]]` -> `[[backup.snapshots]]` in config profile

For `[forget]`:

- `keep-tags` -> now only array
- `keep-ids` -> now only array

Update your configuration file accordingly, changing the key names from singular
to plural. For cases, where the key name was plural before, the value must be
wrapped in an array.

So, for example

```toml
tag = "important"
```

becomes:

```toml
tags = ["important"]
```

Filters being affected are:

```toml
filter-hosts = ["host2"] # Default: []
filter-labels = ["label1"] # Default: []
```

The changes can be also derived from the
[PR](https://github.com/rustic-rs/rustic/pull/1240) that introduced this change.
So you may want to take a look at the PR for more details.
