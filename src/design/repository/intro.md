# Repository Format

All data is stored in a restic repository. A repository is able to store data of
several different types, which can later be requested based on an ID. This
so-called "storage ID" is the SHA-256 hash of the content of a file. All files
in a repository are only written once and never modified afterwards. Writing
should occur atomically to prevent concurrent operations from reading incomplete
files. This allows accessing and even writing to the repository with multiple
clients in parallel. Only the `prune` operation removes data from the
repository.

Repositories consist of several directories and a top-level file called
`config`. For all other files stored in the repository, the name for the file is
the lower case hexadecimal representation of the storage ID, which is the
SHA-256 hash of the file's contents. This allows for easy verification of files
for accidental modifications, like disk read errors, by simply running the
program `sha256sum` on the file and comparing its output to the file name. If
the prefix of a filename is unique amongst all the other files in the same
directory, the prefix may be used instead of the complete filename.

Apart from the files stored within the `keys` directory, all files are encrypted
with AES-256 in counter mode (CTR). The integrity of the encrypted data is
secured by a Poly1305-AES message authentication code (sometimes also referred
to as a "signature").

In the first 16 bytes of each encrypted file the initialisation vector (IV) is
stored. It is followed by the encrypted data and completed by the 16 byte MAC.
The format is: `IV || CIPHERTEXT || MAC`. The complete encryption overhead is 32
bytes. For each file, a new random IV is selected.

The file `config` is encrypted this way and contains a JSON document like the
following:

```json
{
    "version": 2,
    "id": "5956a3f67a6230d4a92cefb29529f10196c7d92582ec305fd71ff6d331d6271b",
    "chunker_polynomial": "25b468838dcb75"
}
```

After decryption, restic first checks that the version field contains a version
number that it understands, otherwise it aborts. At the moment, the version is
expected to be 1 or 2. The list of changes in the repository format is contained
in the section "Changes" below.

The field `id` holds a unique ID which consists of 32 random bytes, encoded in
hexadecimal. This uniquely identifies the repository, regardless if it is
accessed via a remote storage backend or locally. The field `chunker_polynomial`
contains a parameter that is used for splitting large files into smaller chunks
(see below).
