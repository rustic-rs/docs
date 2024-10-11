# Miscellaneous - Working with repositories

A repository is a storage location for all of your snapshots.

The repository is created with the `init` command:

```console
$ rustic init
enter password for new repository:
enter password again:
created rustic repository 7a8c3b2a0c at /srv/rustic-repo
Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

The repository is now ready for use.

**Note**: You can use `latest` to select the last snapshot for a command. For
example, `rustic snapshots latest` will show the last snapshot.
