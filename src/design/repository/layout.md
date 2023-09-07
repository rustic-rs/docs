# Repository Layout

The `local` and `sftp` backends are implemented using files and directories
stored in a file system. The directory layout is the same for both backend types
and is also used for all other remote backends.

The basic layout of a repository is shown here:

```console
/tmp/restic-repo
├── config
├── data
│   ├── 21
│   │   └── 2159dd48f8a24f33c307b750592773f8b71ff8d11452132a7b2e2a6a01611be1
│   ├── 32
│   │   └── 32ea976bc30771cebad8285cd99120ac8786f9ffd42141d452458089985043a5
│   ├── 59
│   │   └── 59fe4bcde59bd6222eba87795e35a90d82cd2f138a27b6835032b7b58173a426
│   ├── 73
│   │   └── 73d04e6125cf3c28a299cc2f3cca3b78ceac396e4fcf9575e34536b26782413c
│   [...]
├── index
│   ├── c38f5fb68307c6a3e3aa945d556e325dc38f5fb68307c6a3e3aa945d556e325d
│   └── ca171b1b7394d90d330b265d90f506f9984043b342525f019788f97e745c71fd
├── keys
│   └── b02de829beeb3c01a63e6b25cbd421a98fef144f03b9a02e46eff9e2ca3f0bd7
├── locks
├── snapshots
│   └── 22a5af1bdc6e616f8a29579458c49627e01b32210d09adb288d1ecda7c5711ec
└── tmp
```

A local repository can be initialized with the `restic init` command, e.g.:

```console
restic -r /tmp/restic-repo init
```

The local and sftp backends will auto-detect and accept all layouts described in
the following sections, so that remote repositories mounted locally e.g. via
fuse can be accessed. The layout auto-detection can be overridden by specifying
the option `-o local.layout=default`, valid values are `default` and `s3legacy`.
The option for the sftp backend is named `sftp.layout`, for the s3 backend
`s3.layout`.
