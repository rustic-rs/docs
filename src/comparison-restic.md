# Comparison between `rustic` and `restic`

Note that we regularly update this document to compare the latest versions of
rustic and restic. Currently, we compare restic 0.18.0 with rustic 0.10.0.

## General differences

|                       | `restic`                  | `rustic`                                               |
| --------------------- | ------------------------- | ------------------------------------------------------ |
| programming language  | Go                        | Rust                                                   |
| development philosopy | conservative with changes | moving fast, add new features early                    |
| test coverage         | ✅                        | ❌ (43% in rustic_core)                                |
| returns error code    | ✅                        | (✅) only 0 or 1; not all commands support it          |
| available as library  | ❌                        | ✅ [rustic_core](https://crates.io/crates/rustic_core) |

## Core features introduced by rustic

rustic's goal is to implement all functionality/features restic offers - and
make some of them even better. It also implements new features which are missing
in restic.

This section is an advertisement of the most important features uniquely
introduced by rustic. Some have been already adopted by restic.

| feature                             | `restic`                       | `rustic`                                            |
| ----------------------------------- | ------------------------------ | --------------------------------------------------- |
| cold storage support                | ❌ (may work in special cases) | ✅ (full support including warm-up of needed data)  |
| lock-free                           | ❌ (roadmap: 0.19)             | ✅ (lock-free operations, two-phase pruning)        |
| in-place restore                    | ✅                             | ✅                                                  |
| config profile support              | ❌ (wrapper tools available)   | ✅                                                  |
| hooks                               | ❌                             | ✅ (at many points, configurable in config profile) |
| additional snapshot information     | (✅) (partly added)            | ✅ (see below for details)                          |
| in-repo config                      | ❌                             | ✅ (see below for details)                          |
| custom chunker config               | ❌                             | ✅ (min/max/average chunk size; fixed size chunker) |
| `<snapshot>:<path>` syntax          | ✅ (most commands)             | ✅                                                  |
| new command: `merge`                | ❌                             | ✅                                                  |
| new command: `webdav`               | ❌                             | ✅                                                  |
| `diff` with local files             | ❌                             | ✅                                                  |
| `backup` can use .gitignore         | ❌ (roadmap: 0.19)             | ✅                                                  |
| `backup` multiple snapshots at once | ❌                             | ✅                                                  |
| `check` uses existing cache         | ❌ (roadmap: 0.19)             | ✅                                                  |
| show file history                   | ❌                             | ✅ (`rustic find --path`)                           |
| more snapshot filter options        | ❌                             | ✅ (see below for details)                          |
| allow to log to file                | ❌                             | ✅                                                  |
| log verbosity                       | `-v` or `--quiet`              | `--log-level`                                       |
| telemetry support                   | ❌                             | ✅ (for `backup`, Prometheus and OpenTelemetry)     |
| interactive mode (TUI)              | ❌                             | ✅                                                  |

## Supported storage backends

| backend                    | `restic`                                        | `rustic`                                                                |
| -------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------- |
| `local`                    | ✅ (built-in)                                   | ✅ (built-in)                                                           |
| `sftp`                     | ✅ (using external `ssh` command)               | ✅ (built-in using `opendal`, windows not supported)                    |
| `rest`                     | ✅ (built-in)                                   | ✅ (built-in)                                                           |
| `s3`                       | ✅ (built-in)                                   | ✅ (built-in using `opendal`)                                           |
| `swift`                    | ✅ (built-in)                                   | ✅ (built-in using `opendal`)                                           |
| `b2`                       | ✅ (built-in)                                   | ✅ (built-in using `opendal`)                                           |
| `azure`                    | ✅ (built-in)                                   | ✅ (built-in using `opendal`)                                           |
| `gs`                       | ✅ (built-in)                                   | ✅ (built-in using `opendal`)                                           |
| `dropbox`                  | ❌                                              | ✅ (built-in using `opendal`)                                           |
| `ftp`                      | ❌                                              | ✅ (built-in using `opendal`)                                           |
| `gdrive`                   | ❌                                              | ✅ (built-in using `opendal`)                                           |
| `onedrive`                 | ❌                                              | ✅ (built-in using `opendal`)                                           |
| `webdav`                   | ❌                                              | ✅ (built-in using `opendal`)                                           |
| `opendal` (other services) | ❌                                              | ✅ (built-in using `opendal`) (see [here](./commands/init/services.md)) |
| `rclone`                   | ✅ (via stdin, using external `rclone` command) | ✅ (via http on localhost, using external `rclone` command)             |

## Commands

| command            | `restic`               | `rustic`                                             |
| ------------------ | ---------------------- | ---------------------------------------------------- |
| `backup`           | ✅                     | ✅                                                   |
| `cache`            | ✅                     | ❌                                                   |
| `cat`              | ✅                     | ✅                                                   |
| `config`           | ❌ (no in-repo config) | ✅                                                   |
| `check`            | ✅                     | ✅                                                   |
| `copy`             | ✅                     | ✅                                                   |
| `diff`             | ✅                     | ✅                                                   |
| `dump`             | ✅                     | ✅                                                   |
| `find`             | ✅                     | ✅                                                   |
| `forget`           | ✅                     | ✅                                                   |
| `generate`         | ✅                     | ✅ `completions`                                     |
| `init`             | ✅                     | ✅                                                   |
| `key list`         | ✅                     | ✅                                                   |
| `key add`          | ✅                     | ✅                                                   |
| `key remove`       | ✅                     | ✅                                                   |
| `key passwd`       | ✅                     | ✅                                                   |
| `list`             | ✅                     | ✅                                                   |
| `ls`               | ✅                     | ✅                                                   |
| `merge`            | ❌                     | ✅                                                   |
| `migrate`          | ✅                     | ❌ (not needed; repo version migration via `config`) |
| `mount`            | ✅                     | ✅ (linux only)                                      |
| `prune`            | ✅                     | ✅                                                   |
| `recover`          | ✅                     | ❌                                                   |
| `repair index`     | ✅                     | ✅                                                   |
| `repair packs`     | ✅                     | ❌                                                   |
| `repair snapshots` | ✅                     | ✅                                                   |
| `repoinfo`         | ❌                     | ✅                                                   |
| `restore`          | ✅                     | ✅                                                   |
| `rewrite`          | ✅                     | ❌                                                   |
| `self-update`      | ✅                     | ✅                                                   |
| `show-config`      | ❌                     | ✅                                                   |
| `snapshots`        | ✅                     | ✅                                                   |
| `stats`            | ✅                     | ❌ (but there is `repoinfo`)                         |
| `tag`              | ✅                     | ✅                                                   |
| `unlock`           | ✅                     | lock-free                                            |
| `webdav`           | ❌                     | ✅                                                   |

## Information saved in snapshots

| information                                                                                    | `restic` | `rustic` |
| ---------------------------------------------------------------------------------------------- | -------- | -------- |
| [from repo design info](https://github.com/restic/restic/blob/master/doc/design.rst#snapshots) | ✅       | ✅       |
| program version used                                                                           | ✅       | ✅       |
| summary (size,...)                                                                             | ✅       | ✅       |
| used command                                                                                   | ✅       | ✅       |
| label                                                                                          | ❌       | ✅       |
| description                                                                                    | ❌       | ✅       |
| delete (protection)                                                                            | ❌       | ✅       |

## General options

| option                       | `restic`                                | `rustic`                                                  |
| ---------------------------- | --------------------------------------- | --------------------------------------------------------- |
| `--cacert`                   | ✅                                      | ❌                                                        |
| `--cache-dir`                | ✅                                      | ✅ (or in config profile)                                 |
| `--cleanup-cache`            | ✅                                      | ❌                                                        |
| `--compression`              | ✅ (auto,max,off); needed in every call | ✅ (-7..22) configure once in in-repo config              |
| `--dry-run`                  | ✅                                      | ✅ (or in config profile)                                 |
| `--insecure-no-password`     | ✅                                      | ✅ (empty passwords work without extra option)            |
| `--insecure-tls`             | ✅                                      | ❌                                                        |
| `--json`                     | ✅                                      | ✅                                                        |
| `--key-hint`                 | ✅                                      | ❌                                                        |
| `--limit-download`           | ✅                                      | (✅) (for opendal option `trottle`)                       |
| `--limit-upload`             | ✅                                      | (✅) (for opendal option `trottle`)                       |
| `--log-file`                 | ❌                                      | ✅ (or in config profile)                                 |
| `--no-cache`                 | ✅                                      | ✅ (or in config profile)                                 |
| `--no-extra-verify`          | ✅ needed in every call                 | ✅ configure once using `config`                          |
| `--no-lock`                  | ✅                                      | ✅ all operations are lock-free                           |
| `--no-progress`              | ❌                                      | ✅ (or in config profile)                                 |
| `--progress-intervall`       | ✅ (via env variable)                   | ✅ (or in config profile)                                 |
| `--option`                   | ✅ as cmd arg or env variable           | ✅ via config profile or env variable                     |
| `--opentelemetry`            | ❌                                      | ✅ (or in config profile, only for `backup` currently)    |
| `--pack-size`                | ✅ fix limit; needed in every call      | ✅ fix or dynamic limit, configure once in in-repo config |
| `--password-command`         | ✅                                      | ✅ (or in config profile)                                 |
| `--password-file`            | ✅                                      | ✅ (or in config profile)                                 |
| `--prometheus`               | ❌                                      | ✅ (or in config profile, only for `backup` currently)    |
| `--prometheus-user`          | ❌ (no prometheus support)              | ✅ (or in config profile)                                 |
| `--prometheus-pass`          | ❌ (no prometheus support)              | ✅ (or in config profile)                                 |
| `--quiet`                    | ✅                                      | ✅                                                        |
| `--repo`                     | ✅                                      | ✅ (or in config profile)                                 |
| `--repo-hot`                 | ❌ (no cold-storage support)            | ✅ (or in config profile)                                 |
| `--repository-file`          | ✅                                      | ❌ (use `repository` in config profile instead)           |
| `--retry-lock`               | ✅                                      | not needed; lock-free                                     |
| `--tls-client-cert`          | ✅                                      | ❌                                                        |
| `--use-profile`              | ❌ (no config profile support)          | ✅ (or in config profile for recursively using profiles)  |
| `--verbose` (multiple times) | ✅                                      | ✅ `--log-level`                                          |
| `--warm-up`                  | ❌ (no cold-storage support)            | ✅ (or in config profile)                                 |
| `--warm-up-wait`             | ❌ (no cold-storage support)            | ✅ (or in config profile)                                 |
| `--warm-up-wait-command`     | ❌ (no cold-storage support)            | ✅ (or in config profile)                                 |

## `rustic` in-repo config options

| option                          | `restic`                                             | `rustic`     |
| ------------------------------- | ---------------------------------------------------- | ------------ |
| `append_only`                   | ❌                                                   | ✅           |
| `chunker`                       | ❌                                                   | ✅           |
| `chunk-size`                    | ❌                                                   | ✅           |
| `chunk-min-size`                | ❌                                                   | ✅           |
| `chunk-max-size`                | ❌                                                   | ✅           |
| `compression`                   | ✅ (by `--compression`)                              | ✅           |
| `treepack_size`                 | ❌ (only all packs: `--pack-size`)                   | ✅           |
| `treepack_growfactor`           | ❌                                                   | ✅           |
| `treepack_size_limit`           | ❌                                                   | ✅           |
| `datapack_size`                 | ❌ (only all packs: `--pack-size`)                   | ✅           |
| `datapack_growfactor`           | ❌                                                   | ✅           |
| `datapack_size_limit`           | ❌                                                   | ✅           |
| `min_packsize_tolerate_percent` | ❌ (hardcoded 80% for `prune --repack-small`)        | ✅           |
| `max_packsize_tolerate_percent` | ❌                                                   | ✅           |
| `extra_verify`                  | ✅ (default, can be unset using `--no-extra-verify`) | ✅ (default) |

## Snapshot filtering

| filter                | `restic`     | `rustic` (options also in config profile) |
| --------------------- | ------------ | ----------------------------------------- |
| by host               | ✅ `--host`  | ✅ `--filter-host`                        |
| by label              | ❌           | ✅ `--filter-label`                       |
| by paths              | ✅ `--paths` | ✅ `--filter-paths`                       |
| by exact pathlists    | ❌           | ✅ `--filter-paths-exact`                 |
| by tags               | ✅ `--tags`  | ✅ `--filter-tags`                        |
| by exact tagists      | ❌           | ✅ `--filter-tags-exact`                  |
| by date/time          | ❌           | ✅ `--filter-before`, `filter-after`      |
| by size               | ❌           | ✅ `--filter-size`                        |
| by size added to repo | ❌           | ✅ `--filter-size-added`                  |
| custom `jq` syntax    | ❌           | ✅ `--filter-jq`                          |

## Comparison of important commands

### `init`

| option                  | `restic`                       | `rustic`                         |
| ----------------------- | ------------------------------ | -------------------------------- |
| `--copy-chunker-params` | ✅                             | (not needed, use `copy --init`)  |
| `--from-*`              | ✅                             | (not needed, see `copy` command) |
| `--hostname`            | ❌ (always sets hostname)      | ✅                               |
| `--repository-version`  | ✅                             | ✅ (use `--set-version`)         |
| `--set-*`               | ❌ (no in-repo config support) | ✅                               |
| `--username`            | ❌ (always sets username)      | ✅                               |
| `--with-created`        | ❌ (always sets creation time) | ✅                               |

### `backup`

| general                                         | `restic`                    | `rustic` |
| ----------------------------------------------- | --------------------------- | -------- |
| allow to create multiple snapshot in single run | ❌                          | ✅       |
| allow to backup relative paths                  | (✅) full path in snapshots | ✅       |

| option                  | `restic`                          | `rustic` (options also in config profile) |
| ----------------------- | --------------------------------- | ----------------------------------------- |
| `--as-path`             | ❌                                | ✅                                        |
| `--command`             | ❌                                | ✅                                        |
| `--custom-ignorefile`   | ❌                                | ✅                                        |
| `--description`         | ❌                                | ✅                                        |
| `--description-from`    | ❌                                | ✅                                        |
| `--delete-never`        | ❌                                | ✅                                        |
| `--delete-after`        | ❌                                | ✅                                        |
| `--exclude`             | ✅                                | ✅ `--glob`                               |
| `--exclude-file`        | ✅                                | ✅ `--glob-file`                          |
| `--exclude-caches`      | ✅                                | ❌ (use `--exclude-if-present`)           |
| `--exclude-if-present`  | ✅ (+ support for header parsing) | ✅ (but no header parsing)                |
| `--exclude-larger-than` | ✅                                | ✅                                        |
| `--files-from`          | ✅                                | ❌                                        |
| `--files-from-raw`      | ✅                                | ❌                                        |
| `--files-from-verbatim` | ✅                                | ❌                                        |
| `--force`               | ✅                                | ✅                                        |
| `--git-ignore`          | ❌ (roadmap: 0.19)                | ✅                                        |
| `--group-by`            | ✅ (host/paths/tags)              | ✅ (host/label/paths/tags)                |
| `--host`                | ✅                                | ✅                                        |
| `--iexclude`            | ✅                                | ✅ `--iglob`                              |
| `--iexclude-file`       | ✅                                | ✅ `--iglob-file`                         |
| `--ignore-ctime`        | ✅                                | ✅                                        |
| `--ignore-inode`        | ✅                                | ✅                                        |
| `--ignore-devid`        | ❌                                | ✅                                        |
| `--init`                | ❌                                | ✅                                        |
| `--label`               | ❌                                | ✅                                        |
| `--no-require-git`      | ❌ (no `--git-ignore`)            | ✅                                        |
| `--no-scan`             | ✅                                | ✅                                        |
| `--one-file-system`     | ✅                                | ✅                                        |
| `--parent`              | ✅                                | ✅                                        |
| `--read-concurrency`    | ✅                                | ❌ (hardcoded)                            |
| `--skip-if-unchanged`   | ✅                                | ✅                                        |
| `--stdin`               | ✅                                | ✅ (use `-` as backup source)             |
| `--stdin-filename`      | ✅                                | ✅                                        |
| `--stdin-from-command`  | ✅                                | ✅                                        |
| `--tag`                 | ✅                                | ✅                                        |
| `--time`                | ✅                                | ✅                                        |
| `--with-atime`          | ✅                                | ✅                                        |

### `restore`

| general                                | `restic` | `rustic` |
| -------------------------------------- | -------- | -------- |
| scan and use already existing files    | ✅       | ✅       |
| resumable restore                      | ✅       | ✅       |
| restore hard links                     | ✅       | ❌       |
| `<snapshotID>:<subfolder>` syntax      | ✅       | ✅       |
| `<snapshotID>:<subfolder>/file` syntax | ❌       | ✅       |
| dry run support                        | ✅       | ✅       |

| option                         | `restic`                | `rustic`                                    |
| ------------------------------ | ----------------------- | ------------------------------------------- |
| filtering options for `latest` | ✅                      | ✅                                          |
| `--delete`                     | ✅                      | ✅                                          |
| `--exclude`                    | ✅                      | ✅ `--glob`                                 |
| `--iexclude`                   | ✅                      | ✅ `--iglob`                                |
| `--iinclude`                   | ✅                      | ✅ `--iglob`                                |
| `--include`                    | ✅                      | ✅ `--glob`                                 |
| `--no-ownership`               | ❌                      | ✅                                          |
| `--numeric-id`                 | ❌                      | ✅                                          |
| `--overwrite`                  | ✅                      | ❌ (missing functionality: if-newer, never) |
| `--sparse`                     | ✅                      | ❌                                          |
| `--target`                     | ✅                      | ✅ (give target as second CLI argument)     |
| `--verify`                     | ✅                      | ❌ (but `diff` can be used to verify after) |
| `--verify-existing`            | ✅ `--overwrite always` | ✅                                          |

### `dump`

| general                             | `restic` | `rustic` |
| ----------------------------------- | -------- | -------- |
| dump files                          | ✅       | ✅       |
| dump dirs                           | ✅       | ✅       |
| allow auto-detection of used format | ❌       | ✅       |

| option                                  | `restic`     | `rustic`                        |
| --------------------------------------- | ------------ | ------------------------------- |
| snapshot filtering options for `latest` | ✅           | ✅                              |
| `--archive`                             | ✅ (tar,zip) | ✅ (tar,targz,zip,content,auto) |
| `--target`                              | ✅           | ✅ `--file`                     |

### `forget`

| general                                 | `restic` | `rustic` |
| --------------------------------------- | -------- | -------- |
| allow to keep all XXX                   | ✅       | ✅       |
| respect "no delete" options in snapshot | ❌       | ✅       |

| option                         | `restic`             | `rustic` (options also in config profile) |
| ------------------------------ | -------------------- | ----------------------------------------- |
| snapshot filtering options     | ✅                   | ✅                                        |
| `--keep-last`                  | ✅                   | ✅                                        |
| `--keep-minutely`              | ❌                   | ✅                                        |
| `--keep-hourly`                | ✅                   | ✅                                        |
| `--keep-daily`                 | ✅                   | ✅                                        |
| `--keep-daily`                 | ✅                   | ✅                                        |
| `--keep-weekly`                | ✅                   | ✅                                        |
| `--keep-monthly`               | ✅                   | ✅                                        |
| `--keep-quarter-yearly`        | ❌                   | ✅                                        |
| `--keep-half-yearly`           | ❌                   | ✅                                        |
| `--keep-yearly`                | ✅                   | ✅                                        |
| `--keep-within`                | ✅                   | ✅                                        |
| `--keep-within-minutely`       | ❌                   | ✅                                        |
| `--keep-within-hourly`         | ✅                   | ✅                                        |
| `--keep-within-daily`          | ✅                   | ✅                                        |
| `--keep-within-weekly`         | ✅                   | ✅                                        |
| `--keep-within-monthly`        | ✅                   | ✅                                        |
| `--keep-within-quarter-yearly` | ❌                   | ✅                                        |
| `--keep-within-half-yearly`    | ❌                   | ✅                                        |
| `--keep-within-yearly`         | ✅                   | ✅                                        |
| `--keep-tag`                   | ✅                   | ✅                                        |
| `--usafe-allow-remove-all`     | ✅                   | ✅ `--keep-none`                          |
| `--delete-unchanged`           | ❌                   | ✅                                        |
| `--compact`                    | ✅                   | ❌                                        |
| `--group-by`                   | ✅ (host/paths/tags) | ✅ (host/label/paths/tags)                |
| `--prune`                      | ✅                   | ✅                                        |

### `prune`

| general                                    | `restic`           | `rustic` |
| ------------------------------------------ | ------------------ | -------- |
| prune plan without reading pack files      | ✅                 | ✅       |
| prune parallel to backup (two-phase prune) | ❌ (roadmap: 0.19) | ✅       |
| different pack sizes for tree/data packs   | ❌                 | ✅       |
| resumable prune                            | ✅                 | ✅       |
| (option to) resize packs                   | ✅                 | ✅       |

| option                           | `restic`                   | `rustic`                                          |
| -------------------------------- | -------------------------- | ------------------------------------------------- |
| `--fast-repack`                  | ❌                         | ✅                                                |
| `--instant-delete`               | ✅ (default, no two-phase) | ✅                                                |
| `--keep-pack`                    | ❌                         | ✅                                                |
| `--keep-delete`                  | ❌ (no two-phase)          | ✅                                                |
| `--max-repack-size`              | ✅                         | ✅ `--max-repack` (size/%/unlimited)              |
| `--max-unused`                   | ✅                         | ✅                                                |
| `--repack-all`                   | ❌                         | ✅                                                |
| `--repack-cacheable-only`        | ✅                         | ✅                                                |
| `--repack-small`                 | ✅                         | ✅ (default behavior; to unset use `--no-resize`) |
| `--repack-uncompressed`          | ✅                         | ✅                                                |
| `--unsafe-recover-no-free-space` | ✅                         | ✅ `--early-delete-index`                         |

### `check`

| general                       | `restic`                                      | `rustic`     |
| ----------------------------- | --------------------------------------------- | ------------ |
| check index files             | ✅                                            | ✅           |
| check index vs packs          | ✅                                            | ✅           |
| check snapshot files          | ✅                                            | ✅           |
| (optionally) check pack files | ✅                                            | ✅           |
| only check given snapshots    | ❌                                            | ✅           |
| cache policy                  | create temporary (use existing: roadmap 0.18) | use existing |
| check cache integrity         | ❌                                            | ✅           |
| check hot/cold integrity      | ❌ (no cold storage support)                  | ✅           |

| option               | `restic`                      | `rustic`              |
| -------------------- | ----------------------------- | --------------------- |
| `--read-data`        | ✅                            | ✅                    |
| `--read-data-subset` | ✅                            | ✅                    |
| `--trust-cache`      | ❌ (no cache integrity check) | ✅                    |
| `--with-cache`       | ✅                            | ✅ (default behavior) |

### `copy`

| general                               | `restic`    | `rustic`          |
| ------------------------------------- | ----------- | ----------------- |
| source/target given by                | CLI options | in config profile |
| multiple targets                      | ❌          | ✅                |
| check for matching chunker parameters | ❌          | ✅                |

| option                     | `restic`                                       | `rustic`                                                |
| -------------------------- | ---------------------------------------------- | ------------------------------------------------------- |
| snapshot filtering options | ✅                                             | ✅                                                      |
| `--from-*`                 | ✅ (target is `--repository`)                  | ✅ (source is `--repository`, target in config profile) |
| `--init`                   | ❌ (extra run of `init --copy-chunker-params`) | ✅                                                      |

### `snapshots`

| general                                   | `restic` | `rustic` |
| ----------------------------------------- | -------- | -------- |
| summarize identical snapshots (like `+3`) | ❌       | ✅       |
| show summary information (sizes)          | ✅       | ✅       |

| option                     | `restic`             | `rustic`                   |
| -------------------------- | -------------------- | -------------------------- |
| snapshot filtering options | ✅                   | ✅                         |
| `--all`                    | ❌                   | ✅                         |
| `--compact`                | ✅                   | ❌                         |
| `--group-by`               | ✅ (host/paths/tags) | ✅ (host/label/paths/tags) |
| `--latest`                 | ✅                   | ❌                         |
| `--long`                   | ❌                   | ✅                         |

### `ls`

| option                                  | `restic` | `rustic` |
| --------------------------------------- | -------- | -------- |
| snapshot filtering options for `latest` | ✅       | ✅       |
| `--glob`                                | ❌       | ✅       |
| `--glob-file`                           | ❌       | ✅       |
| `--human-readable`                      | ✅       | ❌       |
| `--iglob`                               | ❌       | ✅       |
| `--iglob-file`                          | ❌       | ✅       |
| `--long`                                | ✅       | ✅       |
| `--numeric-uid-gid`                     | ❌       | ✅       |
| `--summary`                             | ❌       | ✅       |
| `--recursive`                           | ✅       | ✅       |

### `find`

| general                                               | `restic` | `rustic` |
| ----------------------------------------------------- | -------- | -------- |
| group and sort snapshots by date before finding       | ❌       | ✅       |
| summarize snapshots with identical result (like `+3`) | ❌       | ✅       |
| fast searching for given full paths                   | ❌       | ✅       |

| option                         | `restic`                   | `rustic`                  |
| ------------------------------ | -------------------------- | ------------------------- |
| `--all`                        | ❌ (no summarizing)        | ✅                        |
| `--blob`                       | ✅                         | ❌                        |
| `--glob`                       | ✅ (give patterns as args) | ✅                        |
| `--group-by`                   | ❌                         | ✅                        |
| `--ignore-case`                | ✅                         | ✅ (`--iglob`)            |
| `--long`                       | ✅                         | ❌ (default: long output) |
| `--newest`                     | ✅                         | ✅ use `--filter-before`  |
| `--numeric-uid-gid`            | ❌ (default: numeric ids)  | ✅                        |
| `--oldest`                     | ✅                         | ✅ use `--filter-after`   |
| `--pack`                       | ✅                         | ❌                        |
| `--path` (filter snapshots)    | ✅                         | ✅ `--filter-path`        |
| `--path` (full path to search) | ❌                         | ✅                        |
| `--show-pack-id`               | ✅                         | ❌                        |
| `--show-misses`                | ❌                         | ✅                        |
| `--snapshot`                   | ✅                         | ✅ (give ids as args)     |
| `--tag`                        | ✅                         | ✅ `--filter-tags`        |
| `--tree`                       | ✅                         | ❌                        |

### `diff`

| general                                | `restic` | `rustic` |
| -------------------------------------- | -------- | -------- |
| allow `latest`                         | ❌       | ✅       |
| diff with local files                  | ❌       | ✅       |
| `<snapshotID>:<subfolder>` syntax      | ✅       | ✅       |
| `<snapshotID>:<subfolder>/file` syntax | ❌       | ✅       |

| option                                  | `restic`                      | `rustic` |
| --------------------------------------- | ----------------------------- | -------- |
| snapshot filtering options for `latest` | ❌ (no `latest` support)      | ✅       |
| `--glob`                                | ❌                            | ✅       |
| `--glob-file`                           | ❌                            | ✅       |
| `--iglob`                               | ❌                            | ✅       |
| `--iglob-file`                          | ❌                            | ✅       |
| `--metadata`                            | ✅                            | ✅       |
| `--no-content`                          | ❌                            | ✅       |
| exclude options for local files         | ❌ (no diff with local files) | ✅       |

## External Links

[simple comparison of borg, restic and rustic blogpost](https://archive.ph/So9vG).
