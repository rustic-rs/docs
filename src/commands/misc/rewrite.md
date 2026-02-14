# Rewriting Snapshots

## Removing files

Sometimes you need to change an existing snapshot. Notably, if you discover it
includes unwanted files (e.g., cleartext passwords or large caches).

Instead of deleting the snapshot and then using the `backup` command again, you
may use the `rewrite` command to modify the existing snapshots.

### The long way

Let's imagine I have the following snapshots:

```console
# Initial state:
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 5109f5fe | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |      | Project |   509 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)
```

Unfortunately, the *Project/config/secret.env* file has sneaked in. First thing,
identify the snapshots containing that file:

```console
$ rustic find --glob '**/secret.env' --filter-paths Project

searching in snapshots group (host [kasimir], label [], paths [Project])...
found in 5109f5fe from 2026-02-02 13:17:52
-rw-r--r--     user     user        29  2 Feb 2026 13:17 "Project/config/secret.env"
found in ca466029 from 2026-02-02 13:17:57
-rw-r--r--     user     user        29  2 Feb 2026 13:17 "Project/config/secret.env"
```

**Note:** As an alternative to `rustic find`, you may directly use
`rustic rewrite -n` (`-n` is for dry-run)

```console
$ rustic rewrite -n --glob '!**/secret.env'
Would have rewritten the following snapshots:
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 5109f5fe | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
2 snapshot(s)
```

Apparently, two snapshots are concerned. Let's rewrite them to *exclude* that
file:

```console
$ rustic rewrite --glob '!**/secret.env' 5109f5fe ca466029
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags    | Paths   | Files | Dirs |      Size |
|----------|---------|-------|---------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |         | Project |   507 |   87 | 524.0 KiB |
| 337ff932 | kasimir |       | rewrite | Project |   507 |   87 | 524.0 KiB |
| 5109f5fe | kasimir |       |         | Project |   508 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |         | Project |   509 |   87 | 524.0 KiB |
| cf5416ee | kasimir |       | rewrite | Project |   508 |   87 | 524.0 KiB |
5 snapshot(s)

total: 5 snapshot(s)
```

**Note:** In the `rewrite` command, you must use a negative glob (starting with
an exclamation mark) to specify the files you want to remove.

We now have two *new* snapshots tagged `rewrite`: the updated snapshots with the
secret file removed. You may also notice that the snapshots `5109f5fe` and
`ca466029` remain. By default, the `rewrite` command does *not* remove the
source snapshots. So, in mycase, I will have to remove them manually:

```console
$ rustic forget 5109f5fe ca466029
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags    | Paths   | Files | Dirs |      Size |
|----------|---------|-------|---------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |         | Project |   507 |   87 | 524.0 KiB |
| 337ff932 | kasimir |       | rewrite | Project |   507 |   87 | 524.0 KiB |
| cf5416ee | kasimir |       | rewrite | Project |   508 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)
```

You can now check: no remaining snapshots contain the file we removed:

```console
$ rustic find --glob '**/secret.env' --filter-paths Project

searching in snapshots group (host [kasimir], label [], paths [Project])...
```

Finally, let's get rid of the `rewrite` tags (more on that later):

```console
$ rustic rewrite --forget --remove-tags 'rewrite' --filter-paths Project
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 4b0b7b90 | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 1d95e18e | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)
```

**Note:** The `rewrite` command never modifies an existing snapshot. It always
creates a new one. That's why the *untagged* snapshots above have a different ID
than the *tagged* ones.

### The short way

In the section above, we've outlined all the steps in detail. We especially had
to manually identify and then remove the unwanted snapshots. But using the `-n`
and `--forget` options, the `rewrite command` can do the job with fewer
keystrokes.

Let's resume at the moment we noticed the unwanted file in the backups. If you
remember, we had then three snapshots without knowing the ones that were
containing the `secret.env` file:

```console
# Initial state:
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 5109f5fe | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |      | Project |   509 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)
```

To identify the snapshots containing the `secret.env` file, we previously used
`rustic find`. But `rewrite` also has a *dry-run* option `-n`. It doesn't
perform any action on the repo, but it will show you the snapshot that would
have been affected:

```console
$ rustic rewrite -n --glob '!Project/config/secret.env' --filter-paths Project
Would have rewritten the following snapshots:
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 5109f5fe | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
2 snapshot(s)
```

And using the `--forget` option, we can do the `rewrite` and `forget` steps in
one command:

```console
$ rustic rewrite --forget --glob '!Project/config/secret.env' 5109f5fe ca466029
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| d0f5387f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 27a82e21 | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)
```

Interestingly, the new snapshots replaced the old ones but were not tagged. The
option `--forget` prevent the command from adding the `rewrite` tag. To be more
precise, there is the option `--tags-rewritten` which specifies which tags to
add to rewritten snapshots. It defaults to `rewrite` if no `--forget` is given
and to `''` (i.e. no tag) if `--forget` is used.

