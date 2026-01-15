## Filtering snapshots

rustic allows to filter snapshots by certain criteria. This can be useful when
listing snapshots, copying snapshots between repositories.

Filters can by given in the config profile

```toml
filter-host = ["host2"] # Default: []
filter-label = ["label1"] # Default: []
filter-tags = ["tag1,tag2"] # Default: []
filter-paths = ["path1"] # Default: no paths filter
filter-fn = '|sn| {sn.host == "host1" || sn.description.contains("test")}' # Default: no filter function
```

or alternatively as CLI options `--filter-host`, `--filter-label`,
`--filter-tags`, `--filter-paths` and `--filter-fn`.

The `filter-host` option allows to filter snapshots by the host name. The
`filter-label` option allows to filter snapshots by the label. The `filter-tags`
option allows to filter snapshots by tags. The `filter-paths` option allows to
filter snapshots by the paths in the backup. The `filter-fn` option allows to
filter snapshots by a custom filter function. The filter function is a string
that is parsed as a Rust closure. The closure takes a `Snapshot` struct as
argument and returns a boolean value.

Multiple filter arguments are connected with an `or` condition, i.e. snapshots
are not filtered out if they match anything of what is the given in the array.
For instance, this shows snapshots having either a tag `tag1` or a tag `tag2`:

```toml
filter-tags = ["tag1", "tag2"]
```

Moreover the different `filter-*` options are combined using an `and`
condition - if a snapshot does not satisfy any of the filters, they are sorted
out. For example, to list snapshots that are tagged with `tag1` and `tag2` and
are located in the path `/srv`, use the following configuration:

```toml
filter-tags = ["tag1,tag2"]
filter-paths = ["/srv"]
```

**Note**: For paths and tags it is already possible to give multiple values -
and then they are `and`ed: `filter-tags = ["a,b"]` matches snapshots which have
both "a" and "b" as tags, whereas `filter-tags = ["a","b"]` matches snapshots
which have either "a" or "b" as tag.

Note that `filter-tags` and `filter-paths` also match snapshots which do have
additional tags or paths, respectively.

For example, a snapshot with tags "a,b,c" is currently shown when you specify
`filter-tags=["a,b"]`. If you want to get only snapshots with the exact list of
tags/paths, use `filter-tags-exact` and filter-paths-exact`.

## Filtering snapshots to copy

The list of snapshots to copy can be filtered by using the above described
`filter-*` options:

```console
rustic copy --filter-host luigi --filter-tags foo,bar
```

It is also possible to explicitly specify the list of snapshots to copy, in
which case only these instead of all snapshots will be copied:

```console
rustic copy 410b18a2 4e5d5487 latest
```
