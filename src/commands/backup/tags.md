# Tags for backup

Snapshots can have one or more tags, short strings which add identifying
information. Just specify the tags for a snapshot one by one with `--tag`:

```console
$ rustic -r /srv/rustic-repo backup --tag projectX --tag foo --tag bar ~/work
[...]
```

The tags can later be used to keep (or forget) snapshots with the `forget`
command. The command `tag` can be used to modify tags on an existing snapshot.
