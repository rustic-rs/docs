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

- custom_ignorefile -> custom_ignorefiles
- glob -> globs
- glob_file -> glob_files
- iglob -> iglobs
- iglob_file -> iglob_files
- tag -> tags

Update your configuration file accordingly, changing the key names from singular
to plural. For cases, where the key name was singular before, the value must be
wrapped in an array.

So, for example

```toml
tag = "important"
```

becomes:

```toml
tags = ["important"]
```
