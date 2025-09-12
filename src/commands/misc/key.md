# Key - Manage repository keys

**⚠️ Be aware**: This is not yet implemented!

The `key` command allows you to set multiple access keys or passwords per
repository. In fact, you can use the `list`, `add`, `remove`, and `passwd`
(changes a password) sub-commands to manage these keys very precisely:

```console
    $ rustic key list
    enter password for repository:
     ID          User        Host        Created
    ----------------------------------------------------------------------
    *eb78040b    username    kasimir   2015-08-12 13:29:57

    $ rustic key add
    enter password for repository:
    enter password for new key:
    enter password again:
    saved new key as <Key of username@kasimir, created on 2015-08-12 13:35:05.316831933 +0200 CEST>

    $ rustic key list
    enter password for repository:
     ID          User        Host        Created
    ----------------------------------------------------------------------
     5c657874    username    kasimir   2015-08-12 13:35:05
    *eb78040b    username    kasimir   2015-08-12 13:29:57
```

**Note**: that the currently used key is indicated by an asterisk (`*`).
