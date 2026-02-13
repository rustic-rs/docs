# Backup block devices

Special files like block devices or pipes are by default stored by only the
metadata of the special file like a device number. This allows to restore
something like `/dev` on Unix platforms and to get the original state.

Sometimes, however, you want to backup the contents of block device, e.g. a
snapshot of a disc. To do so, rustic supports the `set-devid` options. You can
define it in the config file:

```toml
[[backup.snapshots]]
sources = ["/dev/my-blockdev"]
set-blockdev = "file"
```

or directly on the command line

```console
rustic backup --set-blockdev=file /dev/my-blockdev
```

Note that for blockdevices, a fixed sized chunking with chunksize being a
multiple of the block size is recommended as this speeds-up the backup from
large single files and also good chunking points are typically on block
boundaries for block devices. Fixed size chunking can be set e.g. during
repository initalization, for example:

```console
rustic init --set-chunker=fixed_size --set-chunk-size=1MiB
```
