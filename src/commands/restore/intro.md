# Restore - Restoring from backup

Restoring from a snapshot is as easy as it sounds, just use the following
command to restore the contents of the latest snapshot to `/tmp/restore-work`:

```console
$ rustic -r /srv/rustic-repo restore 79766175 /tmp/restore-work
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

Use the word `latest` to restore the last backup. You can also combine `latest`
with the `--filter-host` and `--filter-path` filters to choose the last backup for a specific
host, path or both.

```console
$ rustic -r /srv/rustic-repo restore latest /tmp/restore-art --filter-path "/home/art" --filter-host luigi
enter password for repository:
restoring <Snapshot of [/home/art] at 2015-05-08 21:45:17.884408621 +0200 CEST> to /tmp/restore-art
```

Use `--glob` (pattern to exclude/include (can be specified multiple times))
to restrict the restore to a subset of files in
the snapshot. For example, to restore a single file:

```console
$ rustic -r /srv/rustic-repo restore 79766175 /tmp/restore-work --glob /work/foo
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

This will restore the file `foo` to `/tmp/restore-work/work/foo`.

You can use the command `rustic ls latest`

<!-- TODO: or `rustic find foo` (rustic find not implemented yet) --> to find

the path to the file within the snapshot. This path you can then pass to
`--glob` in verbatim to only restore the single file or directory.

There is case insensitive variants of `--glob` called
`--iglob`. This option will behave the same way but ignore
the casing of paths.
