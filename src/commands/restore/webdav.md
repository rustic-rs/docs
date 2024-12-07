# Restore using webdav

Another possibility is browsing your backup as webdav share. This option can be
also used to mount the repository on either remote computers or locally if
`mount` is not supported for your operation system. To start a webdav server,
just run

```console
$ rustic webdav
```

Now you can connect using a standard webdav client.

**NOTE**: Copying files using `webdav` may be not as efficient as using
`restore` as `restore` is highly optimized and knows in the beginning what
exactly will be restored.

**NOTE** Keep in mind that every file access via webdav may involve access to
the repository. Especially read access to lots of data may be expensive
depending on the backend. It is not advised to run compare tools or something
like `rsync` if you mount the webdav in case you are use a remote backend. Use
`diff` or `restore` if you want to compare or sync repository content with each
other or with your local saved files.

**NOTE**: When using a hot/cold repositories, file access is only possible if
the needed data is warmed-up in the cold repository part. By default rustic
therefore forbids file-access by default, see `--file-access` below.

There are various options which can be used with the `webdav` command (most
similar to [mount](using_mount.md))

- You can specify the exact snapshot/path to mount, e.g.
  `rustic webdav latest:/home`
- If no snapshot/path is given, all snapshots are displayed in a tree, you can
  use `--path-template` and `--time-template` to define the tree structure, e.g.
  first group by hostname and then by snapshot date/time.
- `--address` allows you to give the bind address.
- `--symlinks` also enables symlinks.
- `--file-access` allows to restict access to only listing dirs and.
