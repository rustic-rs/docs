# Upgrading the repository format version

Repositories created using earlier rustic (or restic) versions use an older
repository format version and have to be upgraded to allow using all new
features. Upgrading must be done explicitly as a newer repository version
increases the minimum rustic version required to access the repository. For
example the repository format version 2 is only readable using rustic 0.2.0 or
newer.

Upgrading to repo version 2 is a two step process: first run
`config --set-version 2` which will upgrade the repository version. After the
migration is complete, run `prune` to compress the repository metadata. To limit
the amount of data rewritten in at once, you can use the
`prune --max-repack-size size` parameter, see :ref:`customize-pruning` for more
details.

File contents stored in the repository will not be rewritten, data from new
backups will be compressed. Over time more and more of the repository will be
compressed. To speed up this process and compress all not yet compressed data,
you can run `prune --repack-uncompressed`.
