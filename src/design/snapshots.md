# Snapshots

A snapshot represents a directory with all files and sub-directories at a given
point in time. For each backup that is made, a new snapshot is created. A
snapshot is a JSON document that is stored in a file below the directory
`snapshots` in the repository. It uses the file encoding described in the
"Unpacked Data Format" section. The filename is the storage ID. This string is
unique and used within restic to uniquely identify a snapshot.

The command `restic cat snapshot` can be used as follows to decrypt and
pretty-print the contents of a snapshot file:

```console
restic -r /tmp/restic-repo cat snapshot 251c2e58
enter password for repository:
```

```json
{
    "time": "2015-01-02T18:10:50.895208559+01:00",
    "tree": "2da81727b6585232894cfbb8f8bdab8d1eccd3d8f7c92bc934d62e62e618ffdf",
    "dir": "/tmp/testdata",
    "hostname": "kasimir",
    "username": "fd0",
    "uid": 1000,
    "gid": 100,
    "tags": [
        "NL"
    ]
}
```

Here it can be seen that this snapshot represents the contents of the directory
`/tmp/testdata`. The most important field is `tree`. When the meta data (e.g.
the tags) of a snapshot change, the snapshot needs to be re-encrypted and saved.
This will change the storage ID, so in order to relate these seemingly different
snapshots, a field `original` is introduced which contains the ID of the
original snapshot, e.g. after adding the tag `DE` to the snapshot above it
becomes:

```console
$ restic -r /tmp/restic-repo cat snapshot 22a5af1b
enter password for repository:
```

```json
{
    "time": "2015-01-02T18:10:50.895208559+01:00",
    "tree": "2da81727b6585232894cfbb8f8bdab8d1eccd3d8f7c92bc934d62e62e618ffdf",
    "dir": "/tmp/testdata",
    "hostname": "kasimir",
    "username": "fd0",
    "uid": 1000,
    "gid": 100,
    "tags": [
        "NL",
        "DE"
    ],
    "original": "251c2e5841355f743f9d4ffd3260bee765acee40a6229857e32b60446991b837"
}
```

Once introduced, the `original` field is not modified when the snapshot's meta
data is changed again.

All content within a restic repository is referenced according to its SHA-256
hash. Before saving, each file is split into variable sized Blobs of data. The
SHA-256 hashes of all Blobs are saved in an ordered list which then represents
the content of the file.

In order to relate these plaintext hashes to the actual location within a Pack
file, an index is used. If the index is not available, the header of all data
Blobs can be read.
