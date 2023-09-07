# Trees and Data

A snapshot references a tree by the SHA-256 hash of the JSON string
representation of its contents. Trees and data are saved in pack files in a
subdirectory of the directory `data`.

The command `restic cat blob` can be used to inspect the tree referenced above
(piping the output of the command to `jq .` so that the JSON is indented):

```console
$ restic -r /tmp/restic-repo cat blob 2da81727b6585232894cfbb8f8bdab8d1eccd3d8f7c92bc934d62e62e618ffdf | jq .
enter password for repository:
```

```json
{
    "nodes": [
        {
            "name": "testdata",
            "type": "dir",
            "mode": 493,
            "mtime": "2014-12-22T14:47:59.912418701+01:00",
            "atime": "2014-12-06T17:49:21.748468803+01:00",
            "ctime": "2014-12-22T14:47:59.912418701+01:00",
            "uid": 1000,
            "gid": 100,
            "user": "fd0",
            "inode": 409704562,
            "content": null,
            "subtree": "b26e315b0988ddcd1cee64c351d13a100fedbc9fdbb144a67d1b765ab280b4dc"
        }
    ]
}
```

A tree contains a list of entries (in the field `nodes`) which contain meta data
like a name and timestamps. When the entry references a directory, the field
`subtree` contains the plain text ID of another tree object.

When the command `restic cat blob` is used, the plaintext ID is needed to print
a tree. The tree referenced above can be dumped as follows:

```console
$ restic -r /tmp/restic-repo cat blob b26e315b0988ddcd1cee64c351d13a100fedbc9fdbb144a67d1b765ab280b4dc
enter password for repository:
```

```json
{
    "nodes": [
      {
        "name": "testfile",
        "type": "file",
        "mode": 420,
        "mtime": "2014-12-06T17:50:23.34513538+01:00",
        "atime": "2014-12-06T17:50:23.338468713+01:00",
        "ctime": "2014-12-06T17:50:23.34513538+01:00",
        "uid": 1000,
        "gid": 100,
        "user": "fd0",
        "inode": 416863351,
        "size": 1234,
        "links": 1,
        "content": [
          "50f77b3b4291e8411a027b9f9b9e64658181cc676ce6ba9958b95f268cb1109d"
        ]
      },
      [...]
    ]
}
```

This tree contains a file entry. This time, the `subtree` field is not present
and the `content` field contains a list with one plain text SHA-256 hash.

The command `restic cat blob` can also be used to extract and decrypt data given
a plaintext ID, e.g. for the data mentioned above:

```console
$ restic -r /tmp/restic-repo cat blob 50f77b3b4291e8411a027b9f9b9e64658181cc676ce6ba9958b95f268cb1109d | sha256sum
enter password for repository:
50f77b3b4291e8411a027b9f9b9e64658181cc676ce6ba9958b95f268cb1109d  -
```

As can be seen from the output of the program `sha256sum`, the hash matches the
plaintext hash from the map included in the tree above, so the correct data has
been returned.
