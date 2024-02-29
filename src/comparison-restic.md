# Comparison between `rustic` and `restic`

## General differences

|                      | restic                                            | rustic                                             |
| -------------------- | ------------------------------------------------- | -------------------------------------------------- |
| programming language | Go                                                | Rust                                               |
| config file support  | ❌                                                | ✅                                                 |
| locking              | lock files in repository                          | lock-free operations                               |
| cold storage         | ❌ (no direct support, may work in special cases) | ✅ (full support including warm-up of needed data) |
| in-repo config       | ❌                                                | ✅                                                 |
| logging              | `-v` or `--quiet`, no log-file support            | `--log-level`, supports log-file output            |
| return code on error | ✅                                                | ❌                                                 |

## Storage backends

| : backend :            | restic         | rustic                                  |
| ---------------------- | -------------- | --------------------------------------- |
| local                  | ✅             | ✅                                      |
| sftp                   | ✅             | ✅ (via opendal, windows not supported) |
| rest                   | ✅             | ✅                                      |
| s3                     | ✅             | ✅ (via opendal)                        |
| swift                  | ✅             | ✅ (via opendal)                        |
| b2                     | ✅             | ✅ (via opendal)                        |
| azure                  | ✅             | ✅ (via opendal)                        |
| gs                     | ✅             | ✅ (via opendal)                        |
| dropbox                | ❌             | ✅ (via opendal)                        |
| ftp                    | ❌             | ✅ (via opendal)                        |
| gdrive                 | ❌             | ✅ (via opendal)                        |
| onedrive               | ❌             | ✅ (via opendal)                        |
| webdav                 | ❌             | ✅ (via opendal)                        |
| other opendal services | ❌             | ✅ (via opendal)                        |
| rclone                 | ✅ (via stdin) | ✅ (via http on localhost)              |

## Commands

| : command :      | restic                 | rustic                                   |
| ---------------- | ---------------------- | ---------------------------------------- |
| backup           | ✅                     | ✅                                       |
| cache            | ✅                     | ❌                                       |
| cat              | ✅                     | ✅                                       |
| config           | ❌ (no in-repo config) | ✅                                       |
| check            | ✅                     | ✅                                       |
| copy             | ✅                     | ✅                                       |
| diff             | ✅                     | ✅                                       |
| dump             | ✅                     | ✅                                       |
| find             | ✅                     | ❌                                       |
| forget           | ✅                     | ✅                                       |
| generate         | ✅                     | ✅ completions                           |
| init             | ✅                     | ✅                                       |
| key list         | ✅                     | ❌                                       |
| key add          | ✅                     | ✅                                       |
| key remove       | ✅                     | ❌                                       |
| key passwd       | ✅                     | ❌                                       |
| list             | ✅                     | ✅                                       |
| ls               | ✅                     | ✅                                       |
| merge            | ❌                     | ✅                                       |
| migrate          | ✅                     | ❌ (repo version migration via `config`) |
| mount            | ✅                     | ❌ (WIP)                                 |
| prune            | ✅                     | ✅                                       |
| recover          | ✅                     | ❌                                       |
| repair index     | ✅                     | ✅                                       |
| repair packs     | ✅                     | ❌                                       |
| repair snapshots | ✅                     | ✅                                       |
| repoinfo         | ❌                     | ✅                                       |
| restore          | ✅                     | ✅                                       |
| rewrite          | ✅                     | ❌                                       |
| self-update      | ✅                     | ✅                                       |
| show-config      | ❌                     | ✅                                       |
| snapshots        | ✅                     | ✅                                       |
| stats            | ✅                     | ❌ (but there is `repoinfo`)             |
| tag              | ✅                     | ✅                                       |
| unlock           | ✅                     | lock-free                                |
| webdav           | ❌                     | ✅                                       |

## Information saved in snapshots

