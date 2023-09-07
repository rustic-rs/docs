# Environment Variables

**Important**: For always up-to-date information, please make sure to check the
in-repository documentation for the config files available
[here](https://github.com/rustic-rs/rustic/blob/main/config/README.md).

In addition to command-line options, rustic supports passing various options in
environment variables. The following lists these environment variables:

```console
RUSTIC_REPOSITORY                   Location of repository (replaces -r)
RUSTIC_REPO_HOT                     Location of hot repository (replaces -repo-hot)
RUSTIC_PASSWORD                     The actual password for the repository (replaces --password)
RUSTIC_PASSWORD_FILE                Location of password file (replaces --password-file)
RUSTIC_PASSWORD_COMMAND             Command printing the password for the repository to stdout (replaces --password-command)
RUSTIC_CACHE_DIR                    Location of the cache directory (replaces --cache-dir)
RUSTIC_NO_CACHE                     Use no cache (replaces --no-cache)
```

rustic may execute `rclone` (for rclone backends) which may respond to further
environment variables and configuration files.
