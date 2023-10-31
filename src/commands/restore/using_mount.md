# Restore using mount

**NOTE**: `rustic` doesn't support `mount` at this point, please use `restic`
to invoke this operation for the time being. We are working on a `mount`
implementation for `rustic`.

Browsing your backup as a regular file system is also very easy. First, create a
mount point such as `/mnt/rustic` and then use the following command to serve
the repository with FUSE:

```console
$ mkdir /mnt/rustic
$ restic -r /srv/rustic-repo mount /mnt/rustic
enter password for repository:
Now serving /srv/rustic-repo at /mnt/rustic
Use another terminal or tool to browse the contents of this folder.
When finished, quit with Ctrl-c here or umount the mountpoint.
```

Mounting repositories via FUSE is only possible on Linux, macOS and FreeBSD. On
Linux, the `fuse` kernel module needs to be loaded and the `fusermount` command
needs to be in the `PATH`. On macOS, you need
[FUSE for macOS](https://osxfuse.github.io/). On FreeBSD, you may need to
install FUSE and load the kernel module (`kldload fuse`).

<!-- TODO!:
rustic supports storage and preservation of hard links. However, since hard
links exist in the scope of a filesystem by definition, restoring hard links
from a fuse mount should be done by a program that preserves hard links. A
program that does so is `rsync`, used with the option --hard-links.
-->
