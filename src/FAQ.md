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
