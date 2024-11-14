# Excluding Files

You can exclude folders and files by specifying exclude patterns, currently the
exclude options are:

- `--git-ignore` Respect `.gitignore` files and exclude paths/files not handled
  by git.
- `--glob` include/exclude files and dirs based on given glob patterns
- `--iglob` Same as `--glob` but ignores the case of paths
- `--glob-file` Specified one or more times to exclude items listed in a given
  file
- `--iglob-file` Same as `--glob-file` but ignores cases like in `--iglob`
- `--exclude-if-present foo` Specified one or more times to exclude a folder's
  content if it contains a file called `foo`. For example, to exclude cache
  dirs, specify `--exclude-if-present CACHEDIR.TAG`.
- `--exclude-larger-than size` Specified once to excludes files larger than the
  given size

Please see `rustic help backup` for more specific information about each exclude
option.

Let's say we have a file called `glob.txt` with the following content:

```text
# exclude go-files
!*.go
# exclude foo/x/y/z/bar foo/x/bar foo/bar
!foo/**/bar
```

It can be used like this:

```console
rustic backup ~/work --glob="!*.c" --glob-file=glob.txt
```

This instructs rustic to exclude files matching the following criteria:

- All files matching `*.c` (parameter `--glob`)
- All files matching `*.go` (second line in `glob.txt`)
- All files and sub-directories named `bar` which reside somewhere below a
  directory called `foo` (fourth line in `glob.txt`)

By specifying the option `--one-file-system` you can instruct rustic to only
backup files from the file systems the initially specified files or directories
reside on. In other words, it will prevent rustic from crossing filesystem
boundaries and subvolumes when performing a backup.

For example, if you backup `/` with this option and you have external media
mounted under `/media/usb` then rustic will not back up `/media/usb` at all
because this is a different filesystem than `/`. Virtual filesystems such as
`/proc` are also considered different and thereby excluded when using
`--one-file-system`:

```console
rustic -r /srv/rustic-repo backup --one-file-system /
```

Please note that this does not prevent you from specifying multiple filesystems
on the command line, e.g:

```console
rustic backup --one-file-system / /media/usb
```

will back up both the `/` and `/media/usb` filesystems, but will not include
other filesystems like `/sys` and `/proc`.

**Note**: `--one-file-system` is currently unsupported on Windows, and will
cause the backup to immediately fail with an error.

Files larger than a given size can be excluded using the `--exclude-larger-than`
option:

```console
rustic backup ~/work --exclude-larger-than 1M
```

This excludes files in `~/work` which are larger than 1 MiB from the backup.

The default unit for the size value is bytes, so e.g.
`--exclude-larger-than 2048` would exclude files larger than 2048 bytes (2 KiB).
To specify other units, suffix the size value with one of `k`/`K` for KiB (1024
bytes), `m`/`M` for MiB (1024^2 bytes), `g`/`G` for GiB (1024^3 bytes) and
`t`/`T` for TiB (1024^4 bytes), e.g. `1k`, `10K`, `20m`, `20M`, `30g`, `30G`,
`2t` or `2T`).
