# Frequently asked questions

<!-- TOC -->

- [Can I use rustic with my existing restic repositories?](#can-i-use-rustic-with-my-existing-restic-repositories)
- [What are the differences between rustic and restic?](#what-are-the-differences-between-rustic-and-restic)
- [Why is rustic written in Rust](#why-is-rustic-written-in-rust)
- [How does rustic work with cold storages like AWS Glacier?](#how-does-rustic-work-with-cold-storages-like-aws-glacier)
- [How does the lock-free prune work?](#how-does-the-lock-free-prune-work)
- [You said "rustic uses less resources than restic" but I'm observing the opposite](#you-said-rustic-uses-less-resources-than-restic-but-im-observing-the-opposite)

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
