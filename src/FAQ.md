# Frequently asked questions

<!-- TOC -->

- [Can I use rustic with my existing restic repositories?](#can-i-use-rustic-with-my-existing-restic-repositories)
- [What are the differences between rustic and restic?](#what-are-the-differences-between-rustic-and-restic)
- [Why is rustic written in Rust](#why-is-rustic-written-in-rust)
- [How does rustic work with cold storages like AWS Glacier?](#how-does-rustic-work-with-cold-storages-like-aws-glacier)
- [Are all operations lock free?](#are-all-operations-lock-free)
- [How does the lock-free prune work?](#how-does-the-lock-free-prune-work)
- [You said "rustic uses less resources than restic" but I'm observing the opposite](#you-said-rustic-uses-less-resources-than-restic-but-im-observing-the-opposite)
- [File exclusion for Windows Defender](#file-exclusion-for-windows-defender)
- [Pass environment variables to RCLONE](#pass-environment-variables-to-rclone)

<!-- /TOC -->

## Can I use rustic with my existing restic repositories?

Yes, you can. rustic uses the same repository format as restic, so you can use
rustic and restic on the same repository. The only thing you have to take care
of is that you don't run prune with restic and rustic at the same time.

## What are the differences between rustic and restic?

- Written in Rust instead of golang
- Optimized for small resource usage (in particular memory usage, but also
  overall CPU usage)
- Philosophy of development (release new features early)
- New features (e.g. hot/cold repositories, lock-free pruning)
- Some commands or options act a bit different or have slightly different syntax

A more detailed comparison can be found in the
[comparison table](./comparison-restic.md).

## Why is rustic written in Rust

Rust is a powerful language designed to build reliable and efficient software.
This is a very good fit for a backup tool.

## How does rustic work with cold storages like AWS Glacier?

If you want to use cold storage, make sure you always specify an extra
repository `--repo-hot` which contains the hot data. This repository acts like a
cache for all metadata, i.e. config/key/snapshot/index files and tree packs. As
all commands except `restore` only need to access the metadata, they are fully
functional but only need the cold storage to list files while everything else is
read from the "hot repo". Note that the "hot repo" on its own is not a valid
rustic repository. The "cold repo", however, contains all files and is nothing
but a standard rustic repository.

If you additionally use a cache, you effectively have a first level cache on
your local disc and a second level cache with the "hot repo". Note that the "hot
repo" can be also a remote repo, so hot/cold repositories also work for multiple
rustic clients backing up to the same repository.

### More details

rustic doesn't support single repositories/buckets which have objects that can
be in different "states" (something like this is not contained in the restic
REST protocol; rclone therefore isn't able to provide this kind of information).

So, to use S3 glacier, you should use 2 buckets: A hot one which is always
accessible and a cold one where all object should be transitioned/transferred
directly to Glacier when they are stored. The "hot" objects are then in fact
stored in both buckets, but this shouldn't be much overhead as it is only
metadata which usually is less than 1% of the repo size. (The side effect is
that the cold repo contains a full restic-compatible repository; if you warm up
it completely, you can use it a standard repository within rustic and even
restic).

TL;DR: I think you have to do the following:

1. To store data (e.g. `init` or `backup` command):

- Create a regular bucket and use it as `repo-hot`
- Create a bucket for `cold` and configure it to store data directly in Glacier
  (don't know if you can configure this in `rclone` or in AWS)

2. To retrieve data (e.g. `restore`):

- configure `warm-up-command` such that objects from the cold buckets are
  restored, see e.g
  <https://docs.aws.amazon.com/AmazonS3/latest/userguide/restoring-objects.html>.
  That is, the config file should look have a line like
  `warm-up-command  = 'aws s3api restore-object --bucket BUCKET --key path/data/%id --restore-request '{"Days":25,"GlacierJobParameters":{"Tier":"Standard"}}'  '`
  (may need some adaption)
- configure a suitable `warm-up-wait` to ensure that the data is warmed up and
  can be accessed after that
- Note: All commands which need to read cold data now will warm this data up and
  wait for the given time before processing the data

3. removing cold data (`prune`) This actually should work out of the box, but
   note that by default `prune` doesn't repack any cold pack file; instead it
   keeps them until the last blob within is no longer used.

- to do more repacking, have a look at the `repack-cacheable` option which does
  allow to repack cold pack files.
- Use the `keep-pack` option if you have some minimum holding duration, i.e. if
  you would get charged if you remove objects too early. This option ensures
  that only pack files older than the given time will be removed.

Once you get a working configuration, please share it
[in the main repository](https://github.com/rustic-rs/rustic/tree/main/config/services)
so other users can use it as well!

## Are all operations lock free?

Yes, all operations are designed lock-free. This means all commands can run
parallel. This is especially true for multiple backup runs and backup runs
parallel to `prune`. However, make sure that each individual `backup` run won't
take longer as the `--keep-delete` option (default: 23h) if you run `prune`
parallel to `backup` - and that you don't use `--instant-delete`.

Of course, any read-only command also safely runs parallel to any other command.

Multiple parallel `forget` or `prune` runs are designed to work, too. But I have
to admit that we have not tested this in detail so far. Because of this and
other reasons (like ACLs to set up and scheduling) I would recommend to schedule
`forget` and `prune` for the whole repository using a single host to do so.
However feedback on this with multiple parallel runs is highly appreciated.

There is one general caveat: Error handling is not perfect yet in rustic and we
may run into cases where parallel runs may throw errors. E.g. the combination
`forget`/`check` may lead to `check` trying to load just-removed snapshots. Or
`forget`/`forget` may want to remove a just-removed snapshots which then fails.
These are however all cases which should not affect the general repository
consistency. If you encounter such an error handling problem, please report it
in our [issue tracker](https://github.com/rustic-rs/rustic/issues/new)!

## How does the lock-free prune work?

Like the prune within restic, rustic decides for each pack whether to keep it,
remove it or repack it. Instead of removing packs, it however only marks the
packs to remove in a separate index structure. Packs which are marked for
removal are checked if they are really not needed and have been marked long
enough ago. Depending on these checks they are either finally removed, recovered
or kept in the state of being marked for removal.

This two-phase deletion is needed for rustic to work lock-free: If a `backup`
runs parallel to a `prune` run (or `forget --prune`), it could be that prune
decides that some blobs can be removed, but the parallel backup uses these blobs
for the newly generated snapshot.

The time to hold marked packs should be long enough to guarantee that a possibly
parallel backup run has finished in between. It can be set by the
`--keep-delete` option and defaults to 23 hours. In any case, packs will be kept
marked and only deleted by the next prune run.

Note that there is the option `--instant-delete` which circumvents this
two-phase deletion. Only use this option, if you **REALLY KNOW** that there is
no parallel access to your repo, else you risk losing data!

## You said "rustic uses less resources than restic" but I'm observing the opposite

In general rustic uses less resources, but there may be some exceptions. For
instance the crypto libraries of Rust and golang both have optimizations for
some CPUs. But it might be that your CPU benefits from a golang optimization
which is not present in the Rust implementation. If you observe some unexpected
resource usage, please don't hesitate to submit an issue.

## File exclusion for Windows Defender

If you are on Windows and you are using Windows Defender, you might want to
exclude the `rustic.exe` binary from being scanned to speed up all operations.
This can be done by adding an exclusion in the Windows Security settings. Here
is how you can do it on the command line (elevated):

```powershell
Add-MpPreference -ExclusionProcess "rustic.exe"
```

## Pass environment variables to RCLONE

Environment variables are set by `rustic` before calling a subcommand, e.g.
`rclone` or commands defined in the repository options. For example, to set the
`RCLONE_FAST_LIST` environment variable you add the following to your
`<profile>.toml`:

```toml
[global.env]
RCLONE_FAST_LIST = "true"
```

Make sure to check the
[latest config documentation](https://github.com/rustic-rs/rustic/blob/main/config/full.toml#L29)
for the correct syntax.
