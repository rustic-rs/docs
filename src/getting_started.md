# Getting started

![rustic getting started](https://raw.githubusercontent.com/rustic-rs/assets/main/getting_started/gettingstarted.gif)

rustic is a commandline backup tool that provides encrypted, deduplicated
backups to various types of repositories.

Below is an example set of steps you can follow. They are similar but not
identical to the steps in the screencast above. By the end of these steps, you
should have some basic familiarity with rustic, how to set it up, backup, and
restore. You will be able to set up and use rustic in a configuration similar to
the one presented here.

By default, rustic expects to find a configuration file in the home directory of
the user who runs it. For Linux users, that file will almost always be
`~/.config/rustic/rustic.toml`.

```console
$ cat ~/.config/rustic/rustic.toml
[repository]
repository = "/tmp/repo"
password = "test"
```

First, we need to initialize the repository, as it doesn't exist yet:

```console
$ rustic init
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] key e65d9789 successfully added.
[INFO] repository 3f64ae74 successfully created.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601
$ ls /tmp/repo
total 68
drwxr-xr-x 1 jamesvasile jamesvasile    56 Mar 22 12:31 .
drwxrwxrwt 1 root        root  13618 Mar 22 12:31 ..
-rw-r--r-- 1 jamesvasile jamesvasile   155 Mar 22 12:31 config
drwxr-xr-x 1 jamesvasile jamesvasile  1024 Mar 22 12:31 data
drwxr-xr-x 1 jamesvasile jamesvasile     0 Mar 22 12:31 index
drwxr-xr-x 1 jamesvasile jamesvasile   128 Mar 22 12:31 keys
drwxr-xr-x 1 jamesvasile jamesvasile     0 Mar 22 12:31 snapshots
```

Note that `rustic init` created the directory for us. Let's make a test
directory to practice backing up and then we can start our first backup.

```console
$ mkdir src
$ echo "test me" > src/test.txt
$ rustic backup src
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601
[00:00:00] reading index...               ████████████████████████████████████████          0/0
[00:00:00] getting latest snapshot...     ████████████████████████████████████████          0/0
[INFO] using no parent
[INFO] starting to backup "src"...
[00:00:00] backing up...                  ████████████████████████████████████████        8 B/8 B   432 B/s  (ETA 0s)
Files:       1 new, 0 changed, 0 unchanged
Dirs:        2 new, 0 changed, 0 unchanged
Added to the repo: 476 B (raw: 529 B)
processed 1 files, 8 B
snapshot 8171d606 successfully saved.
[INFO] backup of "src" done.
```

Let's use the `rustic snapshots` command to see our snapshot:

```console
$ rustic snapshots
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601

snapshots for (host [francium], label [], paths [src])
| ID       | Time                | Host     | Label | Tags | Paths | Files | Dirs | Size |
|----------|---------------------|----------|-------|------|-------|-------|------|------|
| 8171d606 | 2024-03-22 12:39:04 | francium |       |      | src   |     1 |    2 |  8 B |
1 snapshot(s)

total: 1 snapshot(s)
```

Let's add a file and backup again.

```console
$ echo "Another test" > src/test2.txt
$ rustic backup src
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601
[00:00:00] reading index...               ████████████████████████████████████████          1/1
[00:00:00] getting latest snapshot...     ████████████████████████████████████████          1/1
[INFO] using parent 8171d606
[INFO] starting to backup "src"...
[00:00:00] backing up...                  ████████████████████████████████████████       21 B/21 B  1.09 KiB/s (ETA 0s)
Files:       1 new, 0 changed, 1 unchanged
Dirs:        0 new, 2 changed, 0 unchanged
Added to the repo: 556 B (raw: 902 B)
processed 2 files, 21 B
snapshot 27fda888 successfully saved.
[INFO] backup of "src" done.
```

This again was very fast as it only needed to process the added file. But still,
it generated a full snapshot:

```console
$ rustic snapshots
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601

snapshots for (host [francium], label [], paths [src])
| ID            | Time                | Host     | Label | Tags | Paths | Files | Dirs | Size |
|---------------|---------------------|----------|-------|------|-------|-------|------|------|
| 8171d606      | 2024-03-22 12:39:04 | francium |       |      | src   |     1 |    2 |  8 B |
| 27fda888      | 2024-03-22 12:55:23 | francium |       |      | src   |     2 |    2 | 21 B |
2 snapshot(s)

total: 2 snapshot(s)
```

What happens if we do a backup of a directory in which nothing changed?

```console
$ rustic backup src
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601
[00:00:00] reading index...               ████████████████████████████████████████          2/2
[00:00:00] getting latest snapshot...     ████████████████████████████████████████          2/2
[INFO] using parent 27fda888
[INFO] starting to backup "src"...
[00:00:00] backing up...                  ████████████████████████████████████████       21 B/21 B 2.69 KiB/s (ETA 0s)
Files:       0 new, 0 changed, 2 unchanged
Dirs:        0 new, 0 changed, 2 unchanged
Added to the repo: 0 B (raw: 0 B)
processed 2 files, 21 B
snapshot 2627ccdf successfully saved.
[INFO] backup of "src" done.
$ rustic snapshots
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601

snapshots for (host [francium], label [], paths [src])
| ID            | Time                | Host     | Label | Tags | Paths | Files | Dirs | Size |
|---------------|---------------------|----------|-------|------|-------|-------|------|------|
| 8171d606      | 2024-03-22 12:39:04 | francium |       |      | src   |     1 |    2 |  8 B |
| 27fda888 (+1) | 2024-03-22 12:55:23 | francium |       |      | src   |     2 |    2 | 21 B |
3 snapshot(s)

total: 3 snapshot(s)
```

On this latest backup run, rustic determined that there was no change and
created a snapshot with content identical to the previous one. This is indicated
by the `(+1)`, which shows how many identical backups exist. Note that the
timestamp will match the earliest identical snapshot, not the later ones.

Now we remove the added file:

```console
rm src/test2.txt
```

But it is still contained in the latest snapshot. Let's restore from that
snapshot.

```console
$ rustic restore latest:src/ src/
[INFO] using config /home/jamesvasile/.config/rustic/rustic.toml
[INFO] repository local:/tmp/repo: password is correct.
[INFO] using cache at /home/jamesvasile/.cache/rustic/3f64ae7425245b259ff98d1dcb6f49035af391f271da7b9ff643fd3f26408601
[00:00:00] reading index...               ████████████████████████████████████████          2/2
[00:00:00] getting latest snapshot...     ████████████████████████████████████████          3/3
[00:00:00] collecting file information...
Files:  1 to restore, 1 unchanged, 0 verified, 0 to modify, 0 additional
Dirs:   0 to restore, 0 to modify, 0 additional
[INFO] total restore size: 13 B
[INFO] using 8 B of existing file contents.
[00:00:00] restoring file contents...     ████████████████████████████████████████       13 B/13 B  6.12 KiB/s (ETA 0s)
[00:00:00] setting metadata...
restore done.
$ cat src/test2.txt
Another test
```

Note that rustic checks existing contents and only restores what's needed.

Those are the basics! From here you might want to:

- edit the configuration file to tune your backups and their location
- setup a remote backup repository
- script or schedule your rustic invocations for convenience and reliability
- test restoration from your snapshots
