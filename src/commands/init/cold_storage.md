# Cold storage

Rustic supports to store the repository in a so-called cold storage. These are
storages which are design for long-time storage and offer usually cheap storage
for the price of retarded or expensive access. Examples are Amazon S3 Glacier or
OVH Cloud Archive.

To use a cold storage and not access any data in the storage for every-day
operations, rustic needs an extra repository to store hot data. This repository
can be specified by the `--hot-repo` option or the `RUSTIC_REPO_HOT`
environmental variable, e.g.:

```console
rustic -r rclone:foo:bar --repo-hot rclone:foo:bar init
```

In this example in the repository
``rclone:foo:bar``` all data is saved. In the repository``rclone:foo:bar-hot``
only hot data is saved, i.e. this is not a complete repository.

**Warning**: You have to specify both the cold repository (using `-r`) and the
hot repository (using `--repo-hot`) in the `init` command and all other commands
which access and work with the repository.