### The very short way

Well, as a matter of fact, if you know what you're doing, all can be done just
using `rewrite --forget`:

```console
# Initial state:
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 5109f5fe | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
| ca466029 | kasimir |       |      | Project |   509 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)

# Here is the rewrite command:
$ rustic rewrite --forget --glob '!Project/config/secret.env' --filter-paths Project
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| 09ccdc56 | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
| e176474e | kasimir |       |      | Project |   508 |   87 | 524.0 KiB |
3 snapshot(s)

total: 3 snapshot(s)

# Check:
$ rustic find --glob '**/secret.env' --filter-paths Project

searching in snapshots group (host [kasimir], label [], paths [Project])...
```

**Note:** The `rewrite` command modifies only the index trees. It does not deal
with the data packs. It has two consequences:

- You can rewrite snapshots configured to use cold storage without bringing
  anything into the hot storage.
- Even if the file was removed from the tree index, the data are still present
  in the rustic repository as pack files. For sensitive data, consider using the
  `rustic prune` command to remove the pack files. It will remove the pack
  files, but Rustic has no way to ensure that the data are wipped out from the
  repository's underlying disk/storage system.

## Rewriting tags

The `rustic rewrite` command also allows you to manipulate the snapshots' tags.
Note that there is the special `rustic tag` command which only allows to modify
tags, this is basically a shortcut for the respective options of
`rustic rewrite --force`.

You can use `rewrite --set-tags ...` to set a new tag list to a snapshot:

```console
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
1 snapshot(s)

total: 1 snapshot(s)

# Replace the tag list
$ rustic rewrite --set-tags origin,precious 7f3eef1f
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags     | Paths   | Files | Dirs |      Size |
|----------|---------|-------|----------|---------|-------|------|-----------|
| 0d675c5c | kasimir |       | origin   | Project |   507 |   87 | 524.0 KiB |
|          |         |       | precious |         |       |      |           |
|          |         |       | rewrite  |         |       |      |           |
| 7f3eef1f | kasimir |       |          | Project |   507 |   87 | 524.0 KiB |
2 snapshot(s)

total: 2 snapshot(s)
```

You can also add tags with `rewrite --add-tags ...` and remove them using
`rewrite --add-tags ...`:

```console
# Add more tags
$ rustic rewrite --forget --add-tags main 0d675c5c
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags     | Paths   | Files | Dirs |      Size |
|----------|---------|-------|----------|---------|-------|------|-----------|
| 285e7e62 | kasimir |       | main     | Project |   507 |   87 | 524.0 KiB |
|          |         |       | origin   |         |       |      |           |
|          |         |       | precious |         |       |      |           |
|          |         |       | rewrite  |         |       |      |           |
| 7f3eef1f | kasimir |       |          | Project |   507 |   87 | 524.0 KiB |
2 snapshot(s)

total: 2 snapshot(s)

# Remove some tags
$ rustic rewrite --forget --remove-tags rewrite,origin 285e7e62
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags     | Paths   | Files | Dirs |      Size |
|----------|---------|-------|----------|---------|-------|------|-----------|
| 3f1560c5 | kasimir |       | main     | Project |   507 |   87 | 524.0 KiB |
|          |         |       | precious |         |       |      |           |
| 7f3eef1f | kasimir |       |          | Project |   507 |   87 | 524.0 KiB |
2 snapshot(s)

total: 2 snapshot(s)
```

```console
# Replace the tag list
$ rustic rewrite --forget --set-tags origin 3f1560c5
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags   | Paths   | Files | Dirs |      Size |
|----------|---------|-------|--------|---------|-------|------|-----------|
| 7a690c53 | kasimir |       | origin | Project |   507 |   87 | 524.0 KiB |
| 7f3eef1f | kasimir |       |        | Project |   507 |   87 | 524.0 KiB |
2 snapshot(s)

total: 2 snapshot(s)
```

## Rewriting the hostname

You can change the hotname of an index tree. It can be useful if you have a
master snapshot you want to use for different hosts:

```console
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
1 snapshot(s)

total: 1 snapshot(s)

# Create a new identical snapshot with a different host name
# (--tags-rewritten '' prevent the addition of the `rewrite` tag)
$ rustic rewrite --set-hostname mikado --tags-rewritten '' 7f3eef1f
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------|-------|------|---------|-------|------|-----------|
| 7f3eef1f | kasimir |       |      | Project |   507 |   87 | 524.0 KiB |
1 snapshot(s)

snapshots for (host [mikado], label [], paths [Project])
| ID       | Host   | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|--------|-------|------|---------|-------|------|-----------|
| 5b28ba53 | mikado |       |      | Project |   507 |   87 | 524.0 KiB |
1 snapshot(s)
```

## More

If you want more, try `rustic rewrite --help` to see all the rewrite
capabilities.
