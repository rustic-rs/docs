# Pack Format

All files in the repository except Key and Pack files just contain raw data,
stored as `IV || Ciphertext || MAC`. Pack files may contain one or more Blobs of
data.

A Pack's structure is as follows:

`EncryptedBlob1 || ... || EncryptedBlobN || EncryptedHeader || Header_Length`

At the end of the Pack file is a header, which describes the content. The header
is encrypted and authenticated. `Header_Length` is the length of the encrypted
header encoded as a four byte integer in little-endian encoding. Placing the
header at the end of a file allows writing the blobs in a continuous stream as
soon as they are read during the backup phase. This reduces code complexity and
avoids having to re-write a file once the pack is complete and the content and
length of the header is known.

All the blobs (`EncryptedBlob1`, `EncryptedBlobN` etc.) are authenticated and
encrypted independently. This enables repository reorganisation without having
to touch the encrypted Blobs. In addition it also allows efficient indexing, for
only the header needs to be read in order to find out which Blobs are contained
in the Pack. Since the header is authenticated, authenticity of the header can
be checked without having to read the complete Pack.

After decryption, a Pack's header consists of the following elements:

```text
Type_Blob1 || Data_Blob1 ||
[...]
Type_BlobN || Data_BlobN ||
```

The Blob type field is a single byte. What follows it depends on the type. The
following Blob types are defined:

| Type |       Meaning        |                                   Data                                   |
| :--: | :------------------: | :----------------------------------------------------------------------: |
| 0b00 |      data blob       |              Length(encrypted_blob) or Hash(plaintext_blob)              |
| 0b01 |      tree blob       |              Length(encrypted_blob) or Hash(plaintext_blob)              |
| 0b10 | compressed data blob | Length(encrypted_blob) or Length(plaintext_blob) or Hash(plaintext_blob) |
| 0b11 | compressed tree blob | Length(encrypted_blob) or Length(plaintext_blob) or Hash(plaintext_blob) |

This is enough to calculate the offsets for all the Blobs in the Pack. The
length fields are encoded as four byte integers in little-endian format. In the
Data column, `Length(plaintext_blob)` means the length of the decrypted and
uncompressed data a blob consists of.

All other types are invalid, more types may be added in the future. The
compressed types are only valid for repository format version 2. Data and tree
blobs may be compressed with the zstandard compression algorithm.

In repository format version 1, data and tree blobs should be stored in separate
pack files. In version 2, they must be stored in separate files. Compressed and
non-compress blobs of the same type may be mixed in a pack file.

For reconstructing the index or parsing a pack without an index, first the last
four bytes must be read in order to find the length of the header. Afterwards,
the header can be read and parsed, which yields all plaintext hashes, types,
offsets and lengths of all included blobs.
