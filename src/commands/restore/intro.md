# Restore - Restoring from backup

Restoring from a snapshot is as easy as it sounds, just use the following
command to restore the contents of the latest snapshot to `/tmp/restore-work`:

```console
$ rustic restore 79766175 /tmp/restore-work
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

Use the word `latest` to restore the last backup. You can also combine `latest`
with the `--filter-host` and `--filter-path` filters to choose the last backup
for a specific host, path or both.

```console
$ rustic restore latest /tmp/restore-art --filter-path "/home/art" --filter-host luigi
enter password for repository:
restoring <Snapshot of [/home/art] at 2015-05-08 21:45:17.884408621 +0200 CEST> to /tmp/restore-art
```

**Note**: When restoring specific paths, it's recommended to use the source path
syntax `latest:/path/to/restore/` followed by the destination path.

For example:

```console
$ rustic restore latest:/home/user/bin/ /home/user/bin/
```

This is more efficient as it only processes the specified subtree in the
snapshot and destination, rather than scanning the entire backup or destination
folder.

<!-- TODO: RESTORE USING GLOB NOT IMPLEMENTED YET

Use `--glob` (pattern to exclude/include (can be specified multiple times)) to
restrict the restore to a subset of files in the snapshot. For example, to
restore a single file:

```console
$ rustic restore 79766175 /tmp/restore-work --glob /work/foo
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

This will restore the file `foo` to `/tmp/restore-work/work/foo`. -->

You can use the command `rustic ls latest` to find the path to the file within
the snapshot.

<!-- TODO: or `rustic find foo` (rustic find not implemented yet) -->

<!-- TODO: RESTORE USING GLOB NOT IMPLEMENTED YET

This path you can then pass to
`--glob` in verbatim to only restore the single file or directory.

There is case insensitive variants of `--glob` called `--iglob`. This option
will behave the same way but ignore the casing of paths. -->

## Important Notes

- Restores the permissions of the files and directories to the state they were
  in when the snapshot was taken
- GUIDs and UIDs will be restored as well (this requires elevated privileges to
  work correctly)
- The glob pattern feature (`--glob`) is not yet fully implemented for scanning
  restrictions
- Always include trailing slashes for directory paths to ensure consistent
  behavior
- The source path should be relative to the snapshot's root
- The destination path should be an absolute or relative path on your local
  system

## Troubleshooting

If you're experiencing slow restore operations, check if you're:

1. Using glob patterns instead of direct paths
2. Targeting the root directory (/) unnecessarily
3. Missing trailing slashes in your directory paths
