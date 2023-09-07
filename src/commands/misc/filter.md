# Filtering snapshots to copy

The list of snapshots to copy can be filtered by host, path in the backup and /
or a comma-separated tag list:

```console
rustic -r /srv/rustic-repo copy --repo2 /srv/rustic-repo-copy --host luigi --path /srv --tag foo,bar
```

It is also possible to explicitly specify the list of snapshots to copy, in
which case only these instead of all snapshots will be copied:

```console
rustic -r /srv/rustic-repo copy --repo2 /srv/rustic-repo-copy 410b18a2 4e5d5487 latest
```
