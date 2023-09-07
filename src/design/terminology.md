# Terminology

This section introduces terminology used in this document.

*Repository*: All data produced during a backup is sent to and stored in a
repository in a structured form, for example in a file system hierarchy with
several subdirectories. A repository implementation must be able to fulfill a
number of operations, e.g. list the contents.

*Blob*: A Blob combines a number of data bytes with identifying information like
the SHA-256 hash of the data and its length.

*Pack*: A Pack combines one or more Blobs, e.g. in a single file.

*Snapshot*: A Snapshot stands for the state of a file or directory that has been
backed up at some point in time. The state here means the content and metadata
like the name and modification time for the file or the directory and its
contents.

*Storage ID*: A storage ID is the SHA-256 hash of the content stored in the
repository. This ID is required in order to load the file from the repository.
