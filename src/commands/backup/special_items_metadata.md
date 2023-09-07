# Backup special items and metadata

**Symlinks** are archived as symlinks, `rustic` does not follow them. When you
restore, you get the same symlink again, with the same link target and the same
timestamps.

If there is a **bind-mount** below a directory that is to be saved, rustic
descends into it.

**Device files** are saved and restored as device files. This means that e.g.
`/dev/sda` is archived as a block device file and restored as such. This also
means that the content of the corresponding disk is not read, at least not from
the device file.

By default, rustic does not save the access time (atime) for any files or other
items, since it is not possible to reliably disable updating the access time by
rustic itself. This means that for each new backup a lot of metadata is written,
and the next backup needs to write new metadata again. If you really want to
save the access time for files and directories, you can pass the `--with-atime`
option to the `backup` command.

Note that `rustic` does not back up some metadata associated with files. Of
particular note are::

- file creation date on Unix platforms
- inode flags on Unix platforms
- xattr information
