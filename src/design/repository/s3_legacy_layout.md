# S3 Legacy Layout

Unfortunately during development the Amazon S3 backend uses slightly different
paths (directory names use singular instead of plural for `key`, `lock`, and
`snapshot` files), and the pack files are stored directly below the `data`
directory. The S3 Legacy repository layout looks like this:

```console
/config
/data
  ├── 2159dd48f8a24f33c307b750592773f8b71ff8d11452132a7b2e2a6a01611be1
  ├── 32ea976bc30771cebad8285cd99120ac8786f9ffd42141d452458089985043a5
  ├── 59fe4bcde59bd6222eba87795e35a90d82cd2f138a27b6835032b7b58173a426
  ├── 73d04e6125cf3c28a299cc2f3cca3b78ceac396e4fcf9575e34536b26782413c
[...]
/index
  ├── c38f5fb68307c6a3e3aa945d556e325dc38f5fb68307c6a3e3aa945d556e325d
  └── ca171b1b7394d90d330b265d90f506f9984043b342525f019788f97e745c71fd
/key
  └── b02de829beeb3c01a63e6b25cbd421a98fef144f03b9a02e46eff9e2ca3f0bd7
/lock
/snapshot
  └── 22a5af1bdc6e616f8a29579458c49627e01b32210d09adb288d1ecda7c5711ec
```

The S3 backend understands and accepts both forms, new backends are always
created with the default layout for compatibility reasons.
