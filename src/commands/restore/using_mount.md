# Restore using mount

**NOTE**: `rustic` currently only supports `mount` for linux; we are working on
support for other operation systemss.

Browsing your backup as a regular file system is also very easy. First, create a
mount point such as `/mnt/rustic` and then use the following command to serve
the repository with FUSE:

```console
$ mkdir /mnt/rustic
$ rustic mount /mnt/rustic
```

Now you can access the backup content like a normal filesystem, i.e. you can
simply copy files from the respository snapshots.

**NOTE**: Copying files using `mount` may be not as efficient as using `restore`
as `restore` is highly optimized and knows in the beginning what exactly will be
restored.

**NOTE** Keep in mind that every file access on the mounted repository may
involve access to the repository. Especially read access to lots of data may be
expensive depending on the backend. It is not advised to run compare tools or
something like `rsync` on the mounted dir if you use a remote backend. Use
`diff` or `restore` if you want to compare or sync content with your lokal saved
files.

**NOTE**: When using a hot/cold repositories, file access is only possible if
the needed data is warmed-up in the cold repository part. By default rustic
therefore forbids file-access by default, see `--file-access` below.

There are various options which can be used with the `mount` command:

- You can specify the exact snapshot/path to mount, e.g.
  `rustic mount /mnt/rustic latest:/home`
- If no snapshot/path is given, all snapshots are displayed in a tree, you can
  use `--path-template` and `--time-template` to define the tree structure, e.g.
  first group by hostname and then by snapshot date/time.
- You can give additional mount options using `--exclusive` or `--option`.
- `--file-access` allows to restict access to only listing dirs and.

<!-- TODO!:
Mounting repositories via FUSE is only possible on Linux, macOS and FreeBSD. On
Linux, the `fuse` kernel module needs to be loaded and the `fusermount` command
needs to be in the `PATH`. On macOS, you need
[FUSE for macOS](https://osxfuse.github.io/). On FreeBSD, you may need to
install FUSE and load the kernel module (`kldload fuse`).

rustic supports storage and preservation of hard links. However, since hard
links exist in the scope of a filesystem by definition, restoring hard links
from a fuse mount should be done by a program that preserves hard links. A
program that does so is `rsync`, used with the option --hard-links.
-->
