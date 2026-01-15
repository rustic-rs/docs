# Key - Manage repository passwords/keys

The `key` command allows you to set multiple passwords/access keys for a
repository. Note that all keys share the same masterkey which is actually used
for the encryption. Every password/key can be used to derive the masterkey and
has thus full control over the repository.

You can use the `list`, `add`, `remove`, and `passwd` (changes a password)
sub-commands to manage these passwords/keys very precisely:

```console
    $ rustic key list
    enter password for repository:
    | ID        | User     | Host    | Created                              |
    |-----------|----------|---------|--------------------------------------|
    | *8a9d4286 | username | kasimir | 2025-08-12 13:29:57.759858440 +02:00 |

    $ rustic key add --with-created --username alex --hostname batman
    enter password for new key: [hidden]
    [INFO] key 70970299 successfully added.

    $ rustic key list
    enter password for repository:
    | ID        | User     | Host    | Created                              |
    |-----------|----------|---------|--------------------------------------|
    | *8a9d4286 | username | kasimir | 2025-08-12 13:29:57.759858440 +02:00 |
    | 70970299  | alex     | batman  | 2025-10-11 22:37:29.759858440 +02:00 |

    $ rustic key remove 7
    [INFO] key 70970299 successfully removed.
```

**Note**: If the repository is opend by a password, the corresponding used key
is indicated by an asterisk (`*`) and cannot be removed for safety reasons.

## Changing credentials from password/key-based to masterkey-based

1. Export the masterkey using `rustic key export`, e.g.

```console
$ rustic key export /path/to/key-file
```

2. Set this key as credentials to use, e.g. in the profile

```toml
[repository]
key-file = "/path/to/key-file"
```

3. (optionally) list and remove all passwords/keys

```console
$ rustic key list
$ rustic key remove <FIRST ENTRY>
$ rustic key remove <SECOND ENTRY>
...
```

## Changing credentials from masterkey-based to password/key-based

1. Create a key from a password (if not already existing):

```console
$ rustic key add
enter password for new key: [hidden]
[INFO] key 70970299 successfully added.
```

2. Set this password as credential, e.g.

```toml
[repository]
password = "my-password"
```

3. (optionally) remove masterkey from configs or where it is stored.
