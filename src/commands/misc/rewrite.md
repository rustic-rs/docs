# Rewriting Snapshots

## Removing files

Sometimes you need to change an existing snapshot. Notably, if you discover it
includes unwanted files (e.g., a password in cleartext or large caches).

Instead of deleting the snapshot and then using the `backup` command again, you
may use the `rewrite` command to modify the existing snapshots.

### The long way

Let's imagine I have the following snapshots:

```console
$  rustic snapshots --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Time                | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------------------|---------|-------|------|---------|-------|------|-----------|
| 7cea1643 | 2026-01-23 10:18:52 | kasimir |       |      | Project |  1190 |   87 | 120.3 MiB |
| a1a54f27 | 2026-01-23 11:38:35 | kasimir |       |      | Project |  1191 |   88 | 120.3 MiB |
| d40f48a9 | 2026-01-23 11:52:11 | kasimir |       |      | Project |  1191 |   88 | 120.3 MiB |
3 snapshot(s)

total: 3 snapshot(s)
```

Unfortunately, the *Project/config/secret.env* file has sneaked in. First thing,
identify the snapshots containing that file:

```console
$  rustic find --glob '**/secret.env' --filter-paths Project

searching in snapshots group (host [kasimir], label [], paths [Project])...
found in a1a54f27 from 2026-01-23 11:38:35
-rw-r--r--  sylvain  sylvain        13 23 Jan 2026 11:38 "Project/config/secret.env" 
found in d40f48a9 from 2026-01-23 11:52:11
-rw-------  sylvain  sylvain        13 23 Jan 2026 11:38 "Project/config/secret.env"
```

Apparently, two snapshots are concerned. Let's rewrite them to *exclude* that
file:

```console
$ rustic -n rewrite --glob '!**/secret.env' a1a54f27 d40f48a9
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Time                | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------------------|---------|-------|------|---------|-------|------|-----------|
| 7cea1643 | 2026-01-23 10:18:52 | kasimir |       |      | Project |  1190 |   87 | 120.3 MiB |
| a1a54f27 | 2026-01-23 11:38:35 | kasimir |       |      | Project |  1191 |   88 | 120.3 MiB |
| 1cb5ee37 | 2026-01-23 11:38:35 | kasimir |       |      | Project |   653 |   88 | 120.3 MiB |
| d40f48a9 | 2026-01-23 11:52:11 | kasimir |       |      | Project |  1191 |   88 | 120.3 MiB |
| fa65bc8d | 2026-01-23 11:52:11 | kasimir |       |      | Project |   653 |   88 | 120.3 MiB |
5 snapshot(s)

total: 5 snapshot(s)
```

**Note:** In the `rewrite` command, you must use a negative glob (starting with
an exclamation mark) to specify the files you want to remove.

**XXX**: Why do the rewritten snapshots display such a different number of files
compared to the original?

As shown above, the snapshots `a1a54f27` and `d40f48a9` are still present. By
default, the `rewrite` command does *not* remove the source snapshots. So in my
case, I will have to remove them manually:

```console
$ rustic forget a1a54f27 d40f48a9
$ rustic snapshots --all --filter-paths Project

snapshots for (host [kasimir], label [], paths [Project])
| ID       | Time                | Host    | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------------------|---------|-------|------|---------|-------|------|-----------|
| 7cea1643 | 2026-01-23 10:18:52 | kasimir |       |      | Project |  1190 |   87 | 120.3 MiB |
| 1cb5ee37 | 2026-01-23 11:38:35 | kasimir |       |      | Project |   653 |   88 | 120.3 MiB |
| fa65bc8d | 2026-01-23 11:52:11 | kasimir |       |      | Project |   653 |   88 | 120.3 MiB |
3 snapshot(s)

total: 3 snapshot(s)
```

You can now check: no remaining snapshots contain the file we removed:

```console
$ rustic find --glob '**/secret.env' --filter-paths Project

searching in snapshots group (host [kasimir], label [], paths [Project])...
```

### The short way

In the above section, we've seen all the involved steps with great detail. We
especially had to manually remove the unwanted snapshots. But to do it for you,
the `rewrite` command has the `--forget` option.

Let's restart in a clean state:

```console
$ rustic backup Project 
$ rustic snapshots --all

snapshots for (host [109a0e940612], label [], paths [Project])
| ID       | Time                | Host         | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------------------|--------------|-------|------|---------|-------|------|-----------|
| 22b7d2a4 | 2026-01-23 14:10:29 | kasimir      |       |      | Project |  1191 |   88 | 120.3 MiB |
1 snapshot(s)

total: 1 snapshot(s)
$ rustic find --glob 'Project/config/secret.env' --filter-paths Project

searching in snapshots group (host [109a0e940612], label [], paths [Project])...
found in 22b7d2a4 from 2026-01-23 14:10:29
-rw-------  sylvain  sylvain        13 23 Jan 2026 11:38 "Project/config/secret.env"
```

As you noticed, the `secret.env` file is back in this example snapshot. We will
remove it again, but this time using the `rewrite --forget` command:

```console
$ rustic rewrite --forget --glob '!Project/config/secret.env' --filter-paths Project

$ rustic snapshots --all

snapshots for (host [109a0e940612], label [], paths [Project])
| ID       | Time                | Host         | Label | Tags | Paths   | Files | Dirs |      Size |
|----------|---------------------|--------------|-------|------|---------|-------|------|-----------|
| b7826f11 | 2026-01-23 14:10:29 | kasimir      |       |      | Project |   653 |   88 | 120.3 MiB |
1 snapshot(s)

total: 1 snapshot(s)
```

The original snapshot was removed, and the rewritten one no longer lists the
`secret.env` file.

**Note:** The `rewrite` command modifies only the index trees. It does not deal
with the data packs. It has two consequences:

- You can rewrite snapshots configured to use cold storage without bringing
  anything into the hot storage.
- Even if the file was removed from the tree index, the data are still present
  in the rustic repository as pack files. For sensitive data, consider using the
  `rustic prune` command to remove the pack files. It will remove the pack
  files, but Rustic has no way to ensure that the data are wipped out from the
  repository's underlying disk/storage system.
