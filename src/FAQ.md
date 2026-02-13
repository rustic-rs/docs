# Frequently asked questions

<!-- TOC -->

- [Can I use rustic with my existing restic repositories?](#can-i-use-rustic-with-my-existing-restic-repositories)
- [What are the differences between rustic and restic?](#what-are-the-differences-between-rustic-and-restic)
- [Why is rustic written in Rust](#why-is-rustic-written-in-rust)
- [Can/should I use cold storage?](#canshould-i-use-cold-storage)
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

## Can/should I use cold storage?

rustic is unique in backup tools in having direct support for cold storage like
AWS S3's Glacier storage classes and OVH Cloud Cold Object Storage's Cold
Archive class.

Cold storage is a storage tier that offers a lower cost of storing your data by
increasing the cost or time to upload and/or retrieve your data. Consider the
full lifecycle of your data when comparing costs. You'll most benefit from this
tradeoff if your dataset is large and you never need to retrieve it. (In other
words, if it is your secondary or tertiary storage, serving only as an insurance
policy against total data loss.)

- [Economic side of hosting rustic repo in AWS Glacier](https://kmh.prasil.info/posts/rustic-cold-storage-glacier-economics/)
  [(archive)](https://archive.ph/jGA4r)
- [drinkcat's *AWS S3 Glacier Deep Archive Cost computation*](https://drinkc.at/blog/2024/07/09/backup-to-glacier-calc/)
  [(archive)](https://archive.ph/wip/qUTyY)

If cold storage sounds beneficial for you, then refer to
[*Preparing a new repository* -> *Cold Storage*](./commands/init/cold_storage.md)
for how to set it up.

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
