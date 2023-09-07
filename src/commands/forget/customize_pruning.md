# Customize pruning

To understand the custom options, we first explain how the pruning process
works:

1. All snapshots and directories within snapshots are scanned to determine which
   data is still in use.
2. For all files in the repository, rustic finds out if the file is fully used,
   partly used or completely unused.
3. Completely unused files are marked for deletion. Fully used files are kept. A
   partially used file is either kept or marked for repacking depending on user
   options.

   Note that for repacking, rustic must download the file from the repository
   storage and re-upload the needed data in the repository. This can be very
   time-consuming for remote repositories.
4. After deciding what to do, `prune` will actually perform the repack, modify
   the index according to the changes and delete the obsolete files.

The `prune` command accepts the following options:

- `--max-unused limit` allow unused data up to the specified limit within the
  repository. This allows rustic to keep partly used files instead of repacking
  them.

  The limit can be specified in several ways:

  - As an absolute size (e.g. `200M`). If you want to minimize the space used by
    your repository, pass `0` to this option.
  - As a size relative to the total repo size (e.g. `10%`). This means that
    after prune, at most `10%` of the total data stored in the repo may be
    unused data. If the repo after prune has a size of 500MB, then at most 50MB
    may be unused.
  - If the string `unlimited` is passed, there is no limit for partly unused
    files. This means that as long as some data is still used within a file
    stored in the repo, rustic will just leave it there. Use this if you want to
    minimize the time and bandwidth used by the `prune` operation. Note that
    metadata will still be repacked.

  rustic tries to repack as little data as possible while still ensuring this
  limit for unused data. The default value is 5%.

- `--max-repack-size size` if set limits the total size of files to repack. As
  `prune` first stores all repacked files and deletes the obsolete files at the
  end, this option might be handy if you expect many files to be repacked and
  fear to run low on storage.

- `--repack-cacheable-only` if set to true only files which contain metadata and
  would be stored in the cache are repacked. Other pack files are not repacked
  if this option is set. This allows a very fast repacking using only cached
  data. It can, however, imply that the unused data in your repository exceeds
  the value given by `--max-unused`. The default value is false.

- `--dry-run` only show what `prune` would do.

- `--verbose` increased verbosity shows additional statistics for `prune`.
