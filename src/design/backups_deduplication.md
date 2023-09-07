# Backups and Deduplication

For creating a backup, restic scans the source directory for all files,
sub-directories and other entries. The data from each file is split into
variable length Blobs cut at offsets defined by a sliding window of 64 bytes.
The implementation uses Rabin Fingerprints for implementing this Content Defined
Chunking (CDC). An irreducible polynomial is selected at random and saved in the
file `config` when a repository is initialized, so that watermark attacks are
much harder.

Files smaller than 512 KiB are not split, Blobs are of 512 KiB to 8 MiB in size.
The implementation aims for 1 MiB Blob size on average.

For modified files, only modified Blobs have to be saved in a subsequent backup.
This even works if bytes are inserted or removed at arbitrary positions within
the file.
