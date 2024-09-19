# Cold storage

Rustic supports to store the repository in a so-called cold storage. These are
storages which are designed for long-term storage and offer usually cheap
storage for the price of retarded or expensive access. Examples are Amazon S3
Glacier or OVH Cloud Archive.

To use a cold storage and not access any data in the storage for every-day
operations, rustic needs an extra repository to store hot data. This repository
can be specified by the `hot-repo` option or the `RUSTIC_REPO_HOT` environmental
variable, e.g.:

```toml
[repository]
repository = "rclone:foo:cold-repo"
repo-hot = "rclone:foo:hot-repo"
```

or, as CLI-arguments

```console
rustic -r rclone:foo:cold-repo --repo-hot rclone:foo:hot-repo init
```

In this example in the repository `rclone:foo:cold-repo` all data is saved. In
the repository `rclone:foo:hot-repo` only hot data is saved, i.e. this is not a
complete repository.

**Warning**: You have to specify both the cold repository and the hot repository
in the `init` command and all other commands which access and work with the
repository. Best, you use a config profile to set both within.
