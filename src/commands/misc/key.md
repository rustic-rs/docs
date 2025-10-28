# Key - Manage repository keys

The `key` command allows you to set multiple access keys or passwords per
repository. In fact, you can use the `list`, `add`, `remove`, and `passwd`
(changes a password) sub-commands to manage these keys very precisely:

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
```

**Note**: that the currently used key is indicated by an asterisk (`*`).
