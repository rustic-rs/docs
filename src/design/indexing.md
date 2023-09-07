# Indexing

Index files contain information about Data and Tree Blobs and the Packs they are
contained in and store this information in the repository. When the local cached
index is not accessible any more, the index files can be downloaded and used to
reconstruct the index. The file encoding is described in the "Unpacked Data
Format" section. The plaintext consists of a JSON document like the following:

```json
{
  "supersedes": [
    "ed54ae36197f4745ebc4b54d10e0f623eaaaedd03013eb7ae90df881b7781452"
  ],
  "packs": [
    {
      "id": "73d04e6125cf3c28a299cc2f3cca3b78ceac396e4fcf9575e34536b26782413c",
      "blobs": [
        {
          "id": "3ec79977ef0cf5de7b08cd12b874cd0f62bbaf7f07f3497a5b1bbcc8cb39b1ce",
          "type": "data",
          "offset": 0,
          "length": 38,
          // no 'uncompressed_length' as blob is not compressed
        },
        {
          "id": "9ccb846e60d90d4eb915848add7aa7ea1e4bbabfc60e573db9f7bfb2789afbae",
          "type": "tree",
          "offset": 38,
          "length": 112,
          "uncompressed_length": 511,
        },
        {
          "id": "d3dc577b4ffd38cc4b32122cabf8655a0223ed22edfd93b353dc0c3f2b0fdf66",
          "type": "data",
          "offset": 150,
          "length": 123,
          "uncompressed_length": 234,
        }
      ]
    }, [...]
  ]
}
```

This JSON document lists Packs and the blobs contained therein. In this example,
the Pack `73d04e61` contains two data Blobs and one Tree blob, the plaintext
hashes are listed afterwards. The `length` field corresponds to
`Length(encrypted_blob)` in the pack file header. Field `uncompressed_length` is
only present for compressed blobs and therefore is never present in version 1.
It is set to the value of `Length(blob)`.

The field `supersedes` lists the storage IDs of index files that have been
replaced with the current index file. This happens when index files are
repacked, for example when old snapshots are removed and Packs are recombined.

There may be an arbitrary number of index files, containing information on
non-disjoint sets of Packs. The number of packs described in a single file is
chosen so that the file size is kept below 8 MiB.