| : information :                                                                                | restic   | rustic |
| ---------------------------------------------------------------------------------------------- | -------- | ------ |
| [from repo design info](https://github.com/restic/restic/blob/master/doc/design.rst#snapshots) | ✅       | ✅     |
| program version used                                                                           | ✅       | ✅     |
| summary (size,...)                                                                             | ❌ (WIP) | ✅     |
| used command                                                                                   | ❌       | ✅     |
| label                                                                                          | ❌       | ✅     |
| description                                                                                    | ❌       | ✅     |
| delete (protection)                                                                            | ❌       | ✅     |

## General options

| : option :                   | restic                                        | rustic                                                 |
| ---------------------------- | --------------------------------------------- | ------------------------------------------------------ |
| --cacert file                | ✅                                            | ❌                                                     |
| --cache-dir                  | ✅                                            | ✅ (or in config file)                                 |
| --cleanup-cache              | ✅                                            | ❌                                                     |
| --compression                | (auto,max,off); needed in every call          | ✅ (-7..22) configure once in in-repo config           |
| --dry-run                    | ✅                                            | ✅ (or in config file)                                 |
| --json                       | ✅                                            | ✅                                                     |
| --key-hint                   | ✅                                            | ❌                                                     |
| --limit-download             | ✅                                            | ❌                                                     |
| --limit-upload               | ✅                                            | ❌                                                     |
| --log-file                   | ❌                                            | ✅ (or in config file)                                 |
| --no-cache                   | ✅                                            | ✅ (or in config file)                                 |
| --no-extra-verify            | ✅ needed in every call                       | ✅ configure once using `config`                       |
| --no-lock                    | ✅                                            | lock-free                                              |
| --no-progress                | ❌                                            | ✅                                                     |
| --progress-intervall         | ✅ (via env variable)                         | ✅ (or in config file)                                 |
| --option                     | as cmd arg or env variable                    | via config file or env variable                        |
| --pack-size                  | fix limit, max. 128 MiB; needed in every call | fix or dynamic limit, configure once in in-repo config |
| --password-command           | ✅                                            | ✅ (or in config file)                                 |
| --password-file              | ✅                                            | ✅ (or in config file)                                 |
| --quiet                      | ✅                                            | ✅ (or in config file)                                 |
| --repo                       | ✅                                            | ✅ (or in config file)                                 |
| --repo-hot                   | ❌ (no cold-storage support)                  | ✅ (or in config file)                                 |
| --repository-file            | ✅                                            | ❌ (use config file)                                   |
| --retry-lock                 | ✅                                            | lock-free                                              |
| --tls-client-cert            | ✅                                            | ❌                                                     |
| --use-profile                | ❌ (no config file support)                   | ✅ (or in config file for recursively using profiles)  |
| `--verbose` (multiple times) | `--log-level`                                 |                                                        |
| --warm-up                    | ❌ (no cold-storage support)                  | ✅ (or in config file)                                 |

## Rustic in-repo config options

| : option :                    | restic                                                | rustic       |
| ----------------------------- | ----------------------------------------------------- | ------------ |
| append_only                   | ❌                                                    | ✅           |
| compression                   | ✅ (by `--compression`)                               | ✅           |
| treepack_size                 | ❌ (only all packs: --pack-size)                      | ✅           |
| treepack_growfactor           | ❌                                                    | ✅           |
| treepack_size_limit           | ❌                                                    | ✅           |
| datapack_size                 | ❌ (only all packs: --pack-size)                      | ✅           |
| datapack_growfactor           | ❌                                                    | ✅           |
| datapack_size_limit           | ❌                                                    | ✅           |
| min_packsize_tolerate_percent | ❌ (hardcoded 80% for `prune --repack-small`)         | ✅           |
| max_packsize_tolerate_percent | ❌                                                    | ✅           |
| extra_verify                  | ✅ (default, can be unset using ``--no-extra-verify`) | ✅ (default) |

## Snapshot filtering

| : filter : | restic       | rustic (also in config file)                      |
| ---------- | ------------ | ------------------------------------------------- |
| by host    | ✅ `--host`  | ✅ `--filter-host`                                |
| by label   | ❌           | ✅ `--filter-label`                               |
| by paths   | ✅ `--paths` | ✅ `--filter-paths`                               |
| by tags    | ✅ `--tags`  | ✅ `--filter-tags`                                |
| custom     | ❌           | ✅ `--filter-fn` (using [Rhai](https://rhai.rs/)) |

## Comparison of important commands

### `init`

| : option :            | restic                         | rustic                               |
| --------------------- | ------------------------------ | ------------------------------------ |
| --copy-chunker-params | ✅                             | ✅ (by using `copy --init`)          |
| --from-*              | ✅                             | ❌ (not needed, see `copy` command)) |
| --hostname            | ❌                             | ✅                                   |
| --repository-version  | ✅                             | ✅ (use `--set-version`)             |
| --set-*               | ❌ (no in-repo config support) | ✅                                   |
| --with-created        | ❌                             | ✅                                   |

### `backup`

| : general :                    | restic | rustic |
| ------------------------------ | ------ | ------ |
| allow to backup relative paths | ❌     | ✅     |

| : option :            | restic                            | rustic (also in config file)    |
| --------------------- | --------------------------------- | ------------------------------- |
| --as-path             | ❌                                | ✅                              |
| --command             | ❌                                | ✅                              |
| --custom-ignorefile   | ❌                                | ✅                              |
| --description         | ❌                                | ✅                              |
| --description-from    | ❌                                | ✅                              |
| --delete-never        | ❌                                | ✅                              |
| --delete-after        | ❌                                | ✅                              |
| --exclude             | ✅                                | ✅ --glob                       |
| --exclude-file        | ✅                                | ✅ -glob-file                   |
| --exclude-caches      | ✅                                | ❌ (use `--exclude-if-present`) |
| --exclude-if-present  | ✅ (+ support for header parsing) | ✅                              |
| --exclude-larger-than | ✅                                | ✅                              |
| --files-from          | ✅                                | ❌                              |
| --files-from-raw      | ✅                                | ❌                              |
| --files-from-verbatim | ✅                                | ❌                              |
| --force               | ✅                                | ✅                              |
| --git-ignore          | ❌                                | ✅                              |
| --group-by            | ✅ (host/paths/tags)              | ✅ (host/label/paths/tags)      |
| --host                | ✅                                | ✅                              |
| --iexclude            | ✅                                | ✅ --iglob                      |
| --iexclude-file       | ✅                                | ✅ --iglob-file                 |
| --ignore-ctime        | ✅                                | ✅                              |
| --ignore-inode        | ✅                                | ✅                              |
| --ignore-devid        | ❌                                | ✅                              |
| --init                | ❌                                | ✅                              |
| --label               | ❌                                | ✅                              |
| --no-require-git      | ❌ (no `--git-ignore`)            | ✅                              |
| --no-scan             | ✅                                | ✅                              |
| --one-file-system     | ✅                                | ✅                              |
| --parent              | ✅                                | ✅                              |
| --read-concurrency    | ✅                                | ❌ (hardcoded)                  |
| -skip-identical-paren | ❌                                | ✅                              |
| --stdin               | ✅                                | ✅ (using `-` as backup source) |
| --stdin-filename      | ✅                                | ✅                              |
| --tag                 | ✅                                | ✅                              |
| --time                | ✅                                | ✅                              |
| --with-atime          | ✅                                | ✅                              |

### `restore`

| : general :                            | restic | rustic |
| -------------------------------------- | ------ | ------ |
| scan and use already existing files    | ❌     | ✅     |
| resumable restore                      | ❌     | ✅     |
| restore hard links                     | ✅     | ❌     |
| "<snapshotID>:<subfolder>" syntax      | ✅     | ✅     |
| "<snapshotID>:<subfolder>/file" syntax | ❌     | ✅     |

| : option :                     | restic                             | rustic                                      |
| ------------------------------ | ---------------------------------- | ------------------------------------------- |
| filtering options for "latest" | ✅                                 | ✅                                          |
| --delete                       | ❌                                 | ✅                                          |
| --exclude                      | ✅                                 | ✅ --glob                                   |
| --iexclude                     | ✅                                 | ✅ --iglob                                  |
| --iinclude                     | ✅                                 | ✅ --iglob                                  |
| --include                      | ✅                                 | ✅ --glob                                   |
| --no-ownership                 | ❌                                 | ✅                                          |
| --numeric-id                   | ❌                                 | ✅                                          |
| --sparse                       | ✅                                 | ❌                                          |
| --target                       | ✅                                 | ✅ (give target as second CLI argument)     |
| --verify                       | ✅                                 | ❌ (but `diff` can be used to verify after) |
| --verify-existing              | ❌ (no scanning of existing files) | ✅                                          |

### `dump`

| : general : | restic | rustic |
| ----------- | ------ | ------ |
| dump files  | ✅     | ✅     |
| dump dirs   | ✅     | ❌     |

| : option :                              | restic | rustic                  |
| --------------------------------------- | ------ | ----------------------- |
| snapshot filtering options for "latest" | ✅     | ✅                      |
| --archive                               | ✅     | ❌ (no dumping of dirs) |

### `forget`

| : general :                             | restic | rustic |
| --------------------------------------- | ------ | ------ |
| allow to keep all XXX                   | ✅     | ✅     |
| respect "no delete" options in snapshot | ❌     | ✅     |

| : option :                   | restic               | rustic (also in config file) |
| ---------------------------- | -------------------- | ---------------------------- |
| snapshot filtering options   | ✅                   | ✅                           |
| --keep-last                  | ✅                   | ✅                           |
| --keep-daily                 | ✅                   | ✅                           |
| --keep-weekly                | ✅                   | ✅                           |
| --keep-monthly               | ✅                   | ✅                           |
| --keep-quarter-yearly        | ❌                   | ✅                           |
| --keep-half-yearly           | ❌                   | ✅                           |
| --keep-yearly                | ✅                   | ✅                           |
| --keep-within                | ✅                   | ✅                           |
| --keep-within-hourly         | ✅                   | ✅                           |
| --keep-within-daily          | ✅                   | ✅                           |
| --keep-within-weekly         | ✅                   | ✅                           |
| --keep-within-monthly        | ✅                   | ✅                           |
| --keep-within-quarter-yearly | ❌                   | ✅                           |
| --keep-within-half-yearly    | ❌                   | ✅                           |
| --keep-within-yearly         | ✅                   | ✅                           |
| --keep-tag                   | ✅                   | ✅                           |
| --compact                    | ✅                   | ❌                           |
| --group-by                   | ✅ (host/paths/tags) | ✅ (host/label/paths/tags)   |
| --prune                      | ✅                   | ✅                           |

### `prune`

| : general :                                | restic | rustic |
| ------------------------------------------ | ------ | ------ |
| prune plan without reading pack files      | ✅     | ✅     |
| prune parallel to backup (two-phase prune) | ❌     | ✅     |
| different sizes for tree and data packs    | ❌     | ✅     |
| (option to) resize packs                   | ✅     | ✅     |

| : option :                     | restic                     | rustic                                            |
| ------------------------------ | -------------------------- | ------------------------------------------------- |
| --fast-repack                  | ❌                         | ✅                                                |
| --instant-delete               | ✅ (default, no two-phase) | ✅                                                |
| --keep-pack                    | ❌                         | ✅                                                |
| --keep-delete                  | ❌                         | ✅                                                |
| --max-repack-size              | ✅                         | ✅ --max-repack (size/%/unlimited)                |
| --max-unused                   | ✅                         | ✅                                                |
| --repack-all                   | ❌                         | ✅                                                |
| --repack-cacheable-only        | ✅                         | ✅                                                |
| --repack-small                 | ✅                         | ✅ (default behavior; to unset use `--no-resize`) |
| --repack-uncompressed          | ✅                         | ✅                                                |
| --unsafe-recover-no-free-space | ✅                         | ✅ --early-delete-index                           |

### `check`

| : general :                   | restic                       | rustic       |
| ----------------------------- | ---------------------------- | ------------ |
| check index files             | ✅                           | ✅           |
| check index vs packs          | ✅                           | ✅           |
| check snapshot files          | ✅                           | ✅           |
| (optionally) check pack files | ✅                           | ✅           |
| cache policy                  | create temporary             | use existing |
| check cache integrity         | ❌                           | ✅           |
| check hot/cold integrity      | ❌ (no cold storage support) | ✅           |

| : option :         | restic                        | rustic                |
| ------------------ | ----------------------------- | --------------------- |
| --read-data        | ✅                            | ✅                    |
| --read-data-subset | ✅                            | ❌                    |
| --trust-cache      | ❌ (no cache integrity check) | ✅                    |
| --with-cache       | ✅                            | ✅ (default behavior) |

### `copy`

| : general :                           | restic      | rustic         |
| ------------------------------------- | ----------- | -------------- |
| source/destination given by           | CLI options | in config file |
| multiple destinations                 | ❌          | ✅             |
| check for matching chunker parameters | ❌          | ✅             |

| : option :                 | restic | rustic                              |
| -------------------------- | ------ | ----------------------------------- |
| snapshot filtering options | ✅     | ✅                                  |
| --from-*                   | ✅     | ❌ (destiniation(s) in config file) |
| --init                     | ❌     | ✅                                  |

### `snapshots`

| : general :                               | restic   | rustic |
| ----------------------------------------- | -------- | ------ |
| summarize identical snapshots (like "+3") | ❌       | ✅     |
| show summary information (sizes)          | ❌ (WIP) | ✅     |

| : option :                 | restic               | rustic                     |
| -------------------------- | -------------------- | -------------------------- |
| snapshot filtering options | ✅                   | ✅                         |
| --all                      | ❌                   | ✅                         |
| --compact                  | ✅                   | ❌                         |
| --group-by                 | ✅ (host/paths/tags) | ✅ (host/label/paths/tags) |
| --latest                   | ✅                   | ❌                         |
| --long                     | ❌                   | ✅                         |

### `ls`

| : option :                              | restic | rustic |
| --------------------------------------- | ------ | ------ |
| snapshot filtering options for "latest" | ✅     | ✅     |
| --glob                                  | ❌     | ✅     |
| --glob-file                             | ❌     | ✅     |
| --human-readable                        | ✅     | ❌     |
| --iglob                                 | ❌     | ✅     |
| --iglob-file                            | ❌     | ✅     |
| --long                                  | ✅     | ✅     |
| --numeric-uid-gid                       | ❌     | ✅     |
| --summary                               | ❌     | ✅     |
| --recursive                             | ✅     | ✅     |

### `diff`

| : general :                            | restic | rustic |
| -------------------------------------- | ------ | ------ |
| allow "latest"                         | ❌     | ✅     |
| diff with local files                  | ❌     | ✅     |
| "<snapshotID>:<subfolder>" syntax      | ✅     | ✅     |
| "<snapshotID>:<subfolder>/file" syntax | ❌     | ✅     |

| : option :                              | restic                        | rustic |
| --------------------------------------- | ----------------------------- | ------ |
| snapshot filtering options for "latest" | ❌ (no "latest" support)      | ✅     |
| --glob                                  | ❌                            | ✅     |
| --glob-file                             | ❌                            | ✅     |
| --iglob                                 | ❌                            | ✅     |
| --iglob-file                            | ❌                            | ✅     |
| --metadata                              | ✅                            | ✅     |
| --no-content                            | ❌                            | ✅     |
| other exclude options                   | ❌ (no diff with local files) | ✅     |
