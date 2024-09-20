# Configuration file

rustic supports configuration profile files in the TOML format. This allows you
to use rustic on the command line without the need to specify required or wanted
options. When using config profiles, rustic commands can be simply called as
`rustic backup`, `rustic snapshots` or `rustic forget` and all essential options
are read from the profile. So, using a config profile is the preferred way to
use rustic.

In this documentation, when giving example commands, we assume that a suitable
config profile is present and omit options which are usually set there.

**Important**: For always up-to-date information, please make sure to check the
in-repository documentation for the config files available
[here](https://github.com/rustic-rs/rustic/blob/main/config/README.md).

The configuration profile files are searched in the following locations:

- the global rustic config dir (on unix typically `/etc/rustic`)
- the users' rustic config dir. On unix this is typically
  `$HOME/.config/rustic`, see
  <https://docs.rs/directories/latest/directories/struct.ProjectDirs.html> for
  more details about the config location.
- the current working dir

By default, rustic uses the file `rustic.toml`. This can be overwritten by the
`-P <PROFILE>` option which tells rustic to search for a `<PROFILE>.toml`
configuration file. For example, if you have a `local.toml` configuration for
backing up to a local dir and a `remote.toml` configuration for a remote
storage, you can use `rustic -P local <COMMAND>` and
`rustic -P remote <COMMAND>`, respectively to switch between you two backup
configurations.

Note that options in the config profile file can always be overwritten by ENV
variables.

In the configuration profile file, you can specify all global and
repository-specific options as well as options/sources for the `backup` command
and `forget` options. Using a config file like

```toml
# rustic config file to backup /home and /etc to a local repository

[repository]
repository = "/backup/rustic"
password-file = "/root/key-rustic"
no-cache = true # no cache needed for local repository

[forget]
keep-daily = 14
keep-weekly = 5

[backup]
exclude-if-present = [".nobackup", "CACHEDIR.TAG"]
glob-file = ["/root/rustic-local.glob"]

[[backup.sources]]
source = "/home"
git-ignore = true

[[backup.sources]]
source = "/etc"
```

allows you to use `rustic backup` and `rustic forget --prune` in your regularly
backup/cleanup scripts.

For more config file examples
[check the config here](https://github.com/rustic-rs/rustic/tree/main/config)

## Configuration profile inheritance

rustic allows to read options from multiple configuration profiles and combines
them. This can be seen as inheritance of options from a parent profile. In fact
the `-P` command line option can be given multiple times and all config profiles
will be read and used.

For example, using `repo.toml`:

```toml
[repository]
repository = "/backup/rustic"
password-file = "/root/key-rustic"
no-cache = true # no cache needed for local repository
```

and `retention.toml`:

```toml
[forget]
keep-daily = 14
keep-weekly = 5
```

we can use `rustic -P repo -P retention forget` and get repository information
from the first and retention information from the second profile. This can be
used to stucture your configuration if you need to handle multiple repositories
or multiple use-cases.

Morever, there is also the possibiltiy to (recursively) read other profiles from
within a profile by using `use-profiles`, e.g.:

```toml
[global]
use-profiles = ["repo", "retention"]
```
