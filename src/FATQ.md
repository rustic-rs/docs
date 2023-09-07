# Frequently asked technical questions

## Memory Requirements

> **Question**: *You mention a "huge decrease in memory requirement"; are you
> streaming blobs to the data store without keeping them in memory and just
> keeping the hashes in memory? How does the memory footprint look like?*

The largest memory consuming part is the index which is kept in-memory by restic
and rustic. For restic, however, the index took about 3-4 times the theoretical
size - maybe due to Golangs garbage collector additions or other Golang-specific
stuff.

In rustic, there are two basic improvements:

- the memory consumption is exactly as expected as Rust allows to directly
  define the in-memory structures

- For some operations, not all index information is needed. Rustic allows
  several kinds of index types to be loaded into memory (only trees, only ids,
  full index) and uses only the type needed for a given command. For instance
  with backup you need the list of all present blob IDs, but not the position
  within the repository - rustic thus only saves the IDs in memory for the
  backup command and therefore uses less memory.
