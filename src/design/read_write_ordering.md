# Read and Write Ordering

The repository format allows writing (e.g. backup) and reading (e.g. restore) to
happen concurrently. As the data for each snapshot in a repository spans
multiple files (snapshot, index and packs), it is necessary to follow certain
rules regarding the order in which files are read and written. These ordering
rules also guarantee that repository modifications always maintain a correct
repository even if the client or the storage backend crashes for example due to
a power cut or the (network) connection between both is interrupted.

The correct order to access data in a repository is derived from the following
set of invariants that must be maintained at **any time** in a correct
repository. *Must* in the following is a strict requirement and will lead to
data loss if not followed. *Should* will require steps to fix a repository (e.g.
rebuilding the index) if not followed, but should not cause data loss.
*existing* means that the referenced data is **durably** stored in the
repository.

- A snapshot *must* only reference an existing tree blob.
- A reachable tree blob *must* only reference tree and data blobs that exist
  (recursively). *Reachable* means that the tree blob is reachable starting from
  a snapshot.
- An index *must* only reference valid blobs in existing packs.
- All blobs referenced by a snapshot *should* be listed in an index.

This leads to the following recommended order to store data in a repository.
First, pack files, which contain data and tree blobs, must be written. Then the
indexes which reference blobs in these already written pack files. And finally
the corresponding snapshots.

Note that there is no need for a specific write order of data and tree blobs
during a backup as the blobs only become referenced once the corresponding
snapshot is uploaded.

Reading data should follow the opposite order compared to writing. Only once a
snapshot was written, it is guaranteed that all required data exists in the
repository. This especially means that the list of snapshots to read should be
collected before loading the repository index. The other way round can lead to a
race condition where a recently written snapshot is loaded but not its
accompanying index, which results in a failure to access the snapshot's tree
blob.

For removing or rewriting data from a repository the following rules must be
followed, which are derived from the above invariants.

- A client removing data *must* acquire an exclusive lock first to prevent
  conflicts with other clients.
- A pack *must* be removed from the referencing index before it is deleted.
- Rewriting a pack *must* write the new pack, update the index (add an updated
  index and delete the old one) and only then delete the old pack.
