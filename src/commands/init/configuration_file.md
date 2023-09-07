# Configuration file

**Important**: For always up-to-date information, please make sure to check the
in-repository documentation for the config files available
[here](https://github.com/rustic-rs/rustic/blob/main/config/README.md).

Rustic supports configuration files in the TOML format which should be located
in the rustic config dir. On unix this is typically `$HOME/.config/rustic`, see
<https://docs.rs/directories/latest/directories/struct.ProjectDirs.html> for
more details about the config location. If no rustic config dir is available,
rustic searches the current working dir for configuration files.

By default, rustic uses the file `rustic.toml`. This can be overwritten by the
`-P <PROFILE>` option which tells rustic to search for a `<PROFILE>.toml`
configuration file. For example, if you have a `local.toml` configuration for
backing up to a local dir and a `remote.toml` configuration for a remote
storage, you can use `rustic -P local <COMMAND>` and
`rustic -P remote <COMMAND>`, respectively to switch between you two backup
configurations.

Note that options in the config file can always be overwritten by ENV

In the configuration file, you can specify all global and repository-specific
options as well as options/sources for the `backup` command and `forget`
options. Using a config file like

```toml
# rustic config file to backup /home and /etc to a local repository

[repository]
repository = "/backup/rustic"
password-file = "/root/key-rustic"
no-cache = true # no cache needed for local repository

[forget]
keep-daily = 14 keep-weekly = 5

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
