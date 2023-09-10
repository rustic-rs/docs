# Keys, Encryption and MAC

All data stored by restic in the repository is encrypted with AES-256 in counter
mode and authenticated using Poly1305-AES. For encrypting new data first 16
bytes are read from a cryptographically secure pseudo-random number generator as
a random nonce. This is used both as the IV for counter mode and the nonce for
Poly1305. This operation needs three keys: A 32 byte for AES-256 for encryption,
a 16 byte AES key and a 16 byte key for Poly1305. For details see the original
paper
[The Poly1305-AES message-authentication code](https://cr.yp.to/mac/poly1305-20050329.pdf)
by Dan Bernstein. The data is then encrypted with AES-256 and afterwards a
message authentication code (MAC) is computed over the ciphertext, everything is
then stored as IV \|\| CIPHERTEXT \|\| MAC.

The directory `keys` contains key files. These are simple JSON documents which
contain all data that is needed to derive the repository's master encryption and
message authentication keys from a user's password. The JSON document from the
repository can be pretty-printed for example by using the Python module `json`
(shortened to increase readability):

```console
python -mjson.tool /tmp/restic-repo/keys/b02de82*
```

```json
{
    "hostname": "kasimir",
    "username": "fd0",
    "kdf": "scrypt",
    "N": 65536,
    "r": 8,
    "p": 1,
    "created": "2015-01-02T18:10:13.48307196+01:00",
    "data": "tGwYeKoM0C4j4/9DFrVEmMGAldvEn/+iKC3te/QE/6ox/V4qz58FUOgMa0Bb1cIJ6asrypCx/Ti/pRXCPHLDkIJbNYd2ybC+fLhFIJVLCvkMS+trdywsUkglUbTbi+7+Ldsul5jpAj9vTZ25ajDc+4FKtWEcCWL5ICAOoTAxnPgT+Lh8ByGQBH6KbdWabqamLzTRWxePFoYuxa7yXgmj9A==",
    "salt": "uW4fEI1+IOzj7ED9mVor+yTSJFd68DGlGOeLgJELYsTU5ikhG/83/+jGd4KKAaQdSrsfzrdOhAMftTSih5Ux6w=="
}
```

When the repository is opened by restic, the user is prompted for the repository
password. This is then used with `scrypt`, a key derivation function (KDF), and
the supplied parameters (`N`, `r`, `p` and `salt`) to derive 64 key bytes. The
first 32 bytes are used as the encryption key (for AES-256) and the last 32
bytes are used as the message authentication key (for Poly1305-AES). These last
32 bytes are divided into a 16 byte AES key `k` followed by 16 bytes of secret
key `r`. The key `r` is then masked for use with Poly1305 (see the paper for
details).

Those keys are used to authenticate and decrypt the bytes contained in the JSON
field `data` with AES-256 and Poly1305-AES as if they were any other blob (after
removing the Base64 encoding). If the password is incorrect or the key file has
been tampered with, the computed MAC will not match the last 16 bytes of the
data, and restic exits with an error. Otherwise, the data yields a JSON document
which contains the master encryption and message authentication keys for this
repository (encoded in Base64). The command `restic cat masterkey` can be used
as follows to decrypt and pretty-print the master key:

```console
restic -r /tmp/restic-repo cat masterkey
```

```json
{
    "mac": {
        "k": "evFWd9wWlndL9jc501268g==",
        "r": "E9eEDnSJZgqwTOkDtOp+Dw=="
    },
    "encrypt": "UQCqa0lKZ94PygPxMRqkePTZnHRYh1k1pX2k2lM2v3Q="
}
```

All data in the repository is encrypted and authenticated with these master
keys. For encryption, the AES-256 algorithm in Counter mode is used. For message
authentication, Poly1305-AES is used as described above.

A repository can have several different passwords, with a key file for each.
This way, the password can be changed without having to re-encrypt all data.
