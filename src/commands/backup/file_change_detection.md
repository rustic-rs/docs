# File change detection

When rustic encounters a file that has already been backed up, whether in the
current backup or a previous one, it makes sure the file's contents are only
stored once in the repository. To do so, it normally has to scan the entire
contents of every file. Because this can be very expensive, rustic also uses a
change detection rule based on file metadata to determine whether a file is
likely unchanged since a previous backup. If it is, the file is not scanned
again.

Change detection is only performed for regular files (not special files,
symlinks or directories) that have the exact same path as they did in a previous
backup of the same location. If a file or one of its containing directories was
renamed, it is considered a different file and its entire contents will be
scanned again.

Metadata changes (permissions, ownership, etc.) are always included in the
backup, even if file contents are considered unchanged.

On **Unix** (including Linux and Mac), given that a file lives at the same
location as a file in a previous backup, the following file metadata attributes
have to match for its contents to be presumed unchanged:

- Modification timestamp (mtime).
- Metadata change timestamp (ctime).
- File size.
- Inode number (internal number used to reference a file in a filesystem).

The reason for requiring both mtime and ctime to match is that Unix programs can
freely change mtime (and some do). In such cases, a ctime change may be the only
hint that a file did change.

The following `rustic backup` command line flags modify the change detection
rules:

- `--force`: turn off change detection and rescan all files.
- `--ignore-ctime`: require mtime to match, but allow ctime to differ.
- `--ignore-inode`: require mtime to match, but allow inode number and ctime to
  differ.

The option `--ignore-inode` exists to support FUSE-based filesystems and pCloud,
which do not assign stable inodes to files.

Note that the device id of the containing mount point is never taken into
account. Device numbers are not stable for removable devices and ZFS datasets.
If you want to force a re-scan in such a case, you can change the mountpoint.

The device id will still be tracked and stored as part of the backup,
unless you supply `--ignore-devid`.
