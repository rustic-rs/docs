# Restoring from backup

<!-- TOC -->

- [Restoring from a snapshot](#restoring-from-a-snapshot)
- [Restore using mount](#restore-using-mount)
- [Printing files to stdout](#printing-files-to-stdout)

<!-- /TOC -->

## Restoring from a snapshot

Restoring a snapshot is as easy as it sounds, just use the following command to
restore the contents of the latest snapshot to `/tmp/restore-work`:

```console
$ rustic -r /srv/rustic-repo restore 79766175 --target /tmp/restore-work
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

Use the word `latest` to restore the last backup. You can also combine `latest`
with the `--host` and `--path` filters to choose the last backup for a specific
host, path or both.

```console
$ rustic -r /srv/rustic-repo restore latest --target /tmp/restore-art --path "/home/art" --host luigi
enter password for repository:
restoring <Snapshot of [/home/art] at 2015-05-08 21:45:17.884408621 +0200 CEST> to /tmp/restore-art
```

Use `--exclude` and `--include` to restrict the restore to a subset of files in
the snapshot. For example, to restore a single file:

```console
$ rustic -r /srv/rustic-repo restore 79766175 --target /tmp/restore-work --include /work/foo
enter password for repository:
restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work
```

This will restore the file `foo` to `/tmp/restore-work/work/foo`.

You can use the command `rustic ls latest` or `rustic find foo` to find the path
to the file within the snapshot. This path you can then pass to `--include` in
verbatim to only restore the single file or directory.

There are case insensitive variants of `--exclude` and `--include` called
`--iexclude` and `--iinclude`. These options will behave the same way but ignore
the casing of paths.

## Restore using mount

Browsing your backup as a regular file system is also very easy. First, create a
mount point such as `/mnt/rustic` and then use the following command to serve
the repository with FUSE:

```console
$ mkdir /mnt/rustic
$ rustic -r /srv/rustic-repo mount /mnt/rustic
enter password for repository:
Now serving /srv/rustic-repo at /mnt/rustic
Use another terminal or tool to browse the contents of this folder.
When finished, quit with Ctrl-c here or umount the mountpoint.
```

Mounting repositories via FUSE is only possible on Linux, macOS and FreeBSD. On
Linux, the `fuse` kernel module needs to be loaded and the `fusermount` command
needs to be in the `PATH`. On macOS, you need
`FUSE for macOS <https://osxfuse.github.io/>`__. On FreeBSD, you may need to
install FUSE and load the kernel module (`kldload fuse`).

rustic supports storage and preservation of hard links. However, since hard
links exist in the scope of a filesystem by definition, restoring hard links
from a fuse mount should be done by a program that preserves hard links. A
program that does so is `rsync`, used with the option --hard-links.

## Printing files to stdout

Sometimes it's helpful to print files to stdout so that other programs can read
the data directly. This can be achieved by using the `dump` command, like this:

```console
rustic -r /srv/rustic-repo dump latest production.sql | mysql
```

If you have saved multiple different things into the same repo, the `latest`
snapshot may not be the right one. For example, consider the following snapshots
in a repo:

```console
$ rustic -r /srv/rustic-repo snapshots
ID        Date                 Host        Tags        Directory
----------------------------------------------------------------------
562bfc5e  2018-07-14 20:18:01  mopped                  /home/user/file1
bbacb625  2018-07-14 20:18:07  mopped                  /home/other/work
e922c858  2018-07-14 20:18:10  mopped                  /home/other/work
098db9d5  2018-07-14 20:18:13  mopped                  /production.sql
b62f46ec  2018-07-14 20:18:16  mopped                  /home/user/file1
1541acae  2018-07-14 20:18:18  mopped                  /home/other/work
----------------------------------------------------------------------
```

Here, rustic would resolve `latest` to the snapshot `1541acae`, which does not
contain the file we'd like to print at all (`production.sql`). In this case, you
can pass rustic the snapshot ID of the snapshot you like to restore:

```console
rustic -r /srv/rustic-repo dump 098db9d5 production.sql | mysql
```

Or you can pass rustic a path that should be used for selecting the latest
snapshot. The path must match the patch printed in the "Directory" column, e.g.:

```console
rustic -r /srv/rustic-repo dump --path /production.sql latest production.sql | mysql
```

It is also possible to `dump` the contents of a whole folder structure to
stdout. To retain the information about the files and folders rustic will output
the contents in the tar (default) or zip format:

```console
rustic -r /srv/rustic-repo dump latest /home/other/work > restore.tar
```

```console
rustic -r /srv/rustic-repo dump -a zip latest /home/other/work > restore.zip
```
