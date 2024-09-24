# Checking integrity and consistency

Imagine your repository is saved on a server that has a faulty hard drive, or
even worse, attackers get privileged access and modify the files in your
repository with the intention to make you restore malicious data:

```console
echo "boom" > /srv/rustic-repo/index/de30f3231ca2e6a59af4aa84216dfe2ef7339c549dc11b09b84000997b139628
```

Trying to restore a snapshot which has been modified as shown above will yield
an error:

```console
$ rustic restore c23e491f /tmp/restore-work
...
Fatal: unable to load index de30f323: load <index/de30f3231c>: invalid data returned
```

In order to detect these things before they become a problem, it's a good idea
to regularly use the `check` command to test whether your repository is healthy
and consistent, and that your precious backup data is unharmed. There are two
types of checks that can be performed:

- Structural consistency and integrity, e.g. snapshots, trees and pack files
  (default)
- Integrity of the actual data that you backed up (enabled with flags, see
  below)

To verify the structure of the repository, issue the `check` command. If the
repository is damaged like in the example above, `check` will detect this and
yield the same error as when you tried to restore:

```console
$ rustic check
...
load indexes
error: error loading index de30f323: load <index/de30f3231c>: invalid data returned
Fatal: LoadIndex returned errors
```

If the repository structure is intact, rustic will show that no errors were
found:

```console
$ rustic check
...
load indexes
check all packs
check snapshots, trees and blobs
no errors were found
```

By default, the `check` command does not verify that the actual pack files on
disk in the repository are unmodified, because doing so requires reading a copy
of every pack file in the repository. To tell rustic to also verify the
integrity of the pack files in the repository, use the `--read-data` flag:

```console
$ rustic check --read-data
...
load indexes
check all packs
check snapshots, trees and blobs
read all data
[0:00] 100.00%  3 / 3 items
duration: 0:00
no errors were found
```

**Note**: Since `--read-data` has to download all pack files in the repository,
beware that it might incur higher bandwidth costs than usual and also that it
takes more time than the default `check`.

Alternatively, use the `--read-data-subset` parameter to check only a subset of
the repository pack files at a time. It supports three ways to select a subset.
One selects a specific part of pack files, the second and third selects a random
subset of the pack files by the given percentage or size.

Use `--read-data-subset=n/t` to check a specific part of the repository pack
files at a time. The parameter takes two values, `n` and `t`. When the check
command runs, all pack files in the repository are logically divided in `t`
(roughly equal) groups, and only files that belong to group number `n` are
checked. For example, the following commands check all repository pack files
over 5 separate invocations:

```console
rustic check --read-data-subset=1/5
rustic check --read-data-subset=2/5
rustic check --read-data-subset=3/5
rustic check --read-data-subset=4/5
rustic check --read-data-subset=5/5
```

Use `--read-data-subset=x%` to check a randomly choosen subset of the repository
pack files. It takes one parameter, `x`, the percentage of pack files to check
as an integer or floating point number. This will not guarantee to cover all
available pack files after sufficient runs, but it is easy to automate checking
a small subset of data after each backup. For a floating point value the
following command may be used:

```console
rustic check --read-data-subset=2.5%
```

When checking bigger subsets you most likely want to specify the percentage as
an integer:

```console
rustic check --read-data-subset=10%
```

Use `--read-data-subset=nS` to check a randomly chosen subset of the repository
pack files. It takes one parameter, `nS`, where 'n' is a whole number
representing file size and 'S' is the unit of file size (K/M/G/T) of pack files
to check. Behind the scenes, the specified size will be converted to percentage
of the total repository size. The behaviour of the check command following this
conversion will be the same as the percentage option above. For a file size
value the following command may be used:

```console
rustic check --read-data-subset=50M
rustic check --read-data-subset=10G
```
