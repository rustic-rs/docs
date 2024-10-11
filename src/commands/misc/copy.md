# Copying snapshots between repositories

In case you want to transfer snapshots between two repositories, for example
from a local to a remote repository, you can use the `copy` command:

```console
$ rustic copy --target target-profile
repository d6504c63 opened successfully, password is correct
repository 3dd0878c opened successfully, password is correct

snapshot 410b18a2 of [/home/user/work] at 2020-06-09 23:15:57.305305 +0200 CEST)
    copy started, this may take a while...
snapshot 7a746a07 saved

snapshot 4e5d5487 of [/home/user/work] at 2020-05-01 22:44:07.012113 +0200 CEST)
skipping snapshot 4e5d5487, was already copied to snapshot 50eb62b7
```

The example command copies all snapshots from the source repository
`/srv/rustic-repo` to the destination repository `/srv/rustic-repo-copy`.
Snapshots which have previously been copied between repositories will be skipped
by later copy runs.

**Important**: This process will have to both download (read) and upload (write)
the entire snapshot(s) due to the different encryption keys used in the source
and destination repository. This *may incur higher bandwidth usage and costs*
than expected during normal backup runs.

## Copying to a repository with different chunker parameters

Instead of pre-initializing the repos you want to copy snapshots into, just use
`rustic copy --init` for the first run - this will set the correct chunker
parameters.

**Note**: rustic currently refuses to copy to a repository with different
chunker parameters. If you want to copy to a repository with different chunker
parameters, you have to set the chunker parameters of the target repository to
the same values as the source repository.
